# 1. Java虚拟机

## 1.1  JVM内存数据结构

![img](https://upload-images.jianshu.io/upload_images/2179030-f81a3bf0df216749.png?imageMogr2/auto-orient/strip|imageView2/2/w/813/format/webp)

## 1.2 class字节码文件

### 1.2.1 结构

### 1.2.2 指令

* 编译成字节码后，每一个构造函数对应的一个`<init>`方法
* 所有的static字段，代码块按照声明顺序放入`static <clinit>`方法中
* `private`，`static`修饰过的方法会调用`invokespecial`，`invokestatic`指令，这两个指令虚拟机规定就没有多态。

## 1.3 AFQ

### 1.3.1 String常量池 **vs** class常量池与 **vs** Runtime常量池

* class常量池属于class文件结构的一部分，JVM对class文件结构有严格的格式要求，在编译器就已经确定了相关值
* Runtime常量池：属于方法区的一部分，是整个JVM规范的所规定的一块区域。**将class字节流代表的静态储存结构转化为方法区的运行时数据结构**，其中就包含了class文件常量池进入运行时常量池的过程
* String常量池：因为String是一个不可变对象，区别与普通的常量，因此String被单独列出来

# 2. ClassLoader

> Class loaders are responsible for **loading Java classes during runtime dynamically to the JVM** (Java Virtual Machine). 

## 2.1 class file的旅程

* loading
* linking
* initialization

JVM充当了中间人的角色，他提供二进制class的运行环境，并于底层环境像交互。