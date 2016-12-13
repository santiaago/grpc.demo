# grpc.demo## how

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
