
gRPC
=========================

> 本文主要介绍  Google gRPC 的安装使用以及 gRPC 在微服务中的应用。

## 一. 简介

gRPC 是 Google 开源的基于 HTTP/2 协议的高性能通用 RPC 框架，gRPC 使用 protobuf 来定义数据交换的数据结构，并且支持众多热门开发语言，例如：C++，Java，Go，Python，Node.js 等。

开源项目：[grpc](https://github.com/grpc/grpc)

官方文档：[grpc.io](https://grpc.io/)

## 二. 为什么使用 gRPC

首先 Google gRPC 框架具有以下几个特性：

1. 基于 HTTP/2 协议。
2. 基于 protobuf 定义服务和通信。
3. 支持多种语言。

下面就对上面的三种特性进行详细说明。

### 基于 HTTP/2 协议

相较于其他基于 HTTP 1.1 的 RPC 框架，基于 HTTP/2 的标准设计的 gRPC 带来了像单 TCP 连接上的多路复用，数据双流，头部压缩等一系列的新特性，这些特性使得在客户端和服务端的数据交换中具有更高的性能，更快的响应，以及节省更多的流量。

衍生阅读关键字：多路复用，双流

### 基于 protobuf 定义服务

gRPC 支持使用 protobuf 来定义服务，protobuf 的优缺点已经在前面的文章中介绍过了，详见[服务框架技术栈之Protocol Buffer](https://niklaus0823.github.io/2018/05/09/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%A1%86%E6%9E%B6%E6%8A%80%E6%9C%AF%E6%A0%88%E4%B9%8BProtocol-Buffer/)，protobuf 的具有强大的数据压缩特性，使得 gRPC 的性能更上一层楼。

也许你会说用其他的 RPC 框架，同样也可以使用 protobuf，但是不要忘了 gRPC 和 protobuf 同样都是出自 Google 之手，那么势必在未来的版本迭代中 gRPC 将更有优势，例如相对其他框架，更快的版本更新支持。

### 支持多种语言

gRPC 支持多种语言开发，并能够给予语言自动生成客户端和服务端功能库，这个特性在微服务中是非常重要的，这个特性可以使得微服务架构中允许根据需求选择最合适语言的进行开发成为可能。（微服务特性之一：语言无关）

![gRPC](hhttps://github.com/stultuss/doc/blob/master/images/rpc/gRPC.jpg)

## 三. 如何使用 gRPC 

> 为了能够让读者快速了解 gRPC 和 protobuf 是如何工作的，这里只介绍一种通过加载 proto 文件，快速创建 gRPC 客户端和服务端的方法。而在实际的开发过程中，通常笔者更倾向讲 proto 文件转换成 js 文件和 .d.ts 声明文件，再通过 js 内提供的方法创建客户端和服务端。

### 安装 gRPC 模块

```bash
npm install -save grpc@1.9.1
```

### 定义一个 proto 文件

```protobuf
// protos/book.proto
syntax = "proto3";

package book;

message BookStruct
{
    string isbn = 1;
}

service BookService {
    rpc PostBook (BookStruct) returns (BookStruct) {}
}
```

### 创建 gRPC 服务端

```javascript
// src/server.js
const path = require('path');
const grpc = require('grpc');

const PROTO_PATH = path.join(__dirname, '..', 'protos');
const descriptor = grpc.load(path.join(PROTO_PATH, 'book.proto'));
// descriptor 内的结构是 descriptor.${packageName}.${ServiceName}.service

// 创建 gRPC 服务端，并通过读取的 proto 文件的 service 绑定 gRPC 接口监听和执行方法
const server = new grpc.Server();
server.addService(descriptor.book.BookService.service, {
  postBook: function (call, callback) {
    // 尝试改变 book 的 field：isbn 的值类型，如果与 proto 中定义的类型不一致，客户端会报错
    const book = {
  	  isbn: '10086'
	};
    callback(null, book);
  },
});

// 启动 gRPC 服务
server.bind('0.0.0.0:50051', grpc.ServerCredentials.createInsecure());
server.start();
```

### 创建 gRPC 客户端

```javascript
const path = require('path');
const grpc = require('grpc');

const PROTO_PATH = path.join(__dirname, '..', 'protos');
const descriptor = grpc.load(path.join(PROTO_PATH, 'book.proto'));

// 通过读取的 proto 文件创建客户端
const client = new descriptor.book.BookService('localhost:50051', grpc.credentials.createInsecure());

// 向 gRPC 服务器的 postBook 接口发送请求。
const book = {
  isbn: '10086'
};

// 尝试改变 book 的 field: isbn 的值类型，如果与 proto 中定义的类型不一致，客户端会报错。
client.postBook(book, function (err, res) {
  if (err) {
    // process error
    console.log('err', err);
  } else {
    // process feature
    console.log('res', res);
  }
});
```

通过上述几个步骤，一个简单的 gRPC 客户端-服务端的调用就完成了。

需要注意的是，通信方式一共有四种，这里只介绍了 **UnaryCall** 一种：

- UnaryCall，直接通信，客户端发送普通消息，服务端处理完成后返回消息
- ServerReadableStream，服务端读流，客户端发送流消息，服务端接收完毕后并处理，再返回消息
- ServerWriteableStream，服务端写流，客户端发送普通消息，服务端发送流消息，直到服务端消息发送完。
- DuplexStream，双流，客户端发送流消息，服务端边接收，边返回流消息，直到服务端消息发送完。

## 四. 通过工具生成 gRPC 的代码

项目地址：[grpc-tools](https://github.com/grpc/grpc/tree/master/examples/node/static_codegen)

通过 npm 将 grpc-tools 安装到全局模块后，就可以使用 grpc-tools 命令了。

```bash
npm install -g grpc-tools

grpc_tools_node_protoc \
	--js_out=import_style=commonjs,binary:./output/ \
	--grpc_out=./output \
	--plugin=protoc-gen-grpc=`which grpc_tools_node_protoc_plugin` \
	./protos/book.proto
```

通过上面的命令就可以将 proto 文件在 `./output` 目录下生成 JavaScript 文件，JavaScript 文件分为两部分：

1. proto 文件内定义的 message 结构对应的 JavaScript 文件（即：gRPC 消息结构体）
2. proto 文件内定义的 service 结构对应的 JavaScript 文件（即：gRPC 服务端和客户端代码）

#### 其他工具

Google 官方没有直接提供 TypeScript 的编译器，所以我根据 Google Protocol Buffer 的 Compiler 包自己写的一个命令行工具：[stultuss/protoc-gen-grpc-tc](https://github.com/stultuss/protoc-gen-grpc-ts)

安装方式如下：

```bash
npm config set unsafe-perm true
npm install protoc-gen-grpc -g 
```

> 具体 protoc-gen-grpc 的其他功能，请查阅项目 README，该工具已经封装了 gRPC Service Stub 的代码导出，与原版 protoc 的命令行略有不同

#### 生成代码

```bash
# generate js codes
protoc-gen-grpc \
--js_out=import_style=commonjs,binary:./examples/src \
--grpc_out=./examples/src \
--proto_path ./examples/proto \
./examples/proto/book.proto

# generate d.ts codes
protoc-gen-grpc-ts \
--ts_out=service=true:./examples/src \
--proto_path ./examples/proto \
./examples/proto/book.proto
```