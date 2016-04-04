**Dependencies used by [java\_bean](JsonSerialization#java_bean.md) and [java\_v2protoc\_schema](JsonSerialization#java_v2protoc_schema.md)**
  * protostuff-api - 48kb
  * protostuff-json - 27kb
  * jackson-core-asl-1.7.3.jar - 203kb

This library allows your generated pojos, existing pojos and existing protoc-generated pojos to be serialized and deserialized to json format.

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

**writing to `java.io.OutputStream`**
```
boolean numeric;
OutputStream out;
JsonIOUtil.writeTo(out, person, Person.getSchema(), numeric);
```

**writing to `java.io.Writer`**
```
boolean numeric;
Writer writer;
JsonIOUtil.writeTo(writer, person, Person.getSchema(), numeric);
```

If the boolean arg "numeric" is true, the output would look like:
```
    {
    "1":1,
    "2":"John Doe",
    "3":"Speed kills!",
    "4":1
    }
```

### Deserialization ###

**merging from `java.io.InputStream`**
```
Person person = new Person();
boolean numeric;
InputStream in;
JsonIOUtil.mergeFrom(in, person, Person.getSchema(), numeric);
```

**merging from `java.io.Reader`**
```
Person person = new Person();
boolean numeric;
Reader reader;
JsonIOUtil.mergeFrom(reader, person, Person.getSchema(), numeric);
```

# java\_v2protoc\_schema #

Generated code via **protostuff-compiler**.  See [compiler options](CompilerOptions#java_v2protoc_schema.md) for more details.
  * complements the generated code from the official C++ protoc compiler.
  * allows the existing protoc-generated code to be efficiently serialized to json.

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

**writing to `java.io.OutputStream`**
```
boolean numeric;
OutputStream out;
JsonIOUtil.writeTo(out, person, SchemaFoo.Person.WRITE, numeric);
```

**writing to `java.io.Writer`**
```
boolean numeric;
Writer writer;
JsonIOUtil.writeTo(writer, person, SchemaFoo.Person.WRITE, numeric);
```

If the boolean arg "numeric" is true, the output would look like:
```
    {
    "1":1,
    "2":"John Doe",
    "3":"Speed kills!",
    "4":1
    }
```

### Deserialization ###

**merging from `java.io.InputStream`**
```
Person.Builder person = Person.newBuilder();
boolean numeric;
InputStream in;
JsonIOUtil.mergeFrom(in, person, SchemaFoo.Person.MERGE, numeric);
```

**merging from `java.io.Reader`**
```
Person.Builder person = Person.newBuilder();
boolean numeric;
Reader reader;
JsonIOUtil.mergeFrom(reader, person, SchemaFoo.Person.MERGE, numeric);
```