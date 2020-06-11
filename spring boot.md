# 1. spring boot启动猜想

## 需要的储备的知识

- java注解的工作方式
- 多态
- idea的debug

## start

### 扫描出所有的bean

- 获取的了一些`ApplicationContextInitializer`
- 获得了一些`ApplicationListener`

- 扫描注解，利用反射实例化对象
  - 显示cogfigure

- 自动注入

# 2. 学习Spring Web框架的前提

## 2.1 注解

### 是什么

是一个标签

### 类型

- 普通注解

- 元注解：用于对普通注解进行标记
  - 生命周期：@Retention
  - 最用范围：@Target
  - @Inherited：被它标记过的注解是否可以被继承
  - @Repeatable：将标记过的注解放入一个容器注解中

### 注解属性

- `<type> name( ) `：类型  属性名

- `String name( ) default "hello"` ：指定了默认值

### 注解使用

```java
@TestAnnotation(value = "hello")
public class Test{

}

public @interface TestAnnotation{
	String value();
}
```

- 当注解只有一个value属性是，可以直接赋值`@TestAnnotation("my annotation")`

- 当一个注解没有任何属性的时候，可以省略后面的括号

```java
@NoFieldAnnotation
public class Test{

}

public @interface NoFieldAnnotation{

}
```



#### 别的类使用被注解过的类

**反射 反射 反射**



## 2.2 Cooki Session

完善HTTP协议无状态的特点,记录客户端的状态信息。

cookie在客户端

session在服务端

### 工作流程

client发送请求，若是第一次，服务器响应一个cookie放在`Repose`响应头中,下一次client发送请求会在`Request`头中加入cookie。

### Cookie类型

- 会话类型
  - 存在内存中，关闭浏览器cookie消失
- 持久类型
  - 以文件形式存在，关闭浏览器不会消失

### 认识Session

- 一个超大的map，储存了所有的cookie

- 提高安全性
- 不返回一个包含敏感信息的cookie，而是一个`JSEESSIONID`，将这个sessionid作为cookie返回，下次client再情求时根据这个id找到具体的信息。

# 3. SpringBoot注解

![spring注解](C:\Users\31472\Desktop\java\Java笔记\Service Component and Repository annotation difference Spring framework.png)

- Component：早期spring的注解。
- 其他注解是后面spring版本慢慢加进来的。
- 其他注解是`@Component`的特殊形式，在功能上是一样的（让bean自动被发现并进行依赖注入），但是在Spring MVC框架中有更好的可读性。

# 4. SpringBoot Stater

## 4.1 前提-SpringBoot启动流程

## 4.2 Auto-Configuration



# 5. Spring -Core

## 5.1 AOP

使用`CGlib`或者Java提供的`Proxy`动态生成了原先类的子类的字节码。将这个子类加载到Spring容器中，任何对原先类方法的调用由于多态的存在都会调用到动态生成的子类的实现上。

先实例化TargetObject，然后生成它的ProxyObejct

- JDK Proxy，如果bean使用了接口
- CGlib，bean没有使用接口

### 5.1.1 何时生成TargetObject？

doCreateBean()会先生成普通的TargetObject

### 5.1.2 何时生成的代理对象？

`getBean` ->getSingleton ->createBean ->initalizeBean->`applyBeanPostProcessorsAfterInitalization `

最后一个方法会获取容器所有的`BeanPostProcessor`，依次对bean进行额外的增强配置（类似于责任链模式），这其中有一个`AnnotationAwareAspectJAutoProxyCreator`会对bean进行代理，返回一个代理对象。

### 5.1.3 `AnnotationAwareAspectJAutoProxyCreator`这个BeanPostProcesser何时被注册到容器中？

refresh -> registerBeanPostProcessors 

遍历预置的beanPostProcessor，创建实例，让如容器。

### 5.1.4 被注解标注的bean，它的beanDefinition何时加到容器中？

<font color=#FF0000 >refresh的`invokeBeanFactoryPostProcessors`方法，将所有的注解beanDefiniton加到容器中</font>

## 5.2 ioc

容器启动顺序：核心`refresh()`

1. 先创建容器，会默认创建一个`DefaultListableBeanFactory`的容器。
2. 加载所有的`BeanDefinition`到容器中。
   1. `BeanDefinition`中保存Bean的信息，是哪个类，是否懒加载，依赖哪些bean



### 5.2.1 注解和xml方式加载BeanDefinition的区别。

- xml方式在`AbstractApplicationContext`的`refreshBeanFactory()`会根据xml文件的bean配置加载所有的`BeanDefinition`.
  在`obtainFreshBeanFactory()`方法执行前，beanDefinition没有放入容器中。
- 基于注解的方式在容器实例`AnnotationConfigApplicationContext`创建时，会实例化它的一个是你变量`ClassPathBeanDefinitionScanner`,该变量在实例化时就会将所有的类路径上标注了bean的注解的类作为`BeanDefinition`，也就是在refresh调用之前，beanDefinition已经已经放入容器中。

### 5.2.2 两者默认产生的容器的？

使用了``DefaultListableBeanFactory``

### 5.2.3实例化后的bean存放在容器哪里？

- 存放已经实例化的bean

![image-20200527154324527](C:\Users\31472\AppData\Roaming\Typora\typora-user-images\image-20200527154324527.png)

- 存放会产生bean的工厂类

![image-20200527154632480](C:\Users\31472\AppData\Roaming\Typora\typora-user-images\image-20200527154632480.png)

- 存放早期创建的一些特殊的bean

![image-20200527154903321](C:\Users\31472\AppData\Roaming\Typora\typora-user-images\image-20200527154903321.png)

## 5.2.4 bean何时被放入容器？

`getBean -> getSingleton -> addSingleton`

getSingleton中会创建完整的bean实例，在返回的之前将bean实例添加到容器中。

![image-20200529145543570](C:\Users\31472\AppData\Roaming\Typora\typora-user-images\image-20200529145543570.png)

## 5.3 bean 生命周期

## 5.4 预置的BeanPostProcessor在哪里定义的

在refresh的`invokeBeanFactoryPostProcessors`方法中会添加一些预置的`BeanPostProcessor`