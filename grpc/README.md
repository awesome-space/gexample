## grpc 环境搭建

grpc 采用 `proto buffer` 进行服务定义以及数据编码，所以我们要安装 `proto buffer`

### proto buffer 安装
官网：https://protobuf.dev/

安装 protoc https://github.com/protocolbuffers/protobuf/releases

![](./assets/protoc.png)

### 安装 grpc 

官网：https://grpc.io/

安装对应工具：

![](./assets/grpc.png) 

选择 Go 语言


## grpc 学习

## 编写 proto 文件

[点击查看 proto 文件](proto/ping.proto)

## 生成 message 代码

protoc --go_out=. ping.proto 


[点击查看 ping.db.go 文件](server/ping.pb.go)

## 生成 rpc 代码

protoc --go-rpc_out=. ping.proto

[点击查看 ping_grpc.db.go 文件](server/ping_grpc.pb.go)


