
## go 使用 grpc

<a href="https://grpc.io/docs/languages/go/quickstart/" target="_blank">【Go 语言】 快速开始</a>


## proto 文件

<a href="https://protobuf.dev/getting-started/gotutorial/" target="_blank">【Go 语言】 message 消息编写参考</a>


```protobuf
// 声明语法版本
syntax = "proto3";


// 生成的 go 文件放在的位置以及包名
// "path;package_name" path :生成的文件存放的位置，
// package_name 指定生成的代码的包名
option go_package = "grpc;server";

// 定义 rpc 服务名称
service Ping {
  // 定义服务提供的方法签名
  rpc Ping(PingRequest) returns (PingResponse){}
}


// 定义 rpc 服务所使用的消息
// 会被转化成 type PingRequest struct 
message PingRequest{

}

message PingResponse{
  string msg = 1;
}
```




