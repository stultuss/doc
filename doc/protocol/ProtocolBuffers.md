
Protocol Buffers
=========================

> 本文主要介绍 Protocol Buffers 的安装使用，以及语法结构，并结合 gRPC 来讲解如何应用 .proto 文件

## 一. 简介

Protocol Buffer下文简称 protobuf，是由 Google 公司开发的一款高效且轻量的结构化数据存储格式。它的性能比 JSON，XML 等目前常用的数据交换格式，具有更高的转换效率和时间效率。Protocol Buffers 目前有两个版本：proto2 和 proto3

开源项目：[Protocol Buffer](https://github.com/google/protobuf)

性能测试：[Protocol Buffer Benchmarking](https://github.com/eishay/jvm-serializers/wiki)

## 二. 为什么使用Protocol Buffer

除了我们在前面讲到的 protobuf 拥有非常好的性能之外，还有以下几个优点：

1. gRPC 使用 protobuf 非常高效，允许使用 protobuf 定义服务以及使用 protobuf 进行数据交换。
2. 通过 protobuf 定义的 .proto 文件，允许通过 protobuf 编译器生成不同编程语言的代码来读写这个数据结构，目前 protobuf  提供了多种编程语言的支持。

> 总结：由于微服务架构的跨平台特性，以及 gRPC 对其的良好支持，所以使用 protobuf 可以作为微服务之间进行数据交换的标准之一

## 三. 编写 .proto 文件

> 由于 gRPC 仅支持 proto3，所以本文以 proto3 版本来进行讲解

### 文件名

一个比较好的习惯是认真对待 proto 文件的文件名。比如将命名规则定于如下：

`{packageName}.{MessageName}.proto`

### 结构体

Protocol Buffer 中有两种结构体。

#### 1. Message 结构体

可以使用 Message 定义程序中需要处理的结构化数据。类似 java 和 C 语言的数据定义。

```protobuf
syntax = "proto3";

package book;

message BookStruct 
{ 
	int64 isbn = 1;
    string title = 2;
    string author = 3;
}
```

在上例中，package 名字叫做 book，定义了一个结构体 BookStruct，该消息有三个成员，类型为 int64 的 isbn，一个类型为 string 的 title，还有一个类型未 string 的 author。这三个成员都是可选的。

> 只有 proto2 版本才允许通过 required 和 optional 配置成员是否可选。

#### 2. Service结构体（定义一个 RPC 接口）

如果想要将消息类型用在RPC中，可以在 .proto 文件中定义一个 RPC 服务接口，protobuf 编译器将会根据所选择的不同语言生成对应语言的接口代码以及Stub。

```protobuf
service BookService {
    rpc PostBook (BookStruct) returns (BookStruct) {}
}
```

#### 3. Service结构体（定义一个 HTTP 接口） 

在实际应用中，我们不但需要定义 RPC 接口，还需要定义 HTTP 接口。

通过 Google 提供的标准接口 `google/api/annotations.proto` ，我们可以对 protobuf 服务描述其相应的 HTTP接口形式。通过使用 proto 编译器可以生成相应的 HTTP JSON 的接口实现。

```protobuf
import "google/api/annotations.proto";

service BookService {
    rpc PostBookApi (BookStruct) returns (BookStruct) {
        option (google.api.http) = {
            post: "/v1/postBook"
            body: "*"
        };
    }
}
```

![gRPC-Gateway](https://github.com/stultuss/doc/blob/master/images/protocol/ProtocolBuffers.png?raw=true)

### 更多参考

[Protobuf3 语法指南](http://colobu.com/2017/03/16/Protobuf3-language-guide/)

[Google Protocol Buffer 的使用和原理](https://www.ibm.com/developerworks/cn/linux/l-cn-gpb/)

## 四. 编译 .proto 文件，生成代码

### 安装 protoc 编译器

随 Google Protocol Buffer 源代码一起发布的编译器 protoc，支持多种编程语言。但使用 Google Protocol Buffer 的 Compiler 包，您可以开发出支持其他语言的新的编译器。

| Language                             | Source                                                       |
| ------------------------------------ | ------------------------------------------------------------ |
| C++ (include C++ runtime and protoc) | [src](https://github.com/google/protobuf/blob/master/src)    |
| Java                                 | [java](https://github.com/google/protobuf/blob/master/java)  |
| Python                               | [python](https://github.com/google/protobuf/blob/master/python) |
| Objective-C                          | [objectivec](https://github.com/google/protobuf/blob/master/objectivec) |
| C#                                   | [csharp](https://github.com/google/protobuf/blob/master/csharp) |
| JavaScript                           | [js](https://github.com/google/protobuf/blob/master/js)      |
| Ruby                                 | [ruby](https://github.com/google/protobuf/blob/master/ruby)  |
| Go                                   | [golang/protobuf](https://github.com/golang/protobuf)        |
| PHP                                  | [php](https://github.com/google/protobuf/blob/master/php)    |
| Dart                                 | [dart-lang/protobuf](https://github.com/dart-lang/protobuf)  |

#### Protoc 命令

安装好 protoc 编译器后，就可以使用 protoc 命令了。

```bash
protoc -h
Usage: /usr/bin/protoc [OPTION] PROTO_FILES
Parse PROTO_FILES and generate output based on the options given:
  -IPATH, --proto_path=PATH   Specify the directory in which to search for
                              imports.  May be specified multiple times;
                              directories will be searched in order.  If not
                              given, the current working directory is used.
  --version                   Show version info and exit.
  -h, --help                  Show this text and exit.
  --encode=MESSAGE_TYPE       Read a text-format message of the given type
                              from standard input and write it in binary
                              to standard output.  The message type must
                              be defined in PROTO_FILES or their imports.
  --decode=MESSAGE_TYPE       Read a binary message of the given type from
                              standard input and write it in text format
                              to standard output.  The message type must
                              be defined in PROTO_FILES or their imports.
  --decode_raw                Read an arbitrary protocol message from
                              standard input and write the raw tag/value
                              pairs in text format to standard output.  No
                              PROTO_FILES should be given when using this
                              flag.
  --descriptor_set_in=FILES   Specifies a delimited list of FILES
                              each containing a FileDescriptorSet (a
                              protocol buffer defined in descriptor.proto).
                              The FileDescriptor for each of the PROTO_FILES
                              provided will be loaded from these
                              FileDescriptorSets. If a FileDescriptor
                              appears multiple times, the first occurrence
                              will be used.
  -oFILE,                     Writes a FileDescriptorSet (a protocol buffer,
    --descriptor_set_out=FILE defined in descriptor.proto) containing all of
                              the input files to FILE.
  --include_imports           When using --descriptor_set_out, also include
                              all dependencies of the input files in the
                              set, so that the set is self-contained.
  --include_source_info       When using --descriptor_set_out, do not strip
                              SourceCodeInfo from the FileDescriptorProto.
                              This results in vastly larger descriptors that
                              include information about the original
                              location of each decl in the source file as
                              well as surrounding comments.
  --dependency_out=FILE       Write a dependency output file in the format
                              expected by make. This writes the transitive
                              set of input file paths to FILE
  --error_format=FORMAT       Set the format in which to print errors.
                              FORMAT may be 'gcc' (the default) or 'msvs'
                              (Microsoft Visual Studio format).
  --print_free_field_numbers  Print the free field numbers of the messages
                              defined in the given proto files. Groups share
                              the same field number space with the parent
                              message. Extension ranges are counted as
                              occupied fields numbers.

  --plugin=EXECUTABLE         Specifies a plugin executable to use.
                              Normally, protoc searches the PATH for
                              plugins, but you may specify additional
                              executables not in the path using this flag.
                              Additionally, EXECUTABLE may be of the form
                              NAME=PATH, in which case the given plugin name
                              is mapped to the given executable even if
                              the executable's own name differs.
  --cpp_out=OUT_DIR           Generate C++ header and source.
  --csharp_out=OUT_DIR        Generate C# source file.
  --java_out=OUT_DIR          Generate Java source file.
  --javanano_out=OUT_DIR      Generate Java Nano source file.
  --js_out=OUT_DIR            Generate JavaScript source.
  --objc_out=OUT_DIR          Generate Objective C header and source.
  --php_out=OUT_DIR           Generate PHP source file.
  --python_out=OUT_DIR        Generate Python source file.
  --ruby_out=OUT_DIR          Generate Ruby source file.
```

从上述帮助文件中，我们可以看到 protoc 命令可以添加以下参数

- --cpp_out 导出 C++ 代码
- --csharp_out 导出 C#代码
- --java_out 导出 Java 代码
- --javanano_out 导出 Javanano 代码
- --js_out 导出 JavaScript 代码
- --objc_out 导出 ObjectC 代码
- --php_out 导出 PHP 代码
- --python_out 导出 Python 代码
- --ruby_out 导出 Ruby 代码

举例：

```bash
protoc \
--proto_path=IMPORT_PATH \
--cpp_out=DST_DIR \
--java_out=DST_DIR \
--python_out=DST_DIR \
IMPORT_PATH/file.proto
```

#### Protoc 命令的 plugin 参数

除了可以导出上述代码之外，我们还可以使用 plugin 参数加载第三方插件，导出其他任何我们想要的代码。例如：Swagger JSON Schema。

举例：

```bash
# Install protoc plugin:protoc-gen-swagger
go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger

# Export swagger json schema
protoc \
--plugin=protoc-gen-swagger=`which protoc-gen-swagger`
--proto_path=IMPORT_PATH \
--swagger_out=DST_DIR \
IMPORT_PATH/file.proto
```

### 生成 TypeScript 代码

如上列表显示，Google 官方没有直接提供 TypeScript 的编译器，所我根据 Google Protocol Buffer 的 Compiler 包自己写的一个命令行工具：[stultuss/protoc-gen-grpc-ts](https://github.com/stultuss/protoc-gen-grpc-ts)

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
