# 1. Well REST API 

![](https://wdd.js.org/images/20180607225013_Kuay0l_content_rest_api_design.jpeg)


# 2. 基本原则

## 2.1. 关注点分离


> 关注点分离（Separation of concerns，SOC）是对只与“特定概念、目标”（关注点）相关联的软件组成部分进行“标识、封装和操纵”的能力，即标识、封装和操纵关注点的能力。是处理复杂性的一个原则。由于关注点混杂在一起会导致复杂性大大增加，所以能够把不同的关注点分离开来，分别处理就是处理复杂性的一个原则，一种方法。-- [维基百科](https://zh.wikipedia.org/wiki/%E5%85%B3%E6%B3%A8%E7%82%B9%E5%88%86%E7%A6%BB)

- 大问题拆分成个个小问题
- 万能接口拆分成独立的小接口

![Jietu20180830-135439.jpg](https://i.loli.net/2018/08/30/5b8786ac31523.jpg)

## 2.2. 一致性
- **URL风格** URL风格有以下三种，
  - 驼峰式: userData
  - 中划线式: user-data
  - 下划线式: user_data



## 2.3. 版本管理

版本管理一般有两种

- 位于url中的版本标识： http://example.com/api/v1/
- 位于请求头中的版本标识：Accept: application/vnd.redkavasyl+json; version=2.0

# 3. 请求

![](https://wdd.js.org/images/20180607224524_M1yRtD_content_api_for_restful_web_services.jpeg)

## 3.1. 最小化路径嵌套

过深的路径嵌套，会导致资源之间的关系显得非常混乱。

最多3层嵌套为佳，超过4层，则要考虑拆分接口。

```
// bad
/orgs/{org_id}/apps/{app_id}/dynos/{dyno_id}

// good
/orgs/{org_id}
/orgs/{org_id}/apps
/apps/{app_id}
/apps/{app_id}/dynos
/dynos/{dyno_id}
```

## 3.2. URL参数优先原则

HTTP可以在三个位置携带信息

1. **URL** 包括 path, 查询字符串
2. **headers** 包括请求头、响应头
3. **body** 包括请求体、响应体

```
/call/:callId/hold
/call/callId?action=hold
```

非敏感参数放在URL中以下好处

1. **分析日志会更方便** 一般日志系统都会打印出请求的url, 而不会打印body
2. **减少请求体重传输的数据量**

## 3.3. 资源名

- 资源名建议使用**复数形式**
- **建议使用字典中能查到单词**，不要随意起名字

```
/calls/
/users/
/agents/
```

## 3.4. 请求方法

理解请求方法在http, sql, file的深层含义

层次 | 创建 | 查询 | 修改 | 删除
---|---|---|---|---
HTTP(web层) | POST | GET | PUT | DELETE
SQL(数据库层) | INSERT | SELECT | UPDATE | DELETE
FILE(文件系统层) | CREATE | READ | UPDATE | DELETE

![](https://wdd.js.org/images/20180612085022_UHL82x_content_request_methods.jpeg)

## 3.5. 查询参数
### 3.5.1. 翻页参数 `page`, `pageSize`

- 一个系统，应当统一翻页参数，例如都叫page, pageSize
- 一个系统，应当统一起始页码，要么都从1开始，要么都从0开始
- pageSize应当有`默认值`和`最大值`限制。

```
/agents?page=0&pageSize=20
```

### 3.5.2. 投影参数 `fields`

restful接口返回的数据格式往往都是写死的，例如查询agent信息，也许你要的只是agent姓名和年龄字段，但是往往获取到一个agent的所有字段信息。

如果无法自定义返回字段，那么响应体往往很多无用的信息。

```
//bad
/agents?minAge=12

[{
  "name":"wdd",
  "age":"12",
  "address":"ss",
  "address":"ss",
  "org":{
    "children": {
      ""
      ....
    }
  }
  ....
},{
  "name":"ddw",
  "age":"12",
  "address":"ss",
  "org":{
    "children": {
      ""
      ....
    }
  }
  ....
}]
```

```
// better 只要获取agent name 和 age字段
/agents?fields=name,age&minAge=12

[{
  "name":"wdd",
  "age":"12"
},{
  "name":"ddw",
  "age":"12"
}]
```

### 3.5.3. 控制器参数 `actions`

对于同一资源，当需要改变资源时，有时候仅仅使用`PUT`，无法准确描述其动作。但是restful风格其实并不建议在path中使用动词。

所以对于此种情况，建议增加action, 将信息放在查询字符串中。

```
// bad
/pets/dogs/:id/actions/run
/pets/dogs/:id/actions/eat
/pets/dogs/:id/actions/bark

// better
/pets/dogs/:id?action=run
/pets/dogs/:id?action=eat
/pets/dogs/:id?action=bark
```

### 3.5.4. 其他字段查询参数

- 字段查询应当在符合业务需求的前提下，尽量少的提供查询维度。
- 查询参数的设计，应当尽可能考虑要`命中索引`。这样才能避免在数据量剧增时，导致查询性能消耗极大。



## 3.6. 使用JSON格式的body

```
$ curl -X POST https://service.com/apps \
    -H "Content-Type: application/json" \
    -d '{"name": "demoapp"}'

{
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "name": "demoapp",
  "owner": {
    "email": "username@example.com",
    "id": "01234567-89ab-cdef-0123-456789abcdef"
  },
  ...
}
```


# 4. 响应
## 4.1. 统一资源名称

同一个资源名称，在不同接口响应体中，应当具有相同的字段名。

例如订单号，在A接口中叫做orderID, 在B接口中叫做orderNo。应当统一成一个字段名称。

## 4.2. 提供Request-Id

一个请求异常了，如果没有一个请求的唯一id, 那么只能按照时间范围去排查日志，在日志量大的情况下，也是很难找到相应日志的。

所以，建议在响应头中加入一个字段`Request-Id`，建议用uuid。在将一个请求写入日志中时，同时写入该请求的request-id。由于每个请求uuid都是唯一的，排查问题时会非常方便。

```
...
Request-Id: 3b99e3e0-7598-11e8-90be-95472fb3ecbd
date: Tue, 28 Aug 2018 13:07:53 GMT
expires: Thu, 19 Nov 1981 08:52:00 GMT
...
```

## 4.3. 合适的状态码

:star:最常用的状态码

- **1xx Informational**
  - 100 Continue
  - 101 Switching Protocols
  - 102 Processing (WebDAV)

- **2xx Success**
  - :star:200 OK
  - :star:201 Created
  - 202 Accepted
  - 203 Non-Authoritative Information
  - :star:204 No Content
  - 205 Reset Content
  - 206 Partial Content
  - 207 Multi-Status (WebDAV)
  - 208 Already Reported (WebDAV)
  - 226 IM Used

- **3xx Redirection**
  - 300 Multiple Choices
  - 301 Moved Permanently
  - 302 Found
  - 303 See Other
  - 304 Not Modified
  - 305 Use Proxy
  - 306 (Unused)
  - 307 Temporary Redirect
  - 308 Permanent Redirect (experimental)

- **4xx Client Error**
  - :star:400 Bad Request
  - :star:401 Unauthorized
  - 402 Payment Required
  - :star:403 Forbidden
  - :star:404 Not Found
  - :star:405 Method Not Allowed
  - 406 Not Acceptable
  - 407 Proxy Authentication Required
  - 408 Request Timeout
  - :star:409 Conflict
  - 410 Gone
  - 411 Length Required
  - 412 Precondition Failed
  - 413 Request Entity Too Large
  - 414 Request-URI Too Long
  - 415 Unsupported Media Type
  - 416 Requested Range Not Satisfiable
  - 417 Expectation Failed
  - 418 I'm a teapot (RFC 2324)
  - 420 Enhance Your Calm (Twitter)
  - 422 Unprocessable Entity (WebDAV)
  - 423 Locked (WebDAV)
  - 424 Failed Dependency (WebDAV)
  - 425 Reserved for WebDAV
  - 426 Upgrade Required
  - 428 Precondition Required
  - 429 Too Many Requests
  - 431 Request Header Fields Too Large
  - 444 No Response (Nginx)
  - 449 Retry With (Microsoft)
  - 450 Blocked by Windows Parental Controls (Microsoft)
  - 451 Unavailable For Legal Reasons
  - 499 Client Closed Request (Nginx)

 
- **5xx Server Error**
  - :star:500 Internal Server Error
  - 501 Not Implemented
  - :star:502 Bad Gateway
  - 503 Service Unavailable
  - :star:504 Gateway Timeout
  - 505 HTTP Version Not Supported
  - 506 Variant Also Negotiates (Experimental)
  - 507 Insufficient Storage (WebDAV)
  - 508 Loop Detected (WebDAV)
  - 509 Bandwidth Limit Exceeded (Apache)
  - 510 Not Extended
  - 511 Network Authentication Required

## 4.4. 统一的错误模型

有时候错误的状态码无法准确描述其错误类型，建议可以提供唯一的枚举类型的id来表明具体错误类型。

- message 只是给开发者看的，不要把这个错误信息直接弹窗告诉最终用户
- 开发者可以根据id字段，翻译成具体的提示信息，告诉最终用户。

```
{
  "id":      "rate_limit",
  "message": "Account reached its API rate limit.",
  "url":     "https://docs.service.com/rate-limits"
}
```

## 4.5. 总是提供资源的唯一id

```
[
  {
    id: '3b99e3e0-7598-11e8-90be-95472fb3ecbd',
    ...
  },
  {
    id: '3b99e3e0-7598-11e8-90be-95472fb3ecbd',
    ...
  }
]
```

## 4.6. 最小化响应体

- 如果页面上不展示该数据，就不要将该数据添加到响应体中
- 不要出现无法预知数组长度的数组嵌套，应当查分接口。而且数组嵌套往往难以分页。
- `避免数组中嵌套无法预知长度的数组`
- `避免数组中嵌套无法预知深度的树`，要避免无法预知数据量大小的响应体

总之，`要能够控制住响应体的大小。`


举个栗子，按天的查询，关于详情的数据应该放在另外一个接口中查询。

日期 | 通话时长 | 详情
---|---|---
1号 | 40分钟 | [查看详情]()
2号 | 40分钟 | [查看详情]()

```
// bad
// GET /bills
// 在账单的数组中，嵌套一个无法预估长度的detail数组
// 如果detail数组过长，将会导致响应缓慢，太长的数据将会导致浏览器截断json
[
  {
    id: '1'
    date: 1, 
    talkLength: 40, 
    detail: [
      {callingDevice: '888', ....},
      {callingDevice: '888', ....},
      {callingDevice: '888', ....},
      {callingDevice: '888', ....},
      {callingDevice: '888', ....},
    ]
  },
  {
    id: '2',
    date: 2, 
    talkLength: 80, 
    detail: [
      {callingDevice: '888', ....},
      {callingDevice: '888', ....},
      {callingDevice: '888', ....},
      {callingDevice: '888', ....},
      {callingDevice: '888', ....},
    ]
  }
]

// better
// GET /bills
// 将detail分离到专门查询detail的接口中
[
  {
    date: 1, 
    talkLength: 40
  },
  {
    date: 2, 
    talkLength: 80
  }
]
// GET /bills/calls?day=2017-09-09&page=1&pageSize=20
[{
  {callingDevice: '888', ....},
  {callingDevice: '888', ....},
  {callingDevice: '888', ....},
  {callingDevice: '888', ....},
  {callingDevice: '888', ....},
}]
```

## 4.7. 不要在请求体中封装状态信息

200 ok 就可以说明请求成功了，没必要在请求体中再额外增加一个状态字段。

```
GET 200 ok
{
  "status": "success",
  "agents": [{....}]
}
```

# 5. 接口文档
## 5.1. 使用curl提供快速可执行例子

不要让用户看了半天文档，也不知道怎么传参数。应当直截了当的给出一个可执行的例子。

```
$ curl -X POST https://service.com/apps \
    -H "Content-Type: application/json" \
    -d '{"name": "demoapp"}'

{
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "name": "demoapp",
  "owner": {
    "email": "username@example.com",
    "id": "01234567-89ab-cdef-0123-456789abcdef"
  },
  ...
}
```

## 5.2. 提供在线文档和离线pdf文档

- **文档网站** 专门的在线文档静态网页，适合对外接口文档
- **swagger** 文档，适合对内接口文档
- **markdown** 文档，方便，快捷，很容易转化成pdf或者其他格式
- **pdf文档** 适合与无法访问外网的开发者

# 6. 深入REST
## 6.1. 理解无状态

`无状态不是真正的无状态，而是将状态集中到专门的状态管理服务中。`

下图所示的状态是有状态的服务，一旦一个请求到达某个服务，如果必须要求后续的请求，也必须到达该服务，那么这个服务就是有状态的。一旦该服务示例挂了，所有的状态也就丢失了。

![](https://wdd.js.org/images/20180612141107_qhgDxn_Jietu20180612-141048.jpeg)

`无状态的服务要求实例不存储状态，把状态交给专门的状态服务器去管理。` 集群中的每个实例能够完全相等，一个实例挂了。后续的请求能够自动被分配到状态正常的实例上去。

![Jietu20180830-133446.jpg](https://i.loli.net/2018/08/30/5b87820d0e558.jpg)


# 7. 参考
- [HTTP API Design Guide](https://geemus.gitbooks.io/http-api-design/content/en/)