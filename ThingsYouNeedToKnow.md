# Things you need to know #

## Serialization Formats ##

When choosing a format, you need to know the limitations and tradeoffs each format brings.

#### `protobuf` ####
messages cannot be streamed (comes with its internal format)

what the official protobuf java implementation does:
  * it computes the size of each message and stores it on a private int field named "memoizedSerializedSize".
  * this means when your message contains a lot of nested messages, it will traverse every message (iterates the list if repeated field) on the graph (to compute the total size of the root message) before it can perform any serialization.
  * the good thing about it is, it can perform validation on each message's fields (while computing the size) before the message is actually written (which is the proper way to do so).
  * Tradeoffs:
    * None.  Protobuf was designed to have validation from the get-go, therefore, you cannot be streaming messages anyway.

what protobuf/protostuff does:
  * same applies, cannot be streamed (comes with the format)
  * validation and message size computation needs to happen during serialization (3-in-1 setting)
    * But wait, isn't it a bad idea to perform serialization when the message itself has not yet been declared valid?
      * Yes.
    * Does it mean I could be partially writing to the socket (`OutputStream`) when the `UnintializedMessageException` (required field missing) is thrown?
      * No.  We are buffering the writes to a series of logical buffers (one physical byte array if message size <= byte array size)
      * This means that the message needs to be fully computed and validated (while serializing to a byte array) before it can be written to the socket (`OutputStream`).
    * Tradeoffs
      * Even if you don't require validation, you still need to fully buffer the messages to reduce the serialization step to 1 instead of 2 (which is normally: 1. traverse the graph for validation/computation and 2.  traverse the graph again for the actual serialization)

#### `protostuff` ####
messages can be streamed like most serialization formats (e.g json)

If you need validation, then you need to buffer your writes before writing to the socket (`OutputStream`). Same rules apply for json, xml and other streaming formats.
```
     LinkedBuffer buffer = ...;
     try
     {
       int size = ProtostuffIOUtil.writeTo(buffer, message, schema);
       // you can prefix the message with the size (delimited message)
       LinkedBuffer.writeTo(outputStream, buffer);
     }
     catch(UninitializedMessageException e)
     {
       // your message was not written to the stream
       // because it was missing required fields.
     }
     finally
     {
       buffer.clear();
     }
     
```
Note that validation is intended for incoming messages.

When populating the messages, you as the developer should already know which fields are required.

The above snippet is a safety-net in case you forget.

When you have tested your application on your staging server, on the production server you can directly stream the messages without
full-buffering like the snippet above.

