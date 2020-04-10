RESTful API
=========================

> 记录日常工作中的一些问题

## 介绍

RESTful API 是目前比较成熟的一套互联网程序的的 API 设计理论，并且被绝大多数的互联网开发人员所掌握。

1. 高性能，低成本（服务器可伸缩性）
2. 接口无状态（服务可靠，容易故障恢复）
3. 接口统一，客户端和服务器之间的通信方法是统一化的。
4. 编写接口文档更清晰可读。

## 设计

> 本章节内容来自阮一峰 《[RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)》

#### 1. 域名（Domain）

应该尽量将 API 部署在专用域名之下

- http://api.example.com

也可以考虑放在主域名之下

- http://example.com/api/

#### 2. 版本（Versioning）

应该将 API 的版本号放入到 URL

- http://api.example.com/v1/

#### 3. 路径（Endpoint）

路径又称为“终点”，表示 API 的具体地址，在 RESTful 协议中，每一个网址代表一种资源的转移。**网址中不要应该出现动词，只能使用名词，并且名词往往与数据库表名对应。**

- https://api.example.com/v1/zoos
- https://api.example.com/v1/animals
- https://api.example.com/v1/employees

#### 4. HTTP 动词

常用的HTTP动词有下面五个（括号里是对应的SQL命令）。

- GET（SELECT）：从服务器取出资源（一项或多项）。
- POST（CREATE）：在服务器新建一个资源。
- PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
- PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
- DELETE（DELETE）：从服务器删除资源。

还有两个不常用的HTTP动词。

- HEAD：获取资源的元数据。
- OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。

下面是一些例子。

- GET /zoos：列出所有动物园
- POST /zoos：新建一个动物园
- GET /zoos/ID：获取某个指定动物园的信息
- PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
- PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
- DELETE /zoos/ID：删除某个动物园
- GET /zoos/ID/animals：列出某个指定动物园的所有动物
- DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物

#### 5. 过滤信息（Filtering）

如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。

下面是一些常见的参数。

- ?limit=10：指定返回记录的数量
- ?offset=10：指定返回记录的开始位置。
- ?page=2&per_page=100：指定第几页，以及每页的记录数。
- ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
- ?animal_type_id=1：指定筛选条件

参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，GET /zoo/ID/animals 与 GET /animals?zoo_id=ID 的含义是相同的。

####6. 状态码（Status Codes）

服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）。

- 200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
- 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
- 202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
- 204 NO CONTENT - [DELETE]：用户删除数据成功。
- 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
- 401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。*
- *403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
- 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
- 406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
- 410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
- 422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
- 500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。

状态码的完全列表参见[这里](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)。

#### 7. 错误处理（Error handling）

如果状态码是4xx，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可。

```json
{
    error: "Invalid API key"
}
```

#### 8. 返回结果

针对不同操作，服务器向用户返回的结果应该符合以下规范。

- GET /collection：返回资源对象的列表（数组）
- GET /collection/resource：返回单个资源对象
- POST /collection：返回新生成的资源对象
- PUT /collection/resource：返回完整的资源对象
- PATCH /collection/resource：返回完整的资源对象
- DELETE /collection/resource：返回一个空文档

## 安全

### Security Header

- X-Frame-Options
- X-Content-Type-Options
- X-XSS-Protection
- Strict-Transport-Security

### UUID

各种资源 ID 是 RESTful API 中很重要的一个组成部分，大多数情况下都是用数字来表示的，这会在不经意间泄露了业务信息，例如：泄漏注册用户数量，订单数量等等。解决办法就是不使用按序递增的数字作为 ID，而是使用具有随机性，唯一性，不可预测性的值作为ID，最常见的做法就是使用 UUID。

### 数据泄漏

在前后端分离的情况下，两者会用 JSON 作为数据传输的主体，但有的时候因为疏忽大意，后端 API 返回的 JSON 数据中包含了远远超过前端代码需要的数据，造成数据的泄漏，例如：用户密码。

###速率限制

请求速率限制，根据 api_key 或者用户来判断某段时间的请求次数，将该数据更新到内存数据库（redis，memcached），达到最大数即不接受该用户的请求，同时这样还可以利用到内存数据库 key 在特定时间自动过期的特性。也可以通过HTTP 响应传递限速信息

| 首部名                | 说明               | 类型              |
| --------------------- | ------------------ | ----------------- |
| X-RateLimit-Limit     | 单位时间的访问上限 | Integer           |
| X-RateLimit-Remaining | 剩余的访问次数     | Integer           |
| X-RateLimit-Reset     | 访问次数重置时间   | UTC epoch seconds |

参考文章：https://cnodejs.org/topic/5be8dcf82fed25406c25d805

### 防止数据篡改

### 防止重放攻击

