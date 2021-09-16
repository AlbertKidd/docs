[TOC]

# MongoDB使用简介

本文主要介绍如何对MongoDB进行操作，主要分为以下两个部分：
* 原生MongoDB操作，即在shell中对MongoDB进行操作
* 在Spring中使用MongoDB
下面进行逐一介绍：

## 1. 原生MongoDB操作

在MongoDB server安装完成之后，可以使用shell命令行对MongoDB进行操作，连接mongoDB仅需输入`mongo`即可。


### 1.1 数据库操作

* use [db]
使用指定名称的数据库（若该库不存在，则会在对库中表操作时创建该库）
* db
查看当前使用的数据库名称
* show dbs 
查看所有的数据库名称即储存数据量
* db.dorpDatabase()
删除当前使用的数据库

### 1.2 集合操作

MongoDB中的集合类似于关系型数据库中表的概念，常用操作命令如下：

* db.createCollection(name,options)
创建集合，options为可选，示例如下
```
// 创建hello集合
db.createCollection("c1")
// 创建hello集合，数据条数最大为500条，数据量最大为1000KB，并为_id字段自动创建索引
db.createCollection("c1", { capped: true, autoIndexId: true, size:1000, max: 500 } )
```
* db.[collectionName].drop()
删除集合，例如`db.c1.drop()`
* show collections
查看当前数据库的所有集合

### 1.3 insert操作

当对集合进行插入操作时，若集合不存在，则会创建该集合。

* db.[collectionName].insertOne()
向指定集合中插入一条纪录，例如：
`db.c1.insertOne({name: 'Kidd', age: 33, address: 'xian'})`
* db.[collectionName].insertMany()
向指定集合中插入多条纪录，例如：
`db.c1.insertMany([{name: 'Kidd', age: 33, address: 'xian'}, {name: 'petrel', age:18, address: 'guangzhou'}])`

### 1.4 query操作

一个完整的查询命令如下所示:

`db.[collectionName].find([criteria],[projection]).limit([limitNum]).skip([skipNum]).sort([sort])`

由于参数较多，我们分别进行介绍：

* criteria 
即查询条件，传入一个json对象，常用的查询条件如下：

条件 | 说明
-- | --
$eq | 等于
$ne | 不等于
$gt | 大于
$gte | 大于等于
$lt | 小于
$lte | 小于等于
$in | 在指定列表中
$nin | 不在指定列表中
$regex | 正则匹配
$or | 或操作

criteria实例：
`{name: 'Kidd', age: {$lt: 35}, address: {$regex: 'xian'}, $or:[{gender: 'female'}, {gender: 'neutral'}]}`

如果把此查询条件转为sql来理解，则为
`where name = 'Kidd' and age < 35 and address like '%xian' and (gender = 'female' or gender = 'neutral')`

* projection
projection也是一个json对象，对列进行筛选，若此项为空则返回所有的列。
列值为1则意为包含该列，为0则为排除，不能混合使用。实例如下：
`{name: 1, age: 1}`
`{name: 0, address: 0}`

* limit
整数，即为最大查出的数据条数

* skip
整数，查询忽略前n条的数据

* sort
json格式，对列的排序方向进行指定，1为升序，-1为降序。实例如下：
`{name: 1, age: -1}`

一个完整的查询操作实例如下：
`db.c1.find({name: {$regex: 'dd'}, age: {$lt: 35}}, {address:0}).limit(10).skip(10).sort({name:1, age:-1})`

### 1.5 update操作

* db.[collectionName].updateOne([criteria],[action],[options])
更新找到的第一条数据
* db.[collectionName].updateMany([criteria],[action],[options])
更新找到的所有数据
* db.[collectionName].replaceOne([criteria],[record],[options])
替换找到的第一条数据，record为json形式的数据

此处criteria与query中相同，不再赘述，主要介绍一下action与options。

#### 1.5.1 update中的action
action指的是当对查出的数据进行的操作，常用的action如下：

