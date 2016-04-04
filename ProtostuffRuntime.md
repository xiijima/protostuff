**Please use atleast protostuff-1.0.4 or later if you have raw byte array fields (`byte[],List<byte[]>,etc`). See [Issue 90](http://code.google.com/p/protostuff/issues/detail?id=90) for details.**

The **protostuff-runtime** module allows your existing pojos to be serialized to different formats.

For people who prefer not to have their messages code-generated from proto files, this fits your bill.

The preliminary modules needed:
  * protostuff-api
  * protostuff-collectionschema
  * protostuff-runtime

The advantages of using proto files is that you have explicit control of fields and their corresponding numbers (which is useful
for schema evolution, e.g forward-backward compatibility).

With this module, the field's number is ordered according to their declaration in the pojo (top to bottom).

**Note that the order is not guaranteed on some (non-sun) vms (especially dalvik)**.

Sun jdk6 or higher is recommended for guaranteed ordering ([link](http://stackoverflow.com/questions/5001172/java-reflection-getting-fields-and-methods-in-declaration-order)).

**As of 1.0.5, `@Tag` annotations can be used on fields to have explicit control of the field numbers**
```
// all or nothing.  
// Either you annotate all fields or you don't annotate at all (applies to the relevant class only).
// To exclude certain fields, use java's transient keyword
public final class Bar
{
  @Tag(8)
  int baz;

  // alias is available since 1.0.7
  // useful for json/xml/yaml where you can override the field names
  @Tag(value = 15, alias = "f")
  double foo;
}

// with this approach, versioning with inheritance is now fully supported.
// you simply reserve x-y (range) numbers for the fields of the parent class.
// internally it will be detected when you make mistakes tagging with the same number.
```

**Note that if you have non-static inner classes and want to use @Tag annotations, mark that class as static instead.  See [Issue 146](http://code.google.com/p/protostuff/issues/detail?id=146) for details.**


Without `@Tag` annotations, forward-backward compatibility is still supported via "append-only" schema evolution.
  * To add new fields, append the field in the declaration
  * To remove existing fields, annotate with @Deprecated
  * To exclude fields from being serialized, use the java keyword: **transient**

Here's an example:
```
    public final class Entity
    {
        int id;
        
        String name;
        
        @Deprecated
        String alias;
        
        long timestamp;
    }
```

Schema evolution scenario:
  * v1: 3 initial fields (id=1, name=2, alias=3)
  * v2: Added a new field (timestamp=4).
  * v3: Removed the "alias" field.

With v3, the field mapping would be (id=1, name=2, timestamp=4).  When we encounter the alias field, it is ignored by the deserializer.

The field mapping is still intact despite schema evolution ... w/c makes it forward-backward compatible to different versions.

## 3 possible types of Schema ##

Unlike a static hand-written/code-generated schema, there are 3 possible types of schema that can be used
at runtime.  Below are the types ordered according to their efficiency and performance at runtime.

  * **Static Schema**
    * used when the declared field is a concrete type.
    * compact since no extra metadata included on serialization
    * E.g
```
        public enum SortOrder
        {
            ASCENDING,
            DESCENDING;
        }

        public final class Bar
        {
            Entity entity; // the example above
            List<Long> scalarList; // any scalar type
            List<byte[]> bytesList; // byte arrays are treated as scalar fields (use >= 1.0.4)
            List<Entity> entityList;
            Map<String,byte[]> bytesMapWithScalarKeys;
            Map<String,Entity> entityMapWithScalarKeys;
            Map<SortOrder,Entity> entityMapWithEnumKeys;
            Map<Entity,Date> entityMapWithPojoKeys;
            Map<Entity,Entity> entityMap;
        }
```

  * **`DerivativeSchema`**
    * used when the declared field is an abstract class.
      * Use at least version 1.0.5.  (previous versions allowed interfaces but was changed because enums and most scalars do implement interfaces)
    * less compact since the type metadata is written (field number: 127) on serialization.
    * E.g
```
        public abstract class Instrument
        {
            // ...
        }

        public final class BassGuitar extends Instrument
        {
            // ...
        }

        public final class Piano extends Instrument
        {
            // ...
        }

        // DerivativeSchema will be used on the fields below
        public final class Baz
        {
            Instrument instrument;
            List<Instrument> instrumentList;
            Map<String,Instrument> instrumentMapWithScalarKeys;
            Map<SortOrder,Instrument> instrumentMapWithEnumKeys;
            Map<BassGuitar,Instrument> instrumentMapWithPojoKeys;
            Map<Instrument,Instrument> instrumentMap;
        }

```
    * IMPORTANT
      * If your object heirarchy involves a concrete class subclassing another concrete class (not using abstract classes), set:
> > > > `-Dprotostuff.runtime.morph_non_final_pojos=true`
      * With that property set, `DerivativeSchema` will be used on non-final pojos (concrete types) similar to abstract classes.
> > > > For example:
```
       class Base
       {
         int id = 1;
       }
       class Child extends Base
       {
         int status = 2;
       }
       class Pojo
       {
         Base b = new Child();
       }
       
       // If you serialize Pojo, Child's "status" field will not be 
       // serialized if the system property is not set.
       
       // With that in mind, all pojos that aren't marked final will 
       // have an overhead of extra type metadata on serialization.
       
       // To ensure that no extra type metadata be will written, mark 
       // your pojos final when you know there are no subclasses.
```

  * **`ObjectSchema`** (dynamic)
    * used when the type of the declared fields:
      * are `java.lang.Object`
      * are interfaces
      * are arrays
      * don't have generics
      * are too complex
    * all necessary metadata is included on serialization to be able to deserialize the message correctly.
    * E.g
```

        public final class Dynamic
        {
            Object entity;
    
            Object[] objectArray;
            int[] primitiveArray;
            Integer[] boxedArray;
            Entity[] entityArray;
            IEntity[] ientityArray;

            List noGenericsList;
            List<?> uselessGenericsList;
            List<Object> objectList;
            List<long[]> withArrayList;
    
            Map noGenericsMap;
            Map<?,?> uselessGenericsMap;
            Map<String,Object> withObjectMap;
            Map<?,SortOrder> dynamicKeyMap;
            Map<Entity,?> dynamicValueMap;
            Map<Integer[],int[]> withArrayMap;

            // and complex types
            List<List<String>> aListWithAList;
            Map<String,List<SortOrder>> complexMap;
            Map<Set<Entity>,Long> anotherComplexMap;
        }

```

### Updating fields ###

With the information above, be sure that you update your fields carefully.

For example ... don't add/remove generics when you already have existing data because the deserialization will fail.

For scalar fields:
  * `int` can be updated to `long` (and vice versa)
    * compatible with all suported formats
  * `String` can be updated to `byte[]`/`ByteString` (and vice versa)
    * not compatible with text formats (e.g json/xml/yaml)

```
class Example
{
   int i;
   long l;
   Integer i2;
   Long l2;
   String s;
   byte[] b;
   ByteString bs;
}

```

### Performance guidelines ###

As much as possible, use the concrete type when declaring a field.

For polymorhic datasets, prefer abstract classes vs interfaces.
  * Use `ExplicitIdStrategy` to write the type metadata as int (ser/deser will be faster and the serialized size will be smaller).
    * Register your concrete classes at startup via `ExplicitIdStrategy.Registry`.

  * For objects not known ahead of time, use `IncrementalIdStrategy`
    * You can activate it using the system property:

> > > `-Dprotostuff.runtime.id_strategy_factory=com.dyuproject.protostuff.runtime.IncrementalIdStrategy$Factory`

  * You can also use these strategies independently.  E.g:
```
    final IncrementalIdStrategy strategy = new IncrementalIdStrategy(....);  
    // use its registry if you want to pre-register classes.

    // Then when your app needs a schema, use it.
    RuntimeSchema.getSchema(clazz, strategy);
```

## Usage ##

_Note that on deser, if your object does not have a default constructor, you can always use **schema.newMessage()** to instantiate (internally similar to how the default java-serialization instantiates)_

```
   Foo foo = new Foo("foo", 1, 3.5);

   // this is lazily created and cached by RuntimeSchema
   // so its safe to call RuntimeSchema.getSchema(Foo.class) over and over
   // The getSchema method is also thread-safe
   Schema<Foo> schema = RuntimeSchema.getSchema(Foo.class);
   LinkedBuffer buffer = getApplicationBuffer();
   
   /* -------- protostuff -------- (requires protostuff-core module) */
   // ser
   try
   {
       byte[] protostuff = ProtostuffIOUtil.toByteArray(foo, schema, buffer);
   }
   finally
   {
       buffer.clear();
   }
   // deser
   Foo f = schema.newMessage();
   ProtostuffIOUtil.mergeFrom(protostuff, f, schema);
   
   
   /* -------- protobuf -------- (requires protostuff-core module) */
   // ser
   try
   {
       byte[] protobuf = ProtobufIOUtil.toByteArray(foo, schema, buffer);
   }
   finally
   {
       buffer.clear();
   }
   // deser
   Foo f = schema.newMessage();
   ProtobufIOUtil.mergeFrom(protobuf, f, schema);
   
   
   /* -------- json -------- (requires protostuff-json module)*/
   // ser
   boolean numeric = true;
   byte[] json = JsonIOUtil.toByteArray(foo, schema, numeric, buffer);

   // deser
   Foo f = schema.newMessage();
   JsonIOUtil.mergeFrom(json, f, schema, numeric);
   
    
   /* -------- xml -------- (requires protostuff-xml module)*/
   // ser
   byte[] xml = XmlIOUtil.toByteArray(foo, schema);

   // deser
   Foo f = schema.newMessage();
   XmlIOUtil.mergeFrom(xml, f, schema);
   
   
   /* -------- yaml -------- (requires protostuff-yaml module)*/
   // ser
   try
   {
       byte[] yaml = YamlIOUtil.toByteArray(foo, schema, buffer);
   }
   finally
   {
       buffer.clear();
   }
   
```

### Reading/Writing from/to streams ###
```
   Foo foo = new Foo("foo", 1, 3.5);
   
   Schema<Foo> schema = RuntimeSchema.getSchema(Foo.class);
   LinkedBuffer buffer = getApplicationBuffer();
   
   /* -------- protostuff -------- (requires protostuff-core module)*/
   // ser
   try
   {
       ProtostuffIOUtil.writeTo(outputStream, foo, buffer);
   }
   finally
   {
       buffer.clear();
   }
   // deser
   Foo f = schema.newMessage();
   ProtostuffIOUtil.mergeFrom(inputStream, f, schema, buffer);
   

   /* --------protobuf -------- (requires protostuff-core module)*/
   // ser
   try
   {
       ProtobufIOUtil.writeTo(outputStream, foo, buffer);
   }
   finally
   {
       buffer.clear();
   }
   // deser
   Foo f = schema.newMessage();
   ProtobufIOUtil.mergeFrom(inputStream, f, schema, buffer);
   
   
   /* -------- json -------- (requires protostuff-json module)*/
   // ser
   boolean numeric = false;
   JsonIOUtil.writeTo(outputStream, foo, schema, numeric, buffer);
   
   // deser
   Foo f = schema.newMessage();
   JsonIOUtil.mergeFrom(inputStream, f, schema, numeric, buffer);
   
   
   /* -------- xml -------- (requires protostuff-xml module)*/
   // ser
   XmlIOUtil.writeTo(outputStream, foo, schema);
   
   // deser
   Foo f = schema.newMessage();
   XmlIOUtil.mergeFrom(inputStream, f, schema);
   
   
   /* -------- yaml -------- (requires protostuff-yaml module)*/
   // ser
   try
   {
       YamlIOUtil.writeTo(outputStream, foo, buffer);
   }
   finally
   {
       buffer.clear();
   }

```