validation is built-in on the schema api but that does not mean you have to use it (especially if you're using protostuff as a serialization library for object graphs).
  * If messages are generated from .proto, simply avoid using "required" on any of your fields.
  * If using the runtime schemas, everything is "optional" (or "repeated" for collections).

when serializing utf8 strings, protostuff tries to do 2 things at once by writing to the buffer and computing the size at the same time.
  * Why? Computation is required since a protobuf string requires that it's length be written first before the actual value.
  * How? The max size of a utf8 string can be computed and protostuff uses that to decide how to efficiently perform the serialization.
  * In streaming mode (directly writing to socket `OutputStream`), if the size of a single utf8 string is too large to fit in the initial buffer(`LinkedBuffer.allocate(size)`), protostuff will attach session buffers to it.
    * What are session buffers?
      * These are small incremental buffers that are created, cached and re-used while writing to the `OutputStream`.
      * These buffers are flushed to the stream immediately after a complete utf8 string write (2-in-1: buffer write + utf8 size computation).
  * See the source code for protostuff-api (`StreamedStringSerializer`) for more details.

If you are fully buffering the writes and you have large messages, please do allocate a `LinkedBuffer` relative to the message size.

It is better to allocate large buffers and re-use them (application managed or thread-local) than to create one everytime.

Tradeoffs:
  * protostuff does a lot of atomic operations for speed in exchange for incremental internal buffering when needed.
    * In buffering mode, internal buffering is not needed if the entire message fits in the buffer.
    * In streaming mode, incremental session buffers are not needed if the there is no single utf8 string larger than the buffer.
  * these tradeoffs arguably makes protostuff the [fastest](http://code.google.com/p/thrift-protobuf-compare/wiki/BenchmarkingV2) in terms of serialization speed.


#### `graph` ####

Use it if you're serializing object graphs with cyclic references.

Same tradeoffs with protostuff plus a small overhead of using an `IdentityHashMap` to identify cyclic references.

#### `json` ####

Use it if you're talking to a browser.

An internal set of buffers are kept and recycled by jackson's  `BufferRecycler` when writing/reading to/from streams.

`JsonIOUtil` has methods where you can include the `LinkedBuffer` as an extra arg to effectively re-use your existing buffers when writing/reading to/from streams.

#### `xml` ####

Not as fast as the other formats but could be useful for legacy comms.

#### `yaml` ####

The most human-readable format supported by protostuff.

If you want to visualize the messages coming from your services/server, you'll want to use this.

The default indention is 2 spaces.

For better readability, you can increase this indention with the system property `-Dyamloutput.extra_indent=2` to make the indention 4 spaces.

## Serializing object graphs ##

Not portable to the other formats.

The graph/reference logic is embedded in the serialization format (protostuff) to achieve high performance graph ser/deser.

When you serialize the root object, its nested messages should not have a reference to it.  See [SerializingObjectGraphs](SerializingObjectGraphs.md) for more details.

## Schema Evolution ##

#### Generated pojos ####

To remove a field, simply remove the field from the .proto (or you can comment that out).

Make sure its field number is not re-used. (The same rules apply when adding new fields).

If you're mixing this with protostuff-runtime, its best that you do not use the field number 127.

#### Runtime pojos ####

To remove a field, annotate with `@Deprecated`.

When adding new fields, append the field on your pojo's field declaration. (The order is top to bottom).

The limit on the number of fields is 126.

If you are using the `@Tag` annotation, then there is no limit.
Only requirement is that you don't use 127.

If not and you have base classes and your messages inherit from those classes, make sure that the base classes are not subject to any change.

When a pojo inherits from a parent, the field number ordering will be based from the parent primarily then onto the child.

If you add a field to the parent, the field number ordering of the subclass will be messed up.

The whole concept is to preserve the field numbers.

It is best you use the `@Tag` annotation to fully support schema evolution on object hierarchies.

## Polymorphic serialization ##

This is only available for protostuff-runtime (obviously).

What does this really mean?
  * A pojo can have a field that is not a concrete type.
  * E.g
```
   public interface Foo
   {
       // ...
   }
   public abstract class Bar
   {
       // ...
   }
   public final class Person
   {
       Foo foo;
       Bar bar;
       Object baz; // can be foo or bar
   } 
```

## Runtime Options ##

Runtime options for protostuff-runtime:
  * `-Dprotostuff.runtime.enums_by_name=true`
    * Your enums are serialized using the string from Enum.name();
    * By default, this is disabled for protobuf compatibility (enums serialized using their number)
  * `-Dprotostuff.runtime.collection_schema_on_repeated_fields=true`
    * The collection (repeated field) will be serialized like a regular message (even if the collection is empty).
    * Disabled by default for protobuf compatibility (the collection is not serialized, only its values).
    * Here's an example (read the comments):
```
public final class Foo
{
   List<String> stringList;

   // equals and hashCode methods

}
```
    * Ser/deser
```
   LinkedBuffer buffer = ...;
   Foo foo = new Foo();
   // empty list
   foo.stringList = new ArrayList<String>();

   final byte[] data;
   try
   {
       data = ProtostuffIOUtil.toByteArray(foo, schema, buffer);
   }
   finally
   {
       buffer.clear();
   }

   Foo f = new Foo();
   
   ProtostuffIOUtil.mergeFrom(data, f, schema);
   
   assertEquals(f, foo); // this will fail if the option is not enabled because f.stringList will be null.
   // It would have been an empty list if the option was enabled (which makes it equal)
   
```

  * `-Dprotostuff.runtime.auto_load_polymorphic_classes=false`
    * Polymorphic serialization includes the concrete type of the object being serialized.  Upon deserialization, that className is read and is used to fetch the derived schema.
```
    boolean autoLoad = RuntimeSchema.AUTO_LOAD_POLYMORHIC_CLASSES;
    String className = ...; // read from the input.
    Schema<Object> derivedSchema = RuntimeSchema.getSchema(className, autoLoad);
    // If the class has not been loaded, and autoLoad is true, it will be loaded from the context classloader.
    // If autoLoad is false, a ProtostuffException is thrown (illegal operation, unknown message)
```
    * Enabled by default.  For security purposes, you can pre-load all your known pojos and disable this. Here's how:
```
// the code below preloads the schema of your pojos.
RuntimeSchema.getSchema(SomePojo.class);
RuntimeSchema.getSchema(AnotherPojo.class);
// and so on ...
```

  * `-Dprotostuff.runtime.morph_non_final_pojos=true`
    * Disabled by default (Some devs have a habit of not using the final keyword).
    * Basically, when your pojo's class is not declared final ... it can be subclassed (polymorphic)
    * Note that if you enable this option, every non-final pojo will have the overhead of including the type metadata on serialization
    * So, if you know that a particular class will not be subclassed, mark it final.
    * See [this issue](http://code.google.com/p/protostuff/issues/detail?id=43&can=1) for more details.

  * `-Dprotostuff.runtime.morph_collection_interfaces=true` (since 1.0.7)
    * Disabled by default.  Type metadata will not be included and instead, the collection will be mapped to a default impl.
      * `Collection = ArrayList`
      * `List = ArrayList`
      * `Set = HashSet`
      * `SortedSet = TreeSet`
      * `NavigableSet = TreeSet`
      * `Queue = LinkedList`
      * `BlockingQueue = LinkedBlockingQueue`
      * `Deque = LinkedList`
      * `BlockingDequeue = LinkedBlockingDeque`
    * Enabling this is useful if you want to retain the actual impl used (type metadata will be included).
    * To enable/override for a particular field, annotate the field with `com.dyuproject.protostuff.Morph` (since 1.0.7)
    * Since it is disabled by default, "`List<String> names;`" would be serialized to json like:
```
     {
       "names": ["foo","bar"]
     }
```
    * If enabled:
```
     {
       "names": {"y":"ArrayList", "v":[{"i":"foo"},{"i":"bar"}]}
     }
```
    * If you're using protostuff for webservices, then you'll probably want to leave it disabled and let protostuff map it to an `ArrayList`.

  * `-Dprotostuff.runtime.morph_map_interfaces=true` (since 1.0.7)
    * Disabled by default.  Type metadata will not be included and instead, the map will be mapped to a default impl.
      * `Map = HashMap`
      * `SortedMap = TreeMap`
      * `NavigableMap = TreeMap`
      * `ConcurrentMap = ConcurrentHashMap`
      * `ConcurrentNavigableMap = ConcurrentSkipListMap`
    * Enabling this is useful if you want to retain the actual impl used (type metadata will be included).
    * To enable/override for a particular field, annotate the field with `com.dyuproject.protostuff.Morph` (since 1.0.7)

  * `-Dprotostuff.runtime.id_strategy_factory=com.dyuproject.protostuff.runtime.IncrementalIdStrategy$Factory`
    * By default (if property is not present), the `DefaultIdStrategy` is used, which means a polymorphic pojo is identified by serializing its type as a string (FQCN).
    * If you set the above property, int ids are generated on the fly (thread-safe/atomic) and are mapped to your polymorphic pojos.  The end result is faster ser/deser and the serialized size is smaller (around 1/3-1/4 the size of the default strategy)
    * You can also reserve the first few ids (via `IncrementalIdStrategy.Registry`) for your core pojos, as well as set the max size for the `ArrayList` which holds the ids.

## Collection fields ##

Null values are not serialized. (A deserialized collection coming from a collection with null values will fail on `Collection.equals()`)

`Collections` with simple values(scalar,enum,pojo,delegate) are serialized without type metadata (normal operation).

Complex values (E.g `Queue<List<String>>`, `List<int[]>`, `Set<Object>`, `Deque<?>`) will be serialized **with** type metadata.

## Map fields ##

Allows null keys and values. (You can rely on the equality from  `Map.equals()`)

`Maps` with simple keys and values(scalar,enum,pojo,delegate) are serialized without type metadata (normal operation).

Complex values (E.g `Map<Object,List<String>>`, `HashMap<Set<byte[]>,float[]>`, `TreeMap<long[][],Object>`, `SortedMap<?,?>`) will be serialized **with** type metadata.