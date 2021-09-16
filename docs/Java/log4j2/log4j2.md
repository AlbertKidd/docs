# Log4j2简介

## 1. 概述

顾名思义，log4j是专为java使用的一款日志框架。
2代表着他是该框架的第二个重大版本。由于与原先第一版的框架log4j并不兼容，新的框架被命名为Log4j2。

## 2. log4j2的使用

使用Log4j2记录日志，首先需要在类中定义一个org.apache.logging.log4j.Logger的静态变量即可：

```java
package com.ebig;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class LogDemo{

    // 定义Logger实例
    private static Logger log = LogManager.getLogger();

}
```

在使用有集成lombok库的项目中，可以使用在类上注解@Slf4j、@Log4j2的方式来定义Logger变量：

```java
package com.ebig;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class LogDemo{

    public static void main(String[] args){
        // 使用@Slf4j注解后，类中会自动生成一个log变量
        log.info("hello world");
    }

}
```

获取到Logger实例后，就可以使用它记录日志了。
默认的日志级别有trace/debug/info/warn/error五种，使用的api相差无几，以info为例：

```java
package com.ebig;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class LogDemo{

    public static void main(String[] args){
        // 简单记录日志
        log.info("hello world");

        // 格式化字符串记录日志，后面的参数为不定参数
        log.info("%s %s", "hello", "world");

        // 记录异常栈，第二个参数为Throwable对象(记录信息同e.printStackTrace())
        log.info("something wrong", new RuntimeException(""));
    }

}
```

## 3. log4j2的基本配置

log4j2的使用是十分简单的，但日志记录在哪里，记录的格式，以及记录的内容需要建立在配置的前提下。
log4j2支持多种形式的配置文件，如xml、json、properties等，下面我们以xml形式的配置文件为例，探索一下log4j2是如何配置的。

下面的代码展示了一个官方提供的通用配置结构：
```xml
<?xml version="1.0" encoding="UTF-8"?>;
<Configuration>
  <Filter type="type" ... />
  <Appenders>
    <Appender type="type" name="name">
      <Filter type="type" ... />
    </Appender>
    ...
  </Appenders>
  <Loggers>
    <Logger name="name1">
      <Filter type="type" ... />
    </Logger>
    ...
    <Root level="level">
      <AppenderRef ref="name"/>
    </Root>
  </Loggers>
</Configuration>
```

其整体架构如下：

![log4j2架构](img/Log4jClasses.jpg)

下面进行逐一介绍：

### 3.1 Configuration

Configuration是log4j2配置的根节点，其下包含了Filter（全局级）、Loggers、Appenders等各个组件。
Configuration有多个属性可以配置，最重要的是以下两个：

* **status**: log4j2框架的内部日志打印级别，通常设置在warn级别以上
* **packages**: 读取自定义LookUp，自定义Appender等组件的包路径，只有在已声明包下的组件才会被log4j2框架读取

配置示例：
```xml
<?xml version="1.0" encoding="utf-8" ?>
<Configuration status="Warn"  packages="com.ebig.ebeit.framework.log">
    ...
</Configuration>
```

### 3.2 Loggers

Loggers中可以配置多个Logger，Logger实际上就是我们在使用`LogManager.getLogger()`时获取到的配置。

Logger主要有以下属性：

属性名 | 描述
-- | --
name | 包名，表示哪些包下的类获取本Logger
level | 打印日志的最低级别，例如info，低于此级别的日志将不会被打印
additivity | 是否向上合并读取配置，当有一个包名含括范围更广的Logger存在时，如果此属性配置为true，日志还会以上级的logger配置输出。
AppenderRef | 日志输出实例（可以有多个），表示当前日志会以什么方式进行输出。

例如：
```xml
<Loggers>
    ...
    <Logger name="com.ebig.ebeit.etl" level="info" additivity="false">
        <AppenderRef ref="jobs"/>
        <AppenderRef ref="dbLogger"/>
    </Logger>
</Loggers>
```
上述配置表示了在com.ebig.ebeit.etl包下用log4j2打印日志只会打印info级别以上的日志，并会使用jobs、dbLogger两个日志记录器输出两份日志。

需要注意的是，Loggers中必须要配置一个Root级别的Logger。如果打印日志的类没有被其他Logger配置覆盖到，则会读取Root配置，如下：

```xml
<Loggers>
    ...
    <Root level="info">
        <AppenderRef ref="stdout"/>
        <AppenderRef ref="ebeit"/>
    </Root>
    <Logger name="com.ebig.ebeit.etl" level="info" additivity="false">
        <AppenderRef ref="jobs"/>
        <AppenderRef ref="dbLogger"/>
    </Logger>
</Loggers>
```
使用上述配置，如果我在com.ebig.ebeit.framework包下的类中打印日志时，则会读取到Root配置，打印info级别以上的日志，并向控制台、ebeit日志记录器中分别输出日志。

