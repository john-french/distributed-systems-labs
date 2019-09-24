# GMIT Distributed Systems

## Lab: Binary Serialisation with Protocol Buffers

### Overview 
In this lab we'll look at how to serialise data in a binary form using [Protocol Buffers](https://developers.google.com/protocol-buffers/) (or protobuffers for short), the main mechanism used to store and transmit data internally at Google.
We'll use the example of a very simple addresss book application that can read and write people's contact details to and from a file. 

Instead of using text-based data representation formats like XML or JSON to serialise this structured data, we'll use [Protocol Buffers](https://developers.google.com/protocol-buffers/) (aka protobuffers) to serialise the data in a binary format.

## Lab Procedure
### Project Import
- Using the "Import as Maven project" functionality of your chosen IDE, import the `protobuffersLab` project. Don't import the top-level of the repository (/distributed-systems-labs), this is just a folder containing individual projects. 
- The project contains the following:
    - `AddPerson.java`: a simple Java program to read in new address book data from the user and serialise it to a binary file using protobuffers.
    - `ListPeople.java`: a simple Java program to read the address book data file and print out its contents.
    - `protoc-3.9.1-win64/`: a folder containing version 3.9.1 of the Protocol Buffer compiler `protoc`, built for 64-bit Windows.
    
### Defining the data structure
- Our application needs two data structures: a `Person` and an `AddressBook` containing multiple `Person`s. The `Person` will have a `name`, `id`, `email`, and possible several phone numbers. In order to serialise data like this using protobuffers, we first need to define the structure of this data in a `.proto` file.       
- Create a new file `addressbook.proto` in `src/main/resources to hold our protobuffer definitions, and add the following code:
```
syntax = "proto2";
package gmit.ds;

option java_package = "ie.gmit.ds";
option java_outer_classname = "AddressBookProtos";

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

- Let's go through this file to see what's going on:
    - The `.proto` file starts with a package declaration, which helps to prevent naming conflicts between different projects in potentially different languages, all using the Protocol Buffers namespace. 
    - Next are two Java-specific options: `java_package` and `java_outer_classname`. 
        - `java_package` specifies in what Java package name your generated classes should live. 
        - `java_outer_classname` defines the class name which should contain all of the classes in this file.
    - Next, you have your *message definitions*. A message is just an aggregate containing a set of typed fields, analogous to an object. 
        - You can also add further structure to your messages by using other message types as field types – in the above example the Person message contains `PhoneNumber` messages, while the `AddressBook` message contains `Person` messages.
        - You can even define message types nested inside other messages – as you can see, the PhoneNumber type is defined inside Person. You can also define `enum` types if you want one of your fields to have one of a predefined list of values – here you want to specify that a phone number can be one of MOBILE, HOME, or WORK.
    - The " = 1", " = 2" markers on each element identify the unique "tag" that field uses in the binary encoding.
    - Each field must be annotated with one of the following modifiers:
        - `required`: a value for the field must be provided, otherwise the message will be considered "uninitialized". Trying to build an uninitialized message will throw a RuntimeException. Parsing an uninitialized message will throw an IOException. Other than this, a required field behaves exactly like an optional field.
        - `optional`: the field may or may not be set. If an optional field value isn't set, a default value is used. 
        - `repeated`: the field may be repeated any number of times (including zero). The order of the repeated values will be preserved in the protocol buffer. Think of repeated fields as dynamically sized arrays.

