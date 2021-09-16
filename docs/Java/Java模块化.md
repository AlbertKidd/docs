https://blog.csdn.net/houxuehan/article/details/78549711?locationNum=8&fps=1
http://www.cnblogs.com/IcanFixIt/p/7278696.html

## 1. features
* 更加方便地管理依赖
* 提升安全性
* 更加轻量，提升jdk运行性能
* 大小灵活，适用不同的设备

## 2. 模块化实战
### 2.1 模块的声明
使用模块化须在根目录添加module-info.java，声明模块如下：
```java
module com.kidd.demo.b{
   
}
```
### 2.2 模块的输入与输出
* 输入语法

`requires *`： 依赖指定模块
```java
module com.kidd.demo.b{
   requires com.kidd.demo.c;
}
```

`requires * transitive`： 可传递的依赖
```java
module com.kidd.demo.b{
   requires com.kidd.demo.c transitive;
}
```

* 输出语法

`exports`： 输出当前模块下的某个包
```java
module com.kidd.demo.b{
   exports com.kidd.demo.b;
}
```

`exports * to *`： 将当前模块下的指定包输出至指定模块，仅该模块可使用此包
```java
module com.kidd.demo.b{
   exports com.kidd.demo.b to com.kidd.demo.a;
}
```

### 2.3 模块的松耦合
客户端使用接口时可以不直接使用具体的实现类，可以获取类路径中所有提供此接口的实现类。
例如有三个模块，接口模块、生产者模块、消费者模块，生产者模块与消费者模块依赖于接口模块。
生产者提供接口的实现类，消费者使用接口。
```java
module com.kidd.demo.api{
   exports com.kidd.demo.api;
}
module com.kidd.demo.producer{
   requires com.kidd.demo.api;
   provides com.kidd.demo.api.Sort with com.kidd.demo.producer.QuickSort;
}
module com.kidd.demo.consumer{
   requires com.kidd.demo.api;
   uses com.kidd.demo.api.Sort;
}
```
以下两个关键字需要注意：
* `provides 接口 with 实现类`：提供接口的实现
* `uses 接口`：使用该接口

当消费者模块使用接口时：
```java
// 当 consumer 模块的类路径中包含 producer 项目时，可以使用ServiceLoader 获取到 producer 中在模块声明中提供的所有接口实现类；
// 类似于 Spring 的 getBeansTypeOf() 方法
Iterator<Sort> iterator = ServiceLoader.load(Sort.class).iterator();
iterator.next().sort(list);
```