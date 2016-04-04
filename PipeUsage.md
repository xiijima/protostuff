In protostuff, pipes are a way to transfer/transcode data from an input to another output. See http://en.wikipedia.org/wiki/Transcoding

Suppose you have an incoming protobuf input stream.
Normally, you have to parse the stream and build the message.
When you send the data to the client, you serialize the newly built message to json.

The main point of pipes is to skip the message building ... thus effectively converting one encoding to another.

Here's the normal approach:
```
InputStream protobufInputStream;

// deserialize
Foo foo = new Foo();
ProtobufIOUtil.mergeFrom(protobufInputStream, foo, Foo.getSchema(), inBuf);

// write to json
LinkedBuffer buffer = ...;
JsonIOUtil.writeTo(outputStream, foo, Foo.getSchema(), false, buffer);
```


---


Here's the code using pipes:

  * Note that for generated message, you need to enable the "generate\_pipe\_schema" option (see [compiler options](CompilerOptions#java_bean.md)). A static method "getPipeSchema" will be generated.

```
InputStream protobufInputStream;

// transcode the protobuf encoding to json encoding
LinkedBuffer buffer = ...;
Pipe pipe = ProtobufIOUtil.newPipe(protobufInputStream);
JsonIOUtil.writeTo(outputStream, pipe, Foo.getPipeSchema(), false, buffer);
```


For runtime objects, pipes are also supported.
  * The schema of a runtime object is an instance of `MappedSchema<T>`, which has a getPipeSchema() method.

```
// protostuff byte array from cache or datastore
byte[] protostuffData = ...;
MappedSchema<Foo> schema = (MappedSchema<Foo>)RuntimeSchema.getSchema(Foo.class);

// protostuff to json-numeric (which I use all the time for gwt overlays)
LinkedBuffer buffer = ...;
Pipe pipe = ProtostuffIOUtil.newPipe(protostuffData, 0, protostuffData.length);
JsonIOUtil.writeTo(outputStream, pipe, schema.getPipeSchema(), true, buffer);
```