### 3.3 Appenders

Appenders下面包含多个Appender，可以理解为日志输出器的定义。
log4j2提供了多种Appender的实现，常用的有以下几种：

输出器 | 描述
-- | --
Console | 将日志输出至控制台
RollingRandomAccessFile | 将日志记录至文件中，可以按照时段输出至多个文件
Async | 异步的日志输入器
JDBC | 记录日志到数据库中
JMS | 以消息的方式记录日志
Routing | 将日志进行路由处理，可以将不同条件的日志输出至不同的位置


配置方式如下：
```xml
<Appenders>
    <!-- 文件日志 -->
    <RollingRandomAccessFile name="RollingRandomAccessFile" fileName="logs/app.log"
                    filePattern="logs/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log.gz">
        <PatternLayout>
            <Pattern>%d %p %c{1.} [%t] %m%n</Pattern>
        </PatternLayout>
        <Policies>
        <TimeBasedTriggeringPolicy />
        <SizeBasedTriggeringPolicy size="250 MB"/>
        </Policies>
    </RollingRandomAccessFile>

    <!-- 控制台日志 -->
    <Console name="STDOUT" target="SYSTEM_OUT">
      <PatternLayout pattern="%m%n"/>
    </Console>

    <!-- 异步日志 -->
    <Async name="Async">
      <AppenderRef ref="MyFile"/>
    </Async>

    <!-- 数据库日志 -->
    <JDBC name="databaseAppender" tableName="dbo.application_log">
      <DataSource jndiName="java:/comp/env/jdbc/LoggingDataSource" />
      <Column name="eventDate" isEventTimestamp="true" />
      <Column name="level" pattern="%level" />
      <Column name="logger" pattern="%logger" />
      <Column name="message" pattern="%message" />
      <Column name="exception" pattern="%ex{full}" />
    </JDBC>

    <!-- 消息日志 -->
    <JMS name="jmsQueue" destinationBindingName="MyQueue"
         factoryBindingName="MyQueueConnectionFactory">
      <JsonLayout properties="true"/>
    </JMS>
</Appenders>
```

在日志的记录器的配置中，可以发现大多数记录器都会有一个Layout的属性，比如RollingRandomAccessFile中的PatternLayout，以及JMS中的JsonLayout。
我们最常用的Layout即是PatternLayout，它的作用是指定了单条日志的输出格式。
例如：
`pattern=%d %p %t %F %c( %L ) user:[$${ebeit:loginAccountCode}] userParam:[%X{udfParam}] - %m%n`
这个配置输出的日志信息为：
`2019-06-04 11:24:51,056 INFO qtp706895319-18 LoginListener.java com.ebig.ebeit.security.utils.login.LoginListener( 84 ) user:[admin] userParam:[] - account admin login.`

具体通配符的含义可参见官方文档：[PatternLayout](http://logging.apache.org/log4j/2.x/manual/layouts.html#PatternLayout)

### 3.4 Filter

顾名思义，Filter是日志过滤器，负责过滤哪些日志才能被输出。

Filter有四个位置可以配置，分别是Configuration（全局过滤）、Logger级别、Appender级别、Appender Reference级别，例如：

```xml
<Configuration status="warn" >
    <Filters>
    <!-- 全局级别Filter -->
    <LevelRangeFilter minLevel="DEBUG" maxLevel="ERROR" onMatch="DENY"></LevelRangeFilter>
    </Filters>

    <Appenders>
        <RollingFile name="RollingFile" fileName="logs/app.log"
                    filePattern="logs/app-%d{MM-dd-yyyy}.log.gz">
        <!-- Appender级别的Filter -->
        <BurstFilter level="INFO" rate="16" maxBurst="100"/>
        <PatternLayout>
            <pattern>%d %p %c{1.} [%t] %m%n</pattern>
        </PatternLayout>
        <TimeBasedTriggeringPolicy />
        </RollingFile>
    </Appenders>    

    <Loggers>
        <!-- Logger级别的Filter -->
        <Root level="error">
        <MapFilter onMatch="ACCEPT" onMismatch="NEUTRAL" operator="or">
            <KeyValuePair key="eventId" value="Login"/>
            <KeyValuePair key="eventId" value="Logout"/>
        </MapFilter>
        </Root>

        <Logger name="TestJavaScriptFilter" level="trace" additivity="false">
        <!-- AppenderRef级别的Filter -->
        <AppenderRef ref="List">
            <ScriptFilter onMatch="ACCEPT" onMisMatch="DENY">
            <ScriptRef ref="filter.js" />
            </ScriptFilter>
        </AppenderRef>
        </Logger>
  </Loggers>

</Configuration>
```

具体filter的使用由于尚未使用，本次暂时不作介绍。

### 3.5 Lookup

在log4j2的配置中，可以看到是可以使用通配符来获取系统、环境变量的。
Lookup即是获取变量的实现类，log4j2提供了一些预设的LookUp，例如：
获取打印日志时设置的context临时变量：`${ctx:loginId}`
获取当前日期：`${date:MM-dd-yyyy}`
获取环境变量：`${env:USER}`
获取系统变量：`${sys:jdbc.username}`

我们也可以定义自己的LookUp来获取自定义的变量信息，只需实现对应的LookUp接口，介于篇幅原因，不再多做介绍。

具体的使用可参见官方文档：[PatternLayout](http://logging.apache.org/log4j/2.x/manual/layouts.html#PatternLayout)

## 4. log4j2配置实例

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Configuration status="Warn"  packages="com.ebig.ebeit.framework.log">
    <Appenders>
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout
                    pattern="%d %p %t %F %c( %L ) user:[$${ebeit:loginAccountCode}] userParam:[%X{udfParam}] - %m%n"/>
        </Console>

        <Routing name="ebeit">
            <Routes pattern="$${ebeit:ctx}">
                <Route>
                    <RollingRandomAccessFile name="ebeit-${ebeit:ctx}" append="true"
                                             fileName="./AppData/${ebeit:ctx}/log/ebeit.log"
                                             filePattern="./AppData/${ebeit:ctx}/log/ebeit-%d{yyyy-MM-dd}.log.gz">
                        <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} - %p %c{1}( %L )%x - %m%n"/>
                        <Policies>
                            <TimeBasedTriggeringPolicy />
                        </Policies>
                        <DefaultRolloverStrategy max="7"/>
                    </RollingRandomAccessFile>
                </Route>
            </Routes>
        </Routing>

        <Routing name="jobs">
            <Routes pattern="$${ebeit:ctx}">
                <Route>
                    <RollingRandomAccessFile name="jobs-${ebeit:ctx}" append="true"
                                             fileName="./AppData/${ebeit:ctx}/log/jobs.log"
                                             filePattern="./AppData/${ebeit:ctx}/log/jobs-%d{yyyy-MM-dd}.log.gz">
                        <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} - %-5p %c{1}( %L )%x utid:[${ebeit:utid}] - %m%n"/>
                        <Policies>
                            <TimeBasedTriggeringPolicy />
                        </Policies>
                        <DefaultRolloverStrategy max="7"/>
                    </RollingRandomAccessFile>
                </Route>
            </Routes>
        </Routing>

        <SysLoggerAppender name="dbLogger"/>

    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="stdout"/>
            <AppenderRef ref="ebeit"/>
        </Root>
        <Logger name="SysLogger.createAccess" level="info" additivity="false">
            <AppenderRef ref="jobs"/>
            <AppenderRef ref="dbLogger"/>
        </Logger>
        <Logger name="SysLogger.importAccess" level="info" additivity="false">
            <AppenderRef ref="jobs"/>
            <AppenderRef ref="dbLogger"/>
        </Logger>
        <Logger name="SysLogger.etl" level="info" additivity="false">
            <AppenderRef ref="jobs"/>
            <AppenderRef ref="dbLogger"/>
        </Logger>
        <Logger name="SysLogger.schedule" level="info" additivity="false">
            <AppenderRef ref="jobs"/>
            <AppenderRef ref="dbLogger"/>
        </Logger>
        <Logger name="SysLogger.dbsync" level="info" additivity="false">
            <AppenderRef ref="jobs"/>
            <AppenderRef ref="dbLogger"/>
        </Logger>

        <Logger name="jgroups" level="warn"/>

        <Logger name="org.quartz" level="info" additivity="false">
            <AppenderRef ref="jobs"/>
        </Logger>

        <Logger name="net.ucanaccess" level="info" additivity="false">
            <AppenderRef ref="jobs"/>
        </Logger>

        <Logger name="hsqldb.db" level="info" additivity="false">
            <AppenderRef ref="jobs"/>
        </Logger>

        <Logger name="com.ebig.ebeit.jobs" level="info" additivity="false">
            <AppenderRef ref="jobs"/>
        </Logger>

        <Logger name="com.ebig.ebeit.etl" level="info" additivity="false">
            <AppenderRef ref="jobs"/>
        </Logger>

        <Logger name="com.ebig.spdesb" level="info" additivity="false">
            <AppenderRef ref="jobs"/>
        </Logger>

    </Loggers>
</Configuration>
```

Log4j2官方文档：http://logging.apache.org/log4j/2.x/manual/configuration.html