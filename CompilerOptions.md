_Note that value (key:value) from some options are optional. The option "generate\_field\_map:false" does not disable the option ... remove the entry instead to disable._

_In maven, you would declare the key:value as:
```
<property>
  <name>Foo.implements_declaration</name> <!-- key -->
  <value>implements com.example.Bar</value> <!-- value -->
</property>
```_

# all #

**compile\_imports = true | false| recursive**
  * after the target proto is compiled, compile the imported protos as well.
  * if recursive, the imported protos of it's imported protos will be compiled as well (so on and so forth).

**${oldPackage} = ${newPackage}**
  * the matching package will be used for the compiled output.
  * useful to group certain protos together for certain scenarios (E.g  moving packages to ${gwtpackage}.client).

**header\_source\_path**
  * The source path (org/example/foo.proto) will be printed (as a comment) in the header.
  * By default, it only prints the name (foo.proto)

**underscore\_on\_vars**
  * If you have fields named after java keywords ("package", "default", etc), enable this option to avoid the clash. (appends an underscore)
  * best to avoid using keywords.  Who knows, you might be writing (in the future via reflection) to json output with its field names.

# java\_bean #

**generate\_field\_map**
  * generates a mapping between the field number and field name.
  * If on, you can serialize json encoded messages writing either its field name or field number.
```
      boolean numeric = false;
      JsonIOUtil.writeTo(out, message, numeric);
```

**generate\_pipe\_schema**
  * generates a pipe schema for transcoding an input to a different output.

**alphanumeric**
  * requires the "generate\_field\_map" option to be on.
  * the field names will be its field number prefixed by "f".  (e.g "f1")

**separate\_schema**
  * statically declared schema (lite at runtime).
  * especially great for environments like android or j2me

**builder\_pattern**
  * Well not really the builder pattern, I should have name this to chain\_methods :-)
  * the setters will return the instance.

**generate\_helper\_methods**
  * additional helper methods for repeated fields

**primitive\_numbers\_if\_optional
  * uses primitive numbers (no auto-boxing) if the field is optional.**

**${message.name}.implements\_declaration = implements com.example.Foo**
  * the matching message will implement the given interface.

**${message.name}.extends\_declaration = extends com.example.Bar**
  * the matching message will extend the given class.

# gwt\_overlay #

**numeric**
  * the encoded property name of a message is referenced by its field number.
  * {"firstName":"John","lastName":"Doe"} is json-serialized as {"1":"John","2":"Doe"}

**alphanumeric**
  * the encoded property name of a message is referenced by its field number prefixed by "f".
  * {"firstName":"John","lastName":"Doe"} is json-serialized as {"f1":"John","f2":"Doe"}

**plain\_overlay**
  * simply generate plain overlays that basically returns the property without checks.
  * By default, the gwt overlays generated will return the field's default values if null, as configured on the .proto file.

**dev\_mode**
  * additional code for overlays with enums to work on hosted mode.

**use\_global\_json**
  * the "parse" and "stringify" method would delegate to $wnd.JSON.parse and $wnd.JSON.stringify respectively.

**generate\_helper\_methods**
  * additional helper methods for repeated fields

**${message.name}.implements\_declaration = implements com.example.Foo**
  * the matching message will implement the given interface.
  * Note that gwt overlays can implement interfaces if gwt-2.0 +

# java\_v2protoc\_schema #

**generate\_field\_map**
  * generates a mapping between the field number and field name.
  * If on, you can serialize json encoded messages writing either its field name or field number.
```
      boolean numeric = false;
      JsonIOUtil.writeTo(out, message, numeric);
```

**enums\_by\_name**
  * serialize enums using their name instead of their number.

# java\_bean\_me #

a variant of java\_bean that produces j2me-compatible messages/schemas.