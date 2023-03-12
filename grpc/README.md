> <a href="https://grpc.io/docs/languages/go/quickstart/" target="_blank">【Go 语言】 grpc 快速开始</a>
<br>
<a href="https://grpc.io/docs/languages/go/basics/#defining-the-service" target="_blank">【Go 语言】 service 编写参考</a>
<br>
<a href="https://protobuf.dev/getting-started/gotutorial/" target="_blank">【Go 语言】 message 消息编写参考</a>


利用 grpc 写一个 HelloWorld 服务，HellowWorld 服务中包含一个 SayHello 的方法，接受一个 name 参数，返回一个 massage

### proto 文件

```protobuf
// 声明语法版本
syntax = "proto3";


package helloworld;

// 生成的 go 文件放在的位置以及包名
// "path;package_name" path :生成的文件存放的位置，
// package_name 指定生成的代码的包名
option go_package = "grpc";


// 定义 rpc 服务名称
service HelloWorld {
  // 定义服务提供的方法签名
  rpc SayHello(HelloRequest) returns (HelloReply){}
}


// SayHello 的参数
message HelloRequest {
  string name = 1;
}


// SayHello 的返回值
message HelloReply {
  string message = 1;
}
```

### 使用 protoc 生成 grpc 代码

```shell
# 在 grpc 目录下执行
protoc --go_out=./ --go-grpc_out=./ .\service\helloworld\hello_world.proto

```
