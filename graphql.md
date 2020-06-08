### 一 背景

#### 与 sql

graphql -> structured ql

SQL 协议, MySQL 是实现.

GraphQL 协议, Appllo, Relay 是实现.

SQL 数据源是数据库, GraphQL 数据源可以是 REST API, 微服务, 数据库...

#### 1. rest api 的不足

- 数据拼接(多个请求)
- 过量获取
- 复杂数据请求, 多次调用. 数据请求有依赖关系, 导致两次请求为两次网络传输时间之和

#### 2. GraphQL 如何解决

- api 聚合
- 按需获取. GraphQL 查询语句, 可包含多个  字段, 每个字段对应一种数据结构(多个请求), 并需要显示声明数据结构中需要返回的字段(过量获取).
- 数据分层. 嵌套查询(次级查询), 查询语句中可支持嵌套查询, GraphQL 会从外层一次解析并返回数据, 嵌套的内层查询可获取到 parent 数据.

#### 3. 与 rest 比较

- 数据获取, graphql 按需获取
- api 调用, graphql 只有一个 endpoint, rest 一个资源对应一个 endpoint
- 复杂数据请求, graphql 数据分层, 嵌套查询一次调用
- 错误码, graphql 状态码统一 200, 错误信息包装. rest 精确返回错误码.
- 版本号: graphql 通过 schema 扩展, rest 通过 v1/v2, url 中预览版本路径

schema = types + resolvers

#### 3. GraphQL 优势

1. 工程化

schema 现行
schema -> ts: type

2. 支持订阅

### 二 graphql 介绍

#### 1. server schema 和类型

类型的集合, 它的定义决定了 endpoint 能做些什么, schema 的 Query, Mutation, Subscription 为根类型

schema 中由普通对象类型和特殊类型组成, Query 类型定义了查询入口, Mutation 类型定义了变更的入口, Subscription 类型定义了订阅入口.

##### type

数据模型的抽象, 与表结构无关, 仅表示能从该类型上获取到哪些, type 应是类型完备的.

- 字段
- 参数

1. 对象类型和字段, 字段可为标量和对象
1. 输入类型,  输入参数的约束
1. 联合类型, 可用于  返回具有相似结构的不同对象时
1. 接口, 可用于  返回具有相似结构的不同对象时
1. 标量, 类型的叶子节点, (Int,String, Float, Boolean, ID, 自定义标量类型<需实现序列化、反序列化和验证>)
1. 枚举, 特殊的标量,
1. 根类型(Query/Mutation/Subscripion)
1. 列表/非空, [] !

#### 2. client 查询语法

查询时并行执行, 变更是现行执行, 确保不会出现竞态.

##### 别名

```javascript
{
    myCat:cat(id:"1001"){
        name
        age
    }
    samuelCat:cat(id:"1002"){
        name
        age
    }
}
```

##### 片段

可复用单元, 片段可使用访问查询/变更中的变量, 片段不能引用自身

```javascript

query MyPet($id:ID!){
    myCat:cat(master:$id){
        ...petFields
    }
    myDog:dog(master:$id){
        ...petFields
    }
}
fragment petFields on Pet{
    name
    age
}
{id:"101"}
```

##### 操作名

显示声明操作名可定义变量传入操作内

```javascript
// 查询简写语法, 省略了 操作类型 query 和操作名, 无法为参数提供变量
{
  cat(id:"1001"){
    name
    age
  };
}
// 显示操作类型, 可提供操作名和定义变量
query queryMyCat($id:ID!){
    cat(id:$id){
        name
        age
        master
    }
}
{id:"1001"}
```

##### 变量

参数为变量, 动态传入

```javascript
// 变量前缀$, 类型必须为标量、枚举型或输入对象类型
query queryCat($id: ID!) {
  cat(id: $id) {
    name
    master
    age
  }
}
{id:"1001"}
```

##### 内联片段

字段返回联合类型/接口时, 使用内联片段确定需要的字段

```javscript

query queryAnimal{
    id
    ... on Cat{
        miaomiao
    }
    ... on Dog{
        wangwang
    }
}
```

##### 指令

```javascript
@include(if:boolean){} // 为 true 时包含字段

//  Exapmle
query($queryUser){
    @include(if:$queryUser){
        // queryUser 为 true 时包含该字段
        name
        email
    }
}
{
    queryUser:false
}


@skip(if:boolean){} // 为 true 时不包含该字段
```

#### 3. 执行流程

1. 发送查询操作
2. 服务器验证操作是否有效
3. 验证后执行并返回对应结果

#### 4. 内省

### 三 demo

### 四 不足与应对

防范大量查询, 恶意查询

- 设置请求超时时间,
- 限制返回数据量, 如分页设置每页条数
- 设置查询深度
- 限制查询复杂度
