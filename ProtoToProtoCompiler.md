# Extending proto messages #

**proto\_extender** compiler is designed to make one possible to extend his proto messages
with another ones.

It works like a preprocessor for your proto files and can help you to handle some inheritance
issues.

# Features #

  * extend one message with fields from another
  * annotations on the fields will be included too
  * `[defaults]` on the fields will no be inculded (this is not implemented yet)
  * annotations on the extender will to be included
  * no transitive extend resolution (this is not implemented yet)
  * no extend by multiple sources (msgs) (this is not implemented yet)

# Usage #

There are 2 kinds of ways to ask it to extend for you some message:

  * include ` option extends = ExtenderMessageName; ` at the first line after definition
of extendable message.
  * apply an attribute on extendable message ` @Extend(by = ExtenderMessageName) `

_Warning. Transitive resolution of extendables is not support yet. So if you want A to be extended
by B that should be extended by C, this is not possible right now._

# Sample #

```
    @Extend(by = ItemBase)
    message Item 
    {
        // or instead of @, this can be used: option extends = ItemBase;

        required int32 xxx = 10;
        optional string yyy = 11;
    }

    message ItemBase 
    {
        required int32 id = 1;
        @Test(v = 1, a = "bbb")
        optional string name = 2;
    }
```

the compilation result will be looked like:

```
    // @Extend(by = ItemBase)
    message Item 
    {
        // Extended by ItemBase at ....
        required int32 id = 1;
        @Test(v = 1, a = "bbb")
        optional string name = 2;

        required int32 xxx = 10;
        optional string yyy = 11;
    }

    message ItemBase 
    {
        required int32 id = 1;
        @Test(v = 1, a = "bbb")
        optional string name = 2;
    }   
```

as you can see, fields id's in extendable message should not intersect with
fields id's in extender message.

# Sample 2 #

```
    message Item 
    {
        required int32 id = 1;
        optional string name = 2;
    }

    @Extend(by = Item)
    message Weapon 
    {        
    }

    @Extend(by = Item)
    message Drug 
    {        
    }

    @Extend(by = Item)
    message Pet 
    {        
    }
```