# GMIT Distributed Systems

## Lab: Asynchronous gRPC

### Overview
In this lab we'll build a gRPC Inventory service, which will use an asynchronous method call.

## Lab Procedure
### Project Creation
- Create a new maven project with artifactId `grpc-async-inventory`, and groupId `ie.gmit.ds`.
- We know we'll need the gRPC libraries, and we'll be using the `protobuf-maven-plugin` to automatically compile our `.proto` file, so add this to the pom:

<details>
    <summary><b>pom.xml</b></summary>

```
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <grpc.version>1.23.0</grpc.version><!-- CURRENT_GRPC_VERSION -->
        <protobuf.version>3.9.0</protobuf.version>
        <protoc.version>3.9.0</protoc.version>
        <!-- required for jdk9 -->
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>javax.annotation</groupId>
            <artifactId>javax.annotation-api</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-netty-shaded</artifactId>
            <version>1.23.0</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-protobuf</artifactId>
            <version>1.23.0</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-stub</artifactId>
            <version>1.23.0</version>
        </dependency>
    </dependencies>

    <build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.5.0.Final</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.5.1</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:${protoc.version}:exe:${os.detected.classifier}
                    </protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}
                    </pluginArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

        </plugins>
    </build>
 ```
 
 </details>

### Defining the Service Interface
- Our InventoryService will expose methods to add a new item and to list all items. Create a new file `inventory.proto` in `src/main/proto` (the maven plugin looks here by default), and add the following:

<details>
    <summary><b>inventory.proto</b></summary>

```
syntax = "proto3";
package ie.gmit.ds;
import "google/protobuf/wrappers.proto";
import "google/protobuf/empty.proto";

option java_multiple_files = true;
option java_package = "ie.gmit.ds";

service InventoryService {
   rpc addItem(Item) returns (google.protobuf.BoolValue);
   rpc getItems(google.protobuf.Empty) returns (Items);
}

message Items {
   string itemDesc = 1;
   repeated Item items = 2;
}

message Item {
    string id = 1;
    string name = 2;
    string description = 3;
}
```

</details>

- Running a maven compile should now create our gRPC client and server stubs in an `InventoryServiceGrpc` class in `target/generated-sources/`, along with the `Item` and `Items` data access classes.

### Implementing the Service
- Create a new package `ie.gmit.ds` in `src/main/java`.
- In that package, create a new class `InventoryServiceImpl` which extends `InventoryServiceGrpc.InventoryServiceImplBase`. We'll need to override and provide implementations for the `addItem` and `getItems` methods, as follows:

<details>
    <summary><b>Method Implementations</b></summary>

```
  private ArrayList<Item> itemsList;
    private static final Logger logger =
            Logger.getLogger(InventoryServiceImpl.class.getName());

    public InventoryServiceImpl() {
        itemsList = new ArrayList<>();
        createDummyItems();
    }

    @Override
    public void addItem(Item item,
                        StreamObserver<BoolValue> responseObserver) {
        try {
            itemsList.add(item);
            logger.info("Added new item: " + item);
            responseObserver.onNext(BoolValue.newBuilder().setValue(true).build());
        } catch (RuntimeException ex) {
            responseObserver.onNext(BoolValue.newBuilder().setValue(false).build());
        }
        responseObserver.onCompleted();
    }

    @Override
    public void getItems(Empty request,
                         StreamObserver<Items> responseObserver) {
        Items.Builder items = Items.newBuilder();
        for (Item item : itemsList) {
            items.addItems(item);
        }
        responseObserver.onNext(items.build());
        responseObserver.onCompleted();
    }
```
</details>

- Let's also populate the inventory with some dummy items to help us test:

<details>
    <summary><b>Code to populate inventory</b></summary>

```
    private void createDummyItems() {
        itemsList.add(Item.newBuilder()
                .setName("First Item")
                .setId("001")
                .setDescription("A cool item")
                .build());
        itemsList.add(Item.newBuilder()
                .setName("Second Item")
                .setId("002")
                .setDescription("An even cooler item")
                .build());
        itemsList.add(Item.newBuilder()
                .setName("Third Item")
                .setId("003")
                .setDescription("A crap item")
                .build());
    }
```
</details>

### Create Server and Register Service
- Create a new class `InventoryServer`in `ie.gmit.ds`, and declare the following variables:
```
    private Server grpcServer;
    private static final Logger logger = Logger.getLogger(InventoryServer.class.getName());
    private static final int PORT = 50551;
```
- We'll need to add methods to start and stop our server, and to keep it alive until something terminates it. The `start` method creates a new gRPC Server and registers our Inventory Service implementation with it:

