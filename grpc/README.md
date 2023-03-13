参考文章 :
1. <a href="https://grpc.io/docs/languages/go/quickstart/" target="_blank">【Go 语言】 grpc 快速开始</a>
2. <a href="https://grpc.io/docs/languages/go/basics/#defining-the-service" target="_blank">【Go 语言】 service 编写参考</a>
3. <a href="https://protobuf.dev/getting-started/gotutorial/" target="_blank">【Go 语言】 message 消息编写参考</a>

gRPC 使用 protobuf 语言编写，它定义了 gRPC 服务的消息类型、RPC 方法和服务接口。


## 案例：

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

执行命令后会生成两个文件，一个是  <a href="./service/helloworld/hello_world.pb.go">hello_world.pb.go</a> ; 一个是 <a href="./service/helloworld/hello_world_grpc.pb.go">hello_world_grpc.pb.go</a>


`hello_world.pb.go` 文件里面是 `hello_world.proto` 中 message 在 go 语言上的具体实现。

```go
type HelloRequest struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Name string `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`
}

type HelloReply struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Message string `protobuf:"bytes,1,opt,name=message,proto3" json:"message,omitempty"`
}
```

`hello_world_grpc.pb.go` 文件里面是 `hello_world.proto` 中 service 在 go 语言上的具体实现，里面包含客户端以及服务端的调用 api。

```go
// service 中定义的服务
type HelloWorldServer interface {
	// service 中定义的方法签名
	SayHello(context.Context, *HelloRequest) (*HelloReply, error)
	mustEmbedUnimplementedHelloWorldServer()
}

// UnimplementedHelloWorldServer must be embedded to have forward compatible implementations.
// 这是一个未实现功能的一个 HelloWorldServer 服务
// 我们可以在 UnimplementedHelloWorldServer 中完善方法的实现，这样会破坏生成的文件
// 如果我们的服务如果还有迭代的可能，在下一次生成 grpc 代码时有被覆盖的风险
// 也可以在另外的包或者文件里定义一个新的 struct 嵌入 UnimplementedHelloWorldServer 将具体实现移交给子结构体，不破坏生成的文件，
// 之后再次生成 grpc 代码时，不会被覆盖，推荐这种方式
type UnimplementedHelloWorldServer struct {
}

// service 中定义的方法具体实现
func (UnimplementedHelloWorldServer) SayHello(context.Context, *HelloRequest) (*HelloReply, error) {
	return nil, status.Errorf(codes.Unimplemented, "method SayHello not implemented")
}

// UnsafeHelloWorldServer 接口的实现
func (UnimplementedHelloWorldServer) mustEmbedUnimplementedHelloWorldServer() {}

// UnsafeHelloWorldServer may be embedded to opt out of forward compatibility for this service.
// Use of this interface is not recommended, as added methods to HelloWorldServer will
// result in compilation errors.
// 如果一个 struct 内嵌 HelloWorldServer 后，没有实现 SayHello 方法，那么客户端调用SayGoodbye方法时就会返回 “Unimplemented” 错误。
// 为了避免这种情况，实现此接口，那么调用 SayHello 方法时就会返回一个默认的响应。
type UnsafeHelloWorldServer interface {
	mustEmbedUnimplementedHelloWorldServer()
}

// 将 HelloWorldServer 注册进 gprc 服务
func RegisterHelloWorldServer(s grpc.ServiceRegistrar, srv HelloWorldServer) {
	s.RegisterService(&HelloWorld_ServiceDesc, srv)
}
```

### 启动 Rpc 服务器

```go

// 嵌入 UnimplementedHelloWorldServer 实现 HelloWorldServer 接口
type HelloWorldServer struct {
	helloworld.UnimplementedHelloWorldServer
}

// 重写 SayHello 方法，覆盖  UnimplementedHelloWorldServer 中的方法
func (s *HelloWorldServer) SayHello(ctx context.Context, in *helloworld.HelloRequest) (*helloworld.HelloReply, error) {
	log.Printf("Received: %v", in.GetName())
	return &helloworld.HelloReply{Message: "Hello " + in.GetName()}, nil
}

func main(){
  // 监听 50051 端口号
  lis, err := net.Listen("tcp", ":50051")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
  // 实例化一个 grpc 服务器
	s := grpc.NewServer()
  // 将 HelloWorldServer 服务注册进服务器
	helloworld.RegisterHelloWorldServer(s, &HelloWorldServer{})
	log.Printf("server listening at %v", lis.Addr())
  // 启动 grpc 服务器
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

### grpc 客户端使用


```go
  // 连接 grpc 服务器
	conn, err := grpc.Dial("localhost:50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
  // 创建一个 Helloworld 服务客户端
	c := helloworld.NewHelloWorldClient(conn)
	// 创建一个有超时功能的 context， 一秒中得不到响应
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()
	r, err := c.SayHello(ctx, &helloworld.HelloRequest{Name: *name})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	log.Printf("Greeting: %s", r.GetMessage())
```


### 运行
依次运行 server/main.go 以及 client/main.go


server output:
2023/03/13 11:39:41 server listening at [::]:50051
2023/03/13 13:31:03 Received: world

client output:
2023/03/13 13:31:03 Greeting: Hello world


