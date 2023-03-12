
gRPC是一款高性能、开源的远程过程调用（RPC）框架，由Google开发，支持多种编程语言和平台。

gRPC使用 Protocol Buffers 作为其默认的序列化协议，支持多种编程语言，如C++、Java、Python、Go等。


grpc 官网：https://grpc.io/

![](./assets/grpc.png)


golang 使用 grpc 快速开始：
https://grpc.io/docs/languages/go/quickstart/



**环境搭建**

1. 需要 go 环境
2. 需要安装 proto buffer 编译器 protoc 

    官网：https://protobuf.dev/

    安装 protoc https://github.com/protocolbuffers/protobuf/releases

    ![](./assets/protoc.png)

3. 安装 go 代码生成器

    ```shell
    go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
    go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
    ```


编写 proto 文件


## 编写 proto 文件

[点击查看 proto 文件](proto/ping.proto)


## 生成 message 代码

protoc --go_out=. ping.proto 


[点击查看 ping.db.go 文件](server/ping.pb.go)

## 生成 rpc 代码

protoc --go-rpc_out=. ping.proto

[点击查看 ping_grpc.db.go 文件](server/ping_grpc.pb.go)


