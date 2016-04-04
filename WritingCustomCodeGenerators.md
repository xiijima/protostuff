# Generating code via stringtemplate #

Here's a [stringtemplate cheat sheet](http://www.antlr.org/wiki/display/ST/StringTemplate+cheat+sheet) for those who are not too familiar with it.

You will be generating code via templating (yes, like jsps except it generates code than html pages).

The templates can inherit from the built-in templates for addons and code reuse.

You simply invoke the compiler and supply your .stg resource as the "output".

**java -jar protostuff-compiler-1.0.0-jarjar.jar protostuff.properties**


`protostuff.properties`
```
   modules = foo
   foo.source = path/to/your/foo.proto
   foo.output = path/to/your/foo.java.stg
   foo.outputDir = generated
   foo.options = some_key,key:value,another_key
```

To test it out, try supplying http://protostuff.googlecode.com/files/protodoc.html.stg as the output.

It generates html from your .proto files. (You can optionally enable the option "compile\_imports" to "recursive").


---


Below is a guide/convention to structure/organize your stg (`StringTemplateGroup`)

The goal is to make it modular (de-centralized) so that your template can easily be extended and maintained.

The ".java" in "foo.java.stg" indicates the output target of your custom code generator.

`foo.java.stg`

```

// means you extend the base template (inherit its templates for this group to call)
// this mechanism enables us to maximize code re-use
// extending the template "base" gives you formatting.
// "CC" - camel-case
// "PC" - pascal-case
// "UC" - underscore-case
// "UUC" - uppercased "UC"
group foo : base;

//#proto begin

proto_block(proto, module, options) ::= <<

// this is for generating code with OuterClassname style

public final class <proto.options.java_outer_classname> 
{

    // call the enum_block template
    <proto.enumGroups:enum_block(eg=it, module=module, options=options)>

    // call the message block template
    <proto.messages:message_block(message=it, module=module, options=options)>

}

>>

//#message begin

message_block(message, module, options) ::= <<

public static final class <message.name> 
{
    // message contents
    public static final <message.name> INSTANCE = new <message.name>();

    <message:message_static_method_get_instance(message=it, options=options)>
    
    <message:message_constructor(message=it, options=options)>

}

>>

message_static_method_get_instance(message, options) ::= <<
public static <message.name> getInstance() 
{
    return INSTANCE;
}
>>

message_constructor(message, options) ::= <<
public <message.name>()
{

}
>>

//#enum begin

enum_block(eg, module, options) ::= <<

// enum contents

public enum <eg.name> implements com.dyuproject.protostuff.EnumLite\<<eg.name>\> 
{
    <eg.values:{v|<v.name>(<v.number>)}; separator=",\n">;
    
    public final int number;

    <eg:enum_constructor(eg=it, options=options)>

    <eg:enum_method_get_number(eg=it, options=options)>
    
}

>>

enum_constructor(eg, options) ::= <<
private <eg.name> (int number)
{
    this.number = number;
}
>>

enum_method_get_number(eg, options) ::= <<
public int getNumber()
{
    return number;
}
>>

```

If the rule "proto\_block" is defined, the compiler will supply `[proto,module,options]` as the args.

If not, the "message\_block" is looked up with the args `[message,module,options]`.

For enums, the rule would be "enum\_block" with the args `[eg,module,options]` (eg is a shortcut for `EnumGroup`)

See the [javadoc api](http://protostuff.googlecode.com/svn/javadoc/1.0.0/index.html) for more details.