# 理解REST
REST（Represenational State Transfer）是2000年Roy Fielding 在他的[博士论文](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)中被宣布和定义。REST 是一种设计分布式系统的架构风格。它不是标准而是一种约束，像无状态、拥有客户端/服务器的关系和统一的接口。REST 与 HTTP 并没有严格意义上的关系，但通常与它相关。
## REST的原则
- *资源* 暴露出容易理解的目录结构URI
- *表现层* 将 JSON 或者 XML 转换成表现对象和特征
- *消息* 明确使用 HTTP 方式（例如GET、POST、PUT和DELETE）
- *无状态* 在客户端和服务器请求之间无上下文交互存储。状态依赖性限制和约束可测量性。客户端保持会话状态。

## HTTP 方式
使用 HTTP 方式去映射 CRUD（创建、检索、更新、删除）操作的 HTTP 请求。

### GET
检索信息。GET 请求必须是安全且[幂等](https://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning)，意味着不管重复多少次相同参数（请求），结果都是一样的。它们可能会有用户并不期待的副作用，因此它们系统的运行并非是必要的。请求也可以是部分的或者有条件的。

检索ID为1的地址：
```
GET /addresses/1
```
### POST 
请求URI资源与所提供的实体做一些事情，POST 常常用来去创建实体，而且也能用来去更新实体。

创建新的地址：
```
POST /addresses
```
### PUT
在URI上存储一个实体。PUT 能创建一个新的实体或者更新已经存在的一个实体。一个 PUT 请求是幂等的。幂等性是 PUT 预期与 POST 请求相对的主要区别。

修改ID为1的地址：
```
PUT /addresses/1
```

>提醒：PUT 会替换已经存在的实体。如果只提供数据元素的子集，那么剩下的部分将会替换成空或者null。

### PATCH
只在URI上更新指定的实体字段。一个 PATCH 请求是幂等的。幂等性是 PUT 预期与 POST 请求相对的主要区别。
```
PATCH /addresses/1
```

### DELETE
资源移除请求；然而资源不会立即被移除。它可能是异步的或者长时间运行的请求。

删除ID为1的地址
```
DELETE /addresses/1
```

### HTTP 状态码

状态码代表着 HTTP 请求的结果
- *1XX*- 信息
- *2XX*- 成功
- *3XX*- 重定向
- *4XX*- 客户端错误
- *5XX*- 服务器错误

### 媒体类型
HTTP头``Accpet``和``Content-Type`` 能够用来描述 HTTP 请求中发送或者请求的内容。如果请求响应的是JSON格式，客户端可能会设置``Accept``为``application/json``。相反的。当发送数据时，设置``Content-Type``为``application/xml``来告诉客户端要发送的请求时XML。

原文连接： https://spring.io/understanding/REST
转载请注明来源