# GMIT Distributed Systems

## Lab: Introduction to gRPC

### Overview
In this lab we'll look at an example of a simple gRPC service. The `Greeter` service is essentially a "Hello World" service, which takes a simple text message as input and returns a simple text response, using gRPC as a middleware layer to communicate transparently over the network.

## Lab Procedure
### Project Import
- Using the "Import as Maven project" functionality of your chosen IDE, import the `grpcIntro` project. Don't import the top-level of the repository (/distributed-systems-labs), this is just a folder containing individual projects.
- If using Eclipse, it may compain about there not being a JDK on the build path. Fix this by adding the JDK to the build path.
- The project contains the following files:
    - `helloworld.proto` in `src/main/proto`: This file, in Protocol Buffers format, defines two data structures, `HelloRequest` and `HelloReply`, each containing a single string field. It also defines a gRPC service interface `Greeter`, which has one method called `SayHello`:
    ```
        service Greeter {
            rpc SayHello (HelloRequest) returns (HelloReply) {}
      }
    ```
  - `HelloWorldClient.java` is a class which will call out gRPC `Greeter` service.
  - `HelloWorldServer.java` is a class which will ultimately handle the call to our gRPC `Greeter` service and return a response.

### Generating the gRPC Helper Classes
In the Protocol Buffers lab we manually ran the `protoc` compiler to generate our data access classes. This manual step doesn't lend itself well to build automation, so we'll automate it using a maven plugin, the `protobuf-maven-plugin`. This plugin comes bundled with its own copy of the `protoc` compiler, which it runs automatically whenever we run a maven compile. By default it will compile any `.proto` files it finds in `src/main/proto`, but this can be overridden by configuration.

- Open the `pom.xml` and find the `protobuf-maven-plugin`. Run a maven compile, and verify that the plugin produces the expected generated code in `target/generated-sources/`.
- The `protobuf/java` directory contains the generated data access classes. `protobuf/grpc-java` contains the gRPC server and client stubs we'll need to use to build our simple Hello World application.

### Using the gRPC Helper Classes to Start a gRPC Server
`HelloWorldServer` contains the class `GreeterImpl`, which is an implementation of the `Greeter` service.
```
static class GreeterImpl extends GreeterGrpc.GreeterImplBase {

    @Override
    public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
        HelloReply reply = HelloReply.newBuilder().setMessage("Hello " + req.getName()).build();
        responseObserver.onNext(reply);
        responseObserver.onCompleted();
    }
}
```
- This class overrides the `sayHello` method, providing an implementation which actually does the work of:
    - building a reply `HelloReply reply`
    - sending the reply as a response using a `StreamObserver`, a special gRPC interface for the server to call with its response.
- `HelloWorldServer` also defines a `start` method, which creates a new gRPC server using the `GreeterImpl` service and a defined port, and starts the server:
```
server = ServerBuilder.forPort(port)
        .addService(new HelloWorldServer.GreeterImpl())
        .build()
        .start();
```
- Run the `main` method in `HelloServer` to start the gRPC server.

 _In Eclipse, if the server isn't compiling due to gRPC classes not being found on the build path, then add the `target/generated-sources directory` as a source folder._


### Using the gRPC Helper Classes to Call a gRPC Client
- `HelloWorldClient` creates a new gRPC client for the `Greeter` using a gRPC `Channel`. A `Channel` provides a connection to a gRPC server on a specified host and port and is used when creating a client stub.
- `HelloWorldClient` uses its gRPC `Channel` to create a new blocking stub. A blocking stub is a synchronous client, i.e. calls to it "block" the client until a response is returned.
```
this.channel = ManagedChannelBuilder.forAddress(host, port)
        // Channels are secure by default (via SSL/TLS). For the example we disable TLS to avoid
        // needing certificates.
        .usePlaintext()
        .build();
greeterClientStub = GreeterGrpc.newBlockingStub(channel);
```
- Methods can now be called on this client stub as if they were local methods. The gRPC middleware layer handles all the network communications required to pass the message over the network and return the response. The `greet(String name)` method creates a `HelloRequest` and passes it as a parameter to the client:
```
HelloRequest request = HelloRequest.newBuilder().setName(name).build();
HelloReply response;
try {
    response = greeterClientStub.sayHello(request);
```
This is the essence of RPC: methods (like `sayHello(request)` above) can be called as if they were running local code, whereas in reality the real `sayHello` method is executing in a different process which may be on a different machine, even in a different datacenter or a different part of the world.

### Task
- Based on the example of `SayHello` above, add a new method to this gRPC service and call it from the client. Call the new method `SayHelloAgain`, and have it take the same parameters and return the same response as the `SayHello` method.
- Start by adding `SayHelloAgain` to the service definition in `helloworld.proto`:
```
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
```
