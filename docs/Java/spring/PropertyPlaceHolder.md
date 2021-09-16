# Spring Properties PlaceHolder 详解
## 1. 概述
在使用 Spring 时，大家应该都用过使用占位符来代替固定的配置。例如在注入数据源时：
```
<bean id="datasource" class="org.apache.commons.dbcp2.BasicDataSource"
  p:driverClassName="${jdbc.driverClassName}"
  p:url="${jdbc.url}"
  p:username="${jdbc.username}"
  p:password="${jdbc.password}"/>
```
想要实现像 `${jdbc.password}` 的占位符配置，首先需要对 Spring 进行配置，目前（2018.06）官方提供了两种形式：
1. 在 spring xml 配置文件中定义 ` <property-placeholder> ` 或定义 `PropertyPlaceholderConfigurer` 的 bean 来读取 properties 文件；
2. 在 Java 代码中使用注解的形式进行读取；
两种方法各有长短，下面我们会进行详细的介绍。

## 1. 在 xml 文件中读取 properties 文件

### 1.1 两种写法
在 Spring 配置文件中有两种写法：
1. 使用 ` <property-placeholder> ` ，示例如下：
```
<context:property-placeholder location="classpath:/conf/jdbc.properties" order="1"/>
```
2. 定义 `PropertyPlaceholderConfigurer` 的 bean ，达到同样的效果：
```
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="location" value="classpath:/conf/jdbc.properties"/>
    <property name="order" value="1"/>
</bean>
```
在这里，建议使用 ` <property-placeholder> ` 的形式，因为其内部实际作用的类就是 `PropertyPlaceholderConfigurer` ，而且写法更加简洁。

### 1.2 PropertyPlaceholderConfigurer 的加载顺序
在介绍 ` <property-placeholder> ` 的各项属性之前，我们需要先了解 PropertyPlaceholderConfigurer 的属性加载顺序

首先来明确一下定义：
* `fileProperties`: 定义在 `*.properties` 文件中的属性
* `localProperties`: 定义在 `properties-ref` 中的属性
* `systemProperties`: JVM 属性，即 SysTem.getProperties() 获取到的属性
* `envProperties`: 系统环境变量属性

在默认的读取占位符流程中，会按照 `fileProperties` -> `localProperties` -> `systemProperties` -> `envProperties` 的顺序进行键的读取，如果读取到了便返回其对应的值。

### 1.3 property-placeholder 的属性介绍
下面介绍 ` <property-placeholder> ` 的各项属性配置：
* `location`: properties 文件的路径，可以直接输入当前 resources 目录下的相对路径如 `conf/jdbc.properties`，也可以是 `classpath:/path/file`, `classpath*:/path/file`, `file:/path/file`, `http:host/path/file` 等等。对于多个文件，只需用 `,` 隔开即可，如 `conf/jdbc.properties，file:/path/file`。（注意：当指定多个文件时，如果文件有相同的属性名，则读取的是后文件中的属性值）
* `order`: 加载的优先级，值越小则优先级越高。例如在 Spring 配置文件中写入多个 ` <property-placeholder> ` ，解析同样的属性名则会优先读取 `order` 值最小的。
* `file-encoding`: 指定解析 properties 文件的编码类型
* `ignore-resource-not-found`: 是否忽略 `location` 中的文件未找到时抛出的异常，默认为 `false` ，即抛出异常
* `ignore-unresolvable`: 是否忽略占位符中的键未找到时抛出的异常，默认为 `false` ，即抛出异常
* `local-override`: 默认为 `false` ，当为 `true` 时， `properties-ref` 指定的值将会覆盖从 properties 文件中读取到的值
* `null-value`: 将指定的字符串解析为 `null` ，例如 `""` （空字符串）， `"null"` 等
* `properties-ref`: 指定一个 Properties bean 的 id ，属性由该 bean 中读取
* `system-properties-mode`: 
* `trim-values`:
* `value-separator`:
## 2. Java 代码中使用注解读取 properties 文件
## 3. 两种方式的对比
property-placeholder无法继承
java代码的注解形式可以
http://www.baeldung.com/properties-with-spring#parent-child