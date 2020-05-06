# 重构

# 重构原则

- 不要重复你自己（DRY）
- 易修改，易扩展
- 高内聚低耦合（面向对象）

## 重构技巧

- `switch case`使用枚举类型替代
- 复杂的多个条件判断抽成一个方法
- 大方法仇抽成若干个小方法



# 设计模式

## 策略模式

大量的if，else或者switch case

```java
if(xxx){

}else if(xxx){
    
}else if(xx){
    
}
...
    
// 或者
  switch(xx):
case "xx1" :dosomething1();
case "xx2" :dosomething2();
case "xx3" :dosomething3();
case "xx4" :dosomething4();
case "xx5" :dosomething5();
.....
```

这种时候可以将业务代码与具体的实现细节分离，比如使用枚举

参照`java`线程池`ThreadPoolExecutor`的构造函数最后一参数`RejectedExecutorHandler`.这里的handler是一个接口，它的实现类可以现在还不存在。

```java
 public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
```

## 享元模式

巨量的小对象，但是被很多人所共享，这个时候可以使用一个池子，别人用的时候直接从池子里面拿。

参照`Integer.valueOf()`,这里面就是用了缓存。



## 访问者模式

- 适用于结构固定的数据
- 将数据与访问模式相互分离
- 将访问的操作分离出来