---
layout: post
title:  "Golang gRPC example"
date:   2022-06-12 19:43:30 +0100
---

# What is gRPC?

[gRPC](https://grpc.io/docs/what-is-grpc/introduction/) is a high performance, open source universal RPC framework. By default, it uses [Protocol Buffers](https://developers.google.com/protocol-buffers/docs/overview).

# Example

## Prerequisites

- [Golang](https://golang.org/doc/devel/release.html)
- [Protocol Buffer compiler](https://grpc.io/docs/protoc-installation/) - protoc
```bash
sudo apt install protobuf-compiler
```
- Update `PATH` variable to include the path to the `protoc` executable
- Golang plugins
```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

## Creating gRPC services

The process of creating gRPC services follows three steps:
- Define Messages/Services
- Generate client/server source
- Implement gRPC service methods

## Method types

There are four types of service methods that can be defined.

### Unary RPC

These are the simplest RPCs where client sends single request to server and gets single response back:

```
rpc Method(RequestMessage) returns (ResponseMessage)
```

### Server streaming

Server streaming RPCs where client sends single request, but gets a stream to read a sequence of messages. Example of this can be file download.

```
rpc Method(RequestMessage) returns (stream ResponseMessage)
```

### Client streaming

In this case, the client sends a sequence of messages and once it's finished, the server sends a single response. Example of this can be file upload.

```
rpc Method(stream RequestMessage) returns (ResponseMessage)
```

### Bidirectional streaming

In this case, both client and server are sending a sequence of messages over two streams that operate independently.

```
rpc Method(stream RequestMessage) returns (stream ResponseMessage)
```

## Protocol Buffers

```
syntax = "proto3"
package grpc-example;


service FileShare {
    rpc GetFileSize (GetFileSizeRequest) return (GetFileSizeResponse)
    rpc DownloadFile
    rpc UploadFile
    rpc GetAllFileSizes
}
```