### Generate data access classes using protobuffer compiler
- Now let's use the Protocol Buffer compiler `protoc` to generate the Java classes we'll need in our application. Version 3.9.1 of the compiler, pre-built for Windows 64-bit, is included with the lab code. Other versions are available at the protobuffers [project release page on github](https://github.com/protocolbuffers/protobuf/releases) (Mac version is [here](https://github.com/protocolbuffers/protobuf/releases/download/v3.9.2/protoc-3.9.2-osx-x86_64.zip), Linux version is [here](https://github.com/protocolbuffers/protobuf/releases/download/v3.9.2/protoc-3.9.2-linux-x86_64.zip)).
    - Open a terminal window (Git Bash is recommended). From the folder distributed-systems-labs/protoBuffers, run the following command.
    ```
    protoc-3.9.1-win64/bin/protoc.exe --java_out=./src/main/java ./src/main/resources/addressbook.proto
    ```
    - `--java_out` indicates that we want to generate Java code, and specifies where it should go. 
    - The final parameter is the path to the `.proto` definition file.

- This should create a new file called `AddressBookProtos.java` in the `ie.gmit.ds` package under `src/main/java`. Open this file and take a look at it's structure. Identify the `Person` and `AddressBook` classes, and take a look at the `Builder` class they each contain. We'll use these `Builder`s to create new instances of our message objects.
- The data access classes `Person` and `AddressBook` generated by `protoc` aren't just regular Java POJOs. They have protobuffer serialisation powers, and other methods, that they inherit from the protobuffer library. 
#### Question
What class provides this serialisation functionality? What interface defines what additional methods the data access classes should implement?
<details><summary>Hint</summary>

`Person` and `AddressBook` extend `com.google.protobuf.GeneratedMessageV3`. This provides serialisation functionality. The auto-generated `PersonOrBuilder` and `AddressBookOrBuilder` interfaces define the methods the `Person`/`AddressBook` and their associated builder should implement. 

</details>  

### Use the data access classes in our application
- In order to use these generated data access classes, we'll need the protobuffer java library they depend on. Add the Protocol Buffer runtime dependency to your `pom.xml`
```
    <dependency>
      <groupId>com.google.protobuf</groupId>
      <artifactId>protobuf-java</artifactId>
      <version>3.9.1</version>
    </dependency>
```
#### Tasks
##### Complete the following tasks, referring to the lecture notes and [Protocol Buffer Developer Guide](https://developers.google.com/protocol-buffers/docs/overview) where necessary.
- `AddPerson.java` contains the static method `PromptForAddress` which is supposed to create and return a `Person` object. Right now it  prompts the user for some person data, but doesn't compile because it doesn't return a `Person`. Complete this method by creating a `Person` object using the appropriate builders. A `Person` builder has already been created for you using `Person.newBuilder()`, and some of the fields have been set (`id`, `name`). Finish creating this person by
- adding a phone number (Hint: you'll need a `PhoneNumberBuilder`)
- calling `build()` and returning a `Person` object.
Add your code where it says `ADD CODE HERE` :smiley:.

<details><summary>Solution</summary>
    
```  
    Person.PhoneNumber.Builder phoneNumber =
            Person.PhoneNumber.newBuilder().setNumber(number);
    stdout.print("Is this a mobile, home, or work phone? ");
    String type = stdin.readLine();
    if (type.equals("mobile")) {
        phoneNumber.setType(Person.PhoneType.MOBILE);
    } else if (type.equals("home")) {
        phoneNumber.setType(Person.PhoneType.HOME);
    } else if (type.equals("work")) {
        phoneNumber.setType(Person.PhoneType.WORK);
    } else {
        stdout.println("Unknown phone type.  Using default.");
    }

    person.addPhones(phoneNumber);
    }

    return person.build();
 ```
 </details>
 
 - The `main` method  adds the people we created to the address book. You need to write this address book to an output file. Use the methods provided by the class itself. Add your code where it says `ADD CODE HERE`. When this works you should see a file called `addressBook.data` on the file system. This file contains a binary representation of your data in the protocol buffer format.

<details><summary>Solution</summary>

```  
    // Write the new address book back to disk.
    FileOutputStream output = new FileOutputStream(ADDRESS_BOOK_FILE);
    try {
        addressBook.build().writeTo(output);
    } finally {
        output.close();
    } 
```
</details>


- `ListPeople.java` should read the `addressBook.data` file and print out its contents. A static `Print` method is provided that does the formatted printing for you. You need to add the code to read the data file.

<details><summary>Solution</summary>
    
```
    AddressBook addressBook =
      AddressBook.parseFrom(new FileInputStream(ADDRESS_BOOK_FILE));

    Print(addressBook);
```
</details>

### Conclusion
Now you've successfully:
- defined a data structure in `.proto` format.
- generated data access classes using the protocol buffer compiler.
- used these classes to serialise and deserialise data using the Protocol Buffer binary format. 

   
 _Lab adapted from the [Google Protocol Buffers Java tutorial](https://developers.google.com/protocol-buffers/docs/javatutorial)._
