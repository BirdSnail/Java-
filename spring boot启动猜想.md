## spring boot启动猜想

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

# 前提

## 注解

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



## Cooki Session

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