<details>
    <summary><b>Server startup and shutdown methods</b></summary>

```
    private void start() throws IOException {
        grpcServer = ServerBuilder.forPort(PORT)
                .addService(new InventoryServiceImpl())
                .build()
                .start();
        logger.info("Server started, listening on " + PORT);

    }

    private void stop() {
        if (grpcServer != null) {
            grpcServer.shutdown();
        }
    }

    /**
     * Await termination on the main thread since the grpc library uses daemon threads.
     */
    private void blockUntilShutdown() throws InterruptedException {
        if (grpcServer != null) {
            grpcServer.awaitTermination();
        }
    }
```
</details>

- Give the server a main method so we can easily run it and start the gRPC server:

<details>
    <summary><b>Server main method</b></summary>

```
    public static void main(String[] args) throws IOException, InterruptedException {
        final InventoryServer inventoryServer = new InventoryServer();
        inventoryServer.start();
        inventoryServer.blockUntilShutdown();
    }
```
</details>

### Create Client
- Create a new class `InventoryClient` in `ie.gmit.ds`. We'll need to create a `Channel` to connect to the gRPC server, and stubs to call methods on. We'll use a blocking (synchronous) and asynchronous stub.

<details>
<summary><b>Client Initialisation</b></summary>

```
    private static final Logger logger =
            Logger.getLogger(InventoryClient.class.getName());
    private final ManagedChannel channel;
    private final InventoryServiceGrpc.InventoryServiceStub asyncInventoryService;
    private final InventoryServiceGrpc.InventoryServiceBlockingStub syncInventoryService;

    public InventoryClient(String host, int port) {
        channel = ManagedChannelBuilder
                .forAddress(host, port)
                .usePlaintext()
                .build();
        syncInventoryService = InventoryServiceGrpc.newBlockingStub(channel);
        asyncInventoryService = InventoryServiceGrpc.newStub(channel);
    }
    
    public void shutdown() throws InterruptedException {
        channel.shutdown().awaitTermination(5, TimeUnit.SECONDS);
    }
```

</details>

### Call Client Stubs
#### Synchronous
- We'll add a new inventory item synchronously by calling the blocking stub:

<details>
    <summary><b>Synchronous Call</b></summary>

```
    public void addNewInventoryItem(Item newItem) {
        logger.info("Adding new inventory item " + newItem);
        BoolValue result = BoolValue.newBuilder().setValue(false).build();
        try {
            result = syncInventoryService.addItem(newItem);
        } catch (StatusRuntimeException ex) {
            logger.log(Level.WARNING, "RPC failed: {0}", ex.getStatus());
            return;
        }
        if (result.getValue()) {
            logger.info("Successfully added item " + newItem);
        } else {
            logger.warning("Failed to add item");
        }
    }
```
</details>

- To list items, we'll use an asynchronous call. This time we'll need to pass in a `StreamObserver` to handle the aynchronous response from the server:

<details>
    <summary><b>Asynchronous Call</b></summary>

```
    private void getItems() {
        StreamObserver<Items> responseObserver = new StreamObserver<Items>() {
            @Override
            public void onNext(Items items) {
                logger.info("Received items: " + items);
            }

            @Override
            public void onError(Throwable throwable) {
                Status status = Status.fromThrowable(throwable);

                logger.log(Level.WARNING, "RPC Error: {0}", status);
            }

            @Override
            public void onCompleted() {
                logger.info("Finished receiving items");
                // End program
                System.exit(0);
            }
        };

        try {
            logger.info("Requesting all items ");
            asyncInventoryService.getItems(Empty.newBuilder().build(), responseObserver);
            logger.info("Returned from requesting all items ");
        } catch (
                StatusRuntimeException ex) {
            logger.log(Level.WARNING, "RPC failed: {0}", ex.getStatus());
            return;
        }
    }
```

</details>

- Now let's actually call these client methods by providing a main method which will:
    - Create an `InventoryClient`
    - Create a new Item
    - Use the client to add the new item to the inventory
    - Use the client to list all items (asynchronously)

<details>
    <summary><b>Client main method</b></summary>

```
    public static void main(String[] args) throws Exception {
        InventoryClient client = new InventoryClient("localhost", 50551);
        Item newItem = Item.newBuilder()
                .setId("1234")
                .setName("New Item")
                .setDescription("Best New Item")
                .build();
        try {
            client.addNewInventoryItem(newItem);
            client.getItems();
        } finally {
            // Don't stop process, keep alive to receive async response
            Thread.currentThread().join();
        }
    }
 ```

</details>

### Running the code
- Start the server
- Run the client. You should see the client return from the call to getItems, then subsequently you'll see the result of the callback from getItems.
