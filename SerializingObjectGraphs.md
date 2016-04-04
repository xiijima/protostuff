**Update: please use protostuff-1.0.3 or later**
  * critical bugs fixed (related to maps and dynamic types).

Features:
  * high-performance graph serialization
  * handles cyclic dependencies and polymorphic pojos (interface and abstract classes)

Limitations:
  * When a root message is serialized/deserialized, its nested messages should not contain references to it.

Example:
```
        message ClubFounder {
            optional string name = 1;
            optional Club club = 2;
        }
        
        message Club {
            optional string name = 1;
            repeated Student student = 2;
            repeated Club partner_club = 3;
        }

        message Student {
            optional string name = 1;
            repeated Club club = 2;
        }
```

You cannot directly serialize `Club` since it can potentially have a reference to itself.

The solution is to wrap the target message.
```
   message ClubWrapper {
       optional Club club = 1;
   }
```

On the other hand, you can serialize `ClubFounder`.
```
Schema<ClubFounder> schema = RuntimeSchema.getSchema(ClubFounder.class);
ClubFounder founder = new ClubFounder();

// fill with cyclic club and student objects

byte[] data = GraphIOUtil.toByteArray(founder, schema);

ClubFounder cf = new ClubFounder();
GraphIOUtil.mergeFrom(data, cf, schema);

// check if cf retains the graph
```


For a more complex example, take a look at [PolymorphicRuntimeGraphTest.java](http://code.google.com/p/protostuff/source/browse/trunk/protostuff-runtime/src/test/java/com/dyuproject/protostuff/runtime/PolymorphicRuntimeGraphTest.java)

Note that if you have collection fields that can potentially be cyclic,
you need to enable the system property below:
```
-Dprotostuff.runtime.collection_schema_on_repeated_fields=true
```

E.g
```
class Foo
{
    List<Bar> barList;
    List<Baz> anotherBarList; // if this points to barList, it will be cyclic.

}

```

Doing so treats the collection field as a standalone message (which is handled on cyclic graph serialization).