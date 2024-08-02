
#### 消息编码：
客户端发送的请求包括服务名 Arith，方法名 Multiply，参数 args 三个，服务端的响应包括错误 error，返回值 reply 2 个。我们将请求和响应中的参数和返回值抽象为 body，剩余的信息放在 header 中，那么就可以抽象出数据结构 Header：
```
package codec

import "io"

type Header struct {
    ServiceMethod string // format "Service.Method"
    Seq           uint64 // sequence number chosen by client
    Error         string
}
```
ServiceMethod 是服务名和方法名，通常与 Go 语言中的结构体和方法相映射。
Seq 是请求的序号，也可以认为是某个请求的 ID，用来区分不同的请求。
Error 是错误信息，客户端置为空，服务端如果如果发生错误，将错误信息置于 Error 中。
我们将和消息编解码相关的代码都放到 codec 子目录中，在此之前，还需要在根目录下使用 go mod init geerpc 初始化项目，方便后续子 package 之间的引用。 进一步，抽象出对消息体进行编解码的接口 Codec，抽象出接口是为了实现不同的 Codec 实例
```
type Codec interface {
    io.Closer
    ReadHeader(*Header) error
    ReadBody(interface{}) error
    Write(*Header, interface{}) error
}
```
紧接着，抽象出 Codec 的构造函数，客户端和服务端可以通过 Codec 的 Type 得到构造函数，从而创建 Codec 实例。这部分代码和工厂模式类似，与工厂模式不同的是，返回的是构造函数，而非实例。

```
type NewCodecFunc func(io.ReadWriteCloser) Codec

type Type string

const (
	GobType  Type = "application/gob"
	JsonType Type = "application/json" // not implemented
)

var NewCodecFuncMap map[Type]NewCodecFunc

func init() {
	NewCodecFuncMap = make(map[Type]NewCodecFunc)
	NewCodecFuncMap[GobType] = NewGobCodec
}

```
我们定义了 2 种 Codec，Gob 和 Json，但是实际代码中只实现了 Gob 一种，事实上，2 者的实现非常接近，甚至只需要把 gob 换成 json 即可。

首先定义 GobCodec 结构体，这个结构体由四部分构成，conn 是由构建函数传入，通常是通过 TCP 或者 Unix 建立 socket 时得到的链接实例，dec 和 enc 对应 gob 的 Decoder 和 Encoder，buf 是为了防止阻塞而创建的带缓冲的 Writer，一般这么做能提升性能。

```
package codec

import (
    "bufio"
    "encoding/gob"
    "io"
    "log"
)

type GobCodec struct {
    conn io.ReadWriteCloser
    buf  *bufio.Writer
    dec  *gob.Decoder
    enc  *gob.Encoder
}

var _ Codec = (*GobCodec)(nil)

func NewGobCodec(conn io.ReadWriteCloser) Codec {
    buf := bufio.NewWriter(conn)
        return &GobCodec{
            conn: conn,
            buf:  buf,
            dec:  gob.NewDecoder(conn),
            enc:  gob.NewEncoder(buf),
    }
}
```


参考：七天系列：geeRPC

