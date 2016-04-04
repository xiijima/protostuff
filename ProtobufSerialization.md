The dependencies:
  * protostuff-api - 48kb
  * protostuff-core - 45kb

# java\_bean #

Generate code via [protostuff-compiler](http://protostuff.googlecode.com/files/protostuff-compiler-1.0.7-jarjar.jar).

`foo.proto`
```
package foo;

option optimize_for = LITE_RUNTIME;
option java_package = "com.example.foo";

message Person {
  required int32 id = 1;
  optional string name = 2;
  optional string motto = 3 [default="When the cat is away, the mouse is alone!"];
  enum Gender {
    MALE = 1;
    FEMALE = 2;
  }
  optional Gender gender = 4;
}
```

## Serialization ##
```
Person person = new Person(1);
person.setName("John Doe");
person.setGender(Gender.MALE);
```

All write operations require a `LinkedBuffer` as an arg.

It is better to re-use the buffer (application/threadlocal buffer) to avoid buffer allocation everytime you serialize.

Allocating a new buffer:
```
LinkedBuffer buffer = LinkedBuffer.allocate(512);
```

**writing to `java.io.OutputStream`**
```
OutputStream out;
LinkedBuffer buffer = getApplicationBuffer();
try
{
    int totalBytes = ProtobufIOUtil.writeTo(out, person, Person.getSchema(), buffer);
}
finally
{
    buffer.clear();
}
```

**writing to a `LinkedBuffer`**
```
LinkedBuffer buffer = getApplicationBuffer();
try
{
    int totalBytes = ProtobufIOUtil.writeTo(buffer, person, Person.getSchema());
    // write the buffer to somewhere else
    
}
finally
{
    // after writing, clear the buffer
    buffer.clear();
}
```

**writing to a byte array**
```
LinkedBuffer buffer = getApplicationBuffer();
try
{
    byte[] data = ProtobufIOUtil.toByteArray(person, Person.getSchema(), buffer);
    // do something with data
}
finally
{
    buffer.clear();
}
```

## Deserialization ##

**merging from `java.io.InputStream`**
```
Person person = new Person();
InputStream in;
ProtobufIOUtil.mergeFrom(in, person, Person.getSchema());

// or re-use the LinkedBuffer's internal byte array as a read buffer
LinkedBuffer buffer = getApplicationBuffer();
ProtobufIOUtil.mergeFrom(in, person, Person.getSchema(), buffer);
```

**merging from a byte array**
```
Person person = new Person();
byte[] data;
ProtobufIOUtil.mergeFrom(data, person, Person.getSchema()); 
```

If a nested message's required field is missing on deser,  `UninitializedMessageException is thrown`.

```
catch(UninitializedMessageException e)
{
    // your error handling. 
}

```

If a required field from the root message is missing, nothing is thrown.
This is intentional (saves extra object allocation and stacktrace).

You can simply do:
```
if(!schema.isInitialized(message))
{
    // your error handling. 
}

```