action | 说明
- | -
$currentDate | 将指定的字段设置为当前日期
$inc | 对指定字段的值增加指定的数值
$min | 仅在此处指定的min值比查询值小时将该值更新为min值
$max | 仅在此处指定的max值比查询值大时将该值更新为max值
$mul | 对指定字段的值乘以指定的数值
$rename | 对字段名进行更改
$set | 更改指定字段的值（最常用）
$setOnInsert | 当操作为insert时，设置一些字段的默认值
$unset | 删除指定字段的值

action实例如下：
`{$set: {name: "petrel"},$currentDate: { lastModified: true }}`

#### 1.5.2 update中的options
options为可选参数，可以传入以下的更新选项

option | 说明
-- | --
upsert | boolean值，为true时如果根据条件没有找到数据，则根据此条件与action合并插入一条数据。默认为false。
writeConcern | 写入策略
collation | 指定语言规则
arrayFilters | 数组过滤器

update命令实例如下：
`db.c1.update({name: "Kidd", age: {$lt: 35}}, {$set: {name: "petrel"}}, {upsert: true})`

### 1.6 delete操作

delete操作较为简单，有以下两种：
* db.[collectionName].deleteOne([filter])
从指定集合中删除符合条件的第一条数据
* db.[collectionName].deleteMany([filter])
从指定集合中删除符合条件的所有数据

filter参数同query函数中的criteria参数。

delete命令实例如下：
`db.c1.deleteOne({name: "Kidd", age: {$lt: 35}})`

## 2. 在Spring中使用MongoDB

在Spring框架下对MongoDB进行操作主要有两种方式：
* 使用MongoRepository，采用JPA将数据库与Java对象进行映射等方式进行CRUD操作
* 使用MongoTemplate，类似JdbcTemplate的方式操作MongoDB

### 2.1 MongoRepository的使用

暂略

### 2.2 MongoTemplate API简介

使用MongoTemplate，表名通常可以不进行传入，Spring会通过处理对象的类型将类名作为表名进行处理

#### 2.2.1 插入操作
```java
// 插入单个对象
void insert(Object objectToSave);
void insert(Object objectToSave, String collectionName);

// 向指定表中批量插入
void insert(Collection<? extends Object> batchToSave, String collectionName);
void insert(Collection<? extends Object> batchToSave, Class<?> entityClass);

// 批量插入，集合中可以是不同的类型，插入不同的表中
void insertAll(Collection<? extends Object> objectsToSave);
```

#### 2.2.2 查询操作

MongoTemplate中使用`org.springframework.data.mongodb.core.query.Query`来构造查询配置。
Query中包含多个查询的多个配置：
* Criteria 字段条件
* Sort 排序
* skip 略过前skip数量的数据
* limit 限制查询出数据的最大数量

构造Query查询配置示例如下：
```java
// 构造查询条件
Criteria criteria = Criteria
    .where("key1").is(value1) //等于
    .and("key2").gt(value2) //大于
    .and("key3").regex(value3) //正则匹配

// 构造Sort
Sort sort = Sort.by(Sort.Order.asc("key1"), Sort.Order.desc("key2"));

int start = 0;
int limit = 10;
// 构造Query
Query query = Query.query(criteria)
                    .with(sort)
                    .skip(start)
                    .limit(10);
```

常用的查询API如下所示：

```java
// 根据查询条件查出指定表中的所有数据，并映射为指定类型的对象返回列表
<T> List<T> find(Query query, Class<T> entityClass);
<T> List<T> find(Query query, Class<T> entityClass, String collectionName);

// 查出表中所有的数据，并映射为指定类型的对象返回列表
<T> List<T> findAll(Class<T> entityClass);
<T> List<T> findAll(Class<T> entityClass, String collectionName);

// 根据指定的id查出指定表中的数据，并映射为指定类型的对象
<T> T findById(Object id, Class<T> entityClass);
<T> T findById(Object id, Class<T> entityClass, String collectionName);

// 根据查询条件查出指定表中符合条件的第一条数据，并映射为指定类型的对象返回列表
<T> T findOne(Query query, Class<T> entityClass);
<T> T findOne(Query query, Class<T> entityClass, String collectionName);

// 查询表中符合条件的记录数
long count(Query query, Class<?> entityClass);
long count(Query query, Class<?> entityClass, String collectionName);
long count(Query query, String collectionName);


```

