**Dependencies used by [java\_bean](YamlSerialization#java_bean.md) and [java\_v2protoc\_schema](YamlSerialization#java_v2protoc_schema.md)**
  * protostuff-api - 48kb
  * protostuff-yaml - 8kb

This library allows your generated pojos, existing pojos and existing protoc-generated pojos to be serialized to yaml format.

Take a look at the example:

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

# java\_bean #

Generate code via [protostuff-compiler](http://protostuff.googlecode.com/files/protostuff-compiler-1.0.0-jarjar.jar).
See [compiler options](CompilerOptions#java_bean.md) for more details.
  * simple pojos with a self-contained [schema](Schema.md).

### Serialization ###
```
Person person = new Person(1);
person.setName("John Doe");
person.setMotto("Speed kills!");
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
    int totalBytes = YamlIOUtil.writeTo(out, person, Person.getSchema(), buffer);
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
    int totalBytes = YamlIOUtil.writeTo(buffer, person, Person.getSchema());
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
    byte[] data = YamlIOUtil.toByteArray(person, Person.getSchema(), buffer);
    // do something with data
}
finally
{
    buffer.clear();
}
```

The output would look like:
```
--- !Person
name: John Doe
motto: Speed kills!
gender: 1
```

# java\_v2protoc\_schema #

Generated code via **protostuff-compiler**.  See [compiler options](CompilerOptions#java_v2protoc_schema.md) for more details.
  * complements the generated code from the official C++ protoc compiler.
  * allows the existing protoc-generated code to be efficiently serialized to yaml.

The generated code from c++ protoc would look like:
```
public final class Foo 
{
    public static final class Person extends com.google.protobuf.GeneratedMessageLite 
    {
        // field getters
        // serialization logic
        
        public static final class Builder extends com.google.protobuf.GeneratedMessageLite.Builder<com.example.foo.Foo.Person, Builder> 
        {
            // field getters/setters
            // deserialization logic
        }
    }
}
```

The generated [schema](Schema.md) by protostuff-compiler would look like:
```
public final class SchemaFoo 
{
    public static final class Person 
    {
        public static final MessageSchema WRITE = new MessageSchema();
        public static final BuilderSchema MERGE = new BuilderSchema();
        
        // you can subclass this to further customize
        public static class MessageSchema implements Schema<com.example.foo.Foo.proto.Person>
        {
            // serialization logic
        }
        public static class BuilderSchema implements Schema<com.example.foo.Foo.proto.Person.Builder>
        {
            // deserialization logic
        }
    }
}
```

It contains the ser/deser logic and also allows the fields to be written using its name or number.

### Serialization ###
```
Person person = Person.newBuilder()
    .setName("John Doe")
    .setMotto("Speed kills!")
    .setGender(Gender.MALE)
    .build();
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
    int totalBytes = YamlIOUtil.writeTo(out, person, SchemaFoo.Person.WRITE, buffer);
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
    int totalBytes = YamlIOUtil.writeTo(buffer, person, SchemaFoo.Person.WRITE);
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
    byte[] data = YamlIOUtil.toByteArray(person, SchemaFoo.Person.WRITE, buffer);
    // do something with data
}
finally
{
    buffer.clear();
}
```

The output would look like:
```
--- !Person
name: John Doe
motto: Speed kills!
gender: 1
```