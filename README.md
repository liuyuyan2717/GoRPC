# GoRPC
RPC(Remote Procedure Call，远程过程调用)是一种计算机通信协议，允许调用不同进程空间的程序。RPC 的客户端和服务器可以在一台机器上，也可以在不同的机器上。程序员使用时，就像调用本地程序一样，无需关注内部的实现细节。

不同的应用程序之间的通信方式有很多，比如浏览器和服务器之间广泛使用的基于 HTTP 协议的 Restful API。与 RPC 相比，Restful API 有相对统一的标准，因而更通用，兼容性更好，支持不同的语言。HTTP 协议是基于文本的，一般具备更好的可读性。但是缺点也很明显：

Restful 接口需要额外的定义，无论是客户端还是服务端，都需要额外的代码来处理，而 RPC 调用则更接近于直接调用。
基于 HTTP 协议的 Restful 报文冗余，承载了过多的无效信息，而 RPC 通常使用自定义的协议格式，减少冗余报文。
RPC 可以采用更高效的序列化协议，将文本转为二进制传输，获得更高的性能。
因为 RPC 的灵活性，所以更容易扩展和集成诸如注册中心、负载均衡等功能。

Go 语言广泛地应用于云计算和微服务，成熟的 RPC 框架和微服务框架汗牛充栋。grpc、rpcx、go-micro 等都是非常成熟的框架。一般而言，RPC 是微服务框架的一个子集，微服务框架可以自己实现 RPC 部分，当然，也可以选择不同的 RPC 框架作为通信基座。

考虑性能和功能，上述成熟的框架代码量都比较庞大，而且通常和第三方库，例如 protobuf、etcd、zookeeper 等有比较深的耦合，难以直观地窥视框架的本质。本项目的目的是以最少的代码，实现 RPC 框架中最为重要的部分，理解 RPC 框架在设计时需要考虑什么。