#### 2.2.3 更新操作
MongoTemplate中使用`org.springframework.data.mongodb.core.query.Update`来构造查询配置。
其基础用法如下：
```java

Update update = Update.update("key1", value1) //修改指定字段的值
            .inc("key2", 1) //将某字段的值增加给定的数值
            .rename("key3", "key4") //将某字段名改为新的字段名
```

常用的更新API如下所示：

```java

// 根据查询条件查出第一条数据并进行修改，返回的是修改前的数据
<T> T findAndModify(Query query, Update update, Class<T> entityClass);
<T> T findAndModify(Query query, Update update, Class<T> entityClass, String collectionName);

// 根据查询条件查出第一条数据并进行修改，options中可配置返回修改后的数据
<T> T findAndModify(Query query, Update update, FindAndModifyOptions options, Class<T> entityClass);
<T> T findAndModify(Query query, Update update, FindAndModifyOptions options, Class<T> entityClass,
            String collectionName);

// 修改查询到的第一条数据，返回处理结果
WriteResult updateFirst(Query query, Update update, Class<?> entityClass);
WriteResult updateFirst(Query query, Update update, Class<?> entityClass, String collectionName);
WriteResult updateFirst(Query query, Update update, String collectionName);

// 修改查询到的所有数据，返回处理结果
WriteResult updateMulti(Query query, Update update, Class<?> entityClass);
WriteResult updateMulti(final Query query, final Update update, Class<?> entityClass, String collectionName);
WriteResult updateMulti(Query query, Update update, String collectionName);

// 修改查询到的第一条数据，若未查到数据，则按照Query与Update合并插入一条数据
WriteResult upsert(Query query, Update update, Class<?> entityClass);
WriteResult upsert(Query query, Update update, Class<?> entityClass, String collectionName);
WriteResult upsert(Query query, Update update, String collectionName);

// 保存数据至表中，若该数据存在，则覆盖原数据
void save(Object objectToSave);
void save(Object objectToSave, String collectionName);

```

#### 2.2.4 删除操作

常用的删除API如下：
```java
// 找到符合查询条件的第一条数据并将其删除，返回被删除的记录
<T> T findAndRemove(Query query, Class<T> entityClass);
<T> T findAndRemove(Query query, Class<T> entityClass, String collectionName);

// 找到符合查询条件的所有数据并将其删除，返回被删除的记录列表
<T> List<T> findAllAndRemove(Query query, Class<T> entityClass);
<T> List<T> findAllAndRemove(Query query, Class<T> entityClass, String collectionName);
<T> List<T> findAllAndRemove(Query query, String collectionName);

// 删除符合指定的对象id的数据，返回处理结果
WriteResult remove(Object object);
WriteResult remove(Object object, String collection);

// 删除符合查询条件的所有数据，返回处理结果
WriteResult remove(Query query, Class<?> entityClass);
WriteResult remove(Query query, Class<?> entityClass, String collectionName);
WriteResult remove(Query query, String collectionName);

```


#### 2.2.5 Collection操作

常用API如下：
```java

// 创建表，并返回表对象
<T> DBCollection createCollection(Class<T> entityClass);
<T> DBCollection createCollection(Class<T> entityClass, CollectionOptions collectionOptions);
DBCollection createCollection(String collectionName);
DBCollection createCollection(String collectionName, CollectionOptions collectionOptions);

// 判断表是否存在
<T> boolean collectionExists(Class<T> entityClass);
boolean collectionExists(String collectionName);

// 获取表对象
DBCollection getCollection(String collectionName);

// 根据类获取表名
String getCollectionName(Class<?> entityClass);

// 获取数据库中所有的表名
Set<String> getCollectionNames();

// 删除表
<T> void dropCollection(Class<T> entityClass);
void dropCollection(String collectionName);

```

---
参考文档：
1. [MongoDB官方文档](https://docs.mongodb.com/manual)
2. [MongoTemplate官方文档](https://docs.spring.io/spring-data/mongodb/docs/2.1.9.RELEASE/reference/html/)