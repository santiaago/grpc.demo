# grpc.demo

## why

## how

* install [protoc](https://github.com/google/protobuf/releases/tag/v3.0.0)
* install `protoc-gen-go`
    `go get -u github.com/golang/protobuf/protoc-gen-go`

* create proto file:

~~~proto
syntax = "proto3";

package reverse;

service ReverseService {
    rpc ReverseString (ReverseRequest) returns (ReverseReply) {}
}

message ReverseRequest {
    string data = 1;
}

message ReverseReply {
    string reversed = 2;
}
~~~

* generate go code:

~~~
> GOROOT\src\github.com\santiaago\grpc.demo> protoc -I .\proto\ .\proto\reverse.proto --go_out=plugins=grpc:proto
~~~

This generates a `reverse.pb.go` file. It holds the `ReverseRequest` and `ReverseReply` messages types as well as the `ReverseService` client and server.

* create a go server:

Create a server type that implements the `ReverseString` function and make the server serve the grpc service:

~~~go
package main

import (
	"log"
	"net"

	pb "github.com/santiaago/grpc.demo/proto"

	"golang.org/x/net/context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"
)

const (
	port = ":50051"
)

type server struct{}

func (s *server) ReverseString(ctx context.Context, in *pb.ReverseRequest) (*pb.ReverseReply, error) {
	// your reverse string implementation here
	return &pb.ReverseReply{Reversed: reversed}, nil
}

func main() {
	lis, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	pb.RegisterReverseServiceServer(s, &server{})
	reflection.Register(s)
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to server: %v", err)
	}
}
~~~

* create a Go client:

~~~go
package main

import (
	"log"

	pb "github.com/santiaago/grpc.demo/proto"

	"golang.org/x/net/context"
	"google.golang.org/grpc"
)

const (
	address = "localhost:50051"
)

func main() {
	conn, err := grpc.Dial(address, grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}

	defer conn.Close()
	c := pb.NewReverseServiceClient(conn)
	r, err := c.ReverseString(context.Background(), &pb.ReverseRequest{
		Data: "Hello, world",
	})

	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	log.Println("Reversed string: ", r.Reversed)
}
~~~

Output:

server:
~~~
...\go.server
> go run .\main.go
2016/12/13 14:28:10 transport: http2Server.HandleStreams failed to read frame: read tcp [::1]:50051->[::1]:1195: wsarecv: An existing connection was forcibly closed by the remote host.
~~~

client:
~~~
...\go.client
> go run .\main.go
2016/12/13 14:29:20 Reversed string:  dlrow ,olleH
~~~

## csharp

* install `Grpc.Core`, `Grpc.Tools` and `Google.Protobuf` from NuGet.
* generate c# stub:

~~~
...\csharp.client
> .\packages\Grpc.Tools.1.0.1\tools\windows_x86\protoc.exe -I..\proto --csharp_out . --grpc_out . ..\proto\reverse.proto --plugin=protoc-gen-grpc=packages/Grp
c.Tools.1.0.1/tools/windows_x86/grpc_csharp_plugin.exe
~~~

This generates `Reverse.cs` and `ReverseGrpc.cs` files to include in your project.

Create client:

~~~cs
using System;
using Grpc.Core;
using Reverse;

namespace csharp.client
{
    internal class Program
    {
        private static void Main()
        {
            var channel = new Channel("127.0.0.1:50051", ChannelCredentials.Insecure);

            var client = new ReverseService.ReverseServiceClient(channel);

            var reply = client.ReverseString(new ReverseRequest
            {
                Data = "Hello, World"
            });

            Console.WriteLine("Got: " + reply.Reversed);

            channel.ShutdownAsync().Wait();

            Console.WriteLine("Press any key to exit...");
            Console.ReadKey();
        }
    }
}
~~~

Run it:

~~~
Got: dlroW ,olleH
Press any key to exit...
~~~

## dotnet core:

* create dotnet core project
* add grpc dependencies
* generate csharp files, same as before.
* build
* run

Output

~~~
...\dotnetcore.client
> dotnet run
Project dotnetcore.client (.NETCoreApp,Version=v1.0) was previously compiled. Skipping compilation.
Got: dlroW ,olleH
Press any key to exit...
~~~

## Resources:

* [c# quickstart](http://www.grpc.io/docs/quickstart/csharp.html)
* [gRPC repo](https://github.com/grpc/grpc)
* [protocol buffers - Language Guide](https://developers.google.com/protocol-buffers/docs/proto)
* [gRPC Go Quick start](http://www.grpc.io/docs/quickstart/go.html)
* [Go support for Google's protocol buffers](https://github.com/golang/protobuf)
* [Practical guide to protocol buffers](http://www.minaandrawos.com/2014/05/27/practical-guide-protocol-buffers-protobuf-go-golang/)
* [GothamGo 2015: gRPC: Google's high-performance, open-source RPC framework by Sameer Ajmani](https://www.youtube.com/watch?v=sZx3oZt7LVg)
* [gotham-grpc demo code](https://github.com/golang/talks/tree/master/2015/gotham-grpc)
* [gRPC: a true internet-scale RPC framework is now 1.0 and ready for production deployments](https://cloudplatform.googleblog.com/2016/08/gRPC-a-true-Internet-scale-RPC-framework-is-now-1-and-ready-for-production-deployments.html)
* [Google shares gRPC as alternative to REST for microservices](https://opensource.com/bus/15/3/google-grpc-open-source-remote-procedure-calls)
* [gRPC Motivation and Design Principles](http://www.grpc.io/blog/principles)
