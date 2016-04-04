# `Using the proto compiler via Ant` #

First, download the compiler [protostuff-compiler-1.0.0-jarjar.jar](http://protostuff.googlecode.com/files/protostuff-compiler-1.0.0-jarjar.jar).

Currently there are 3 outputs:
  * java\_bean
  * gwt\_overlay
  * java\_v2protoc\_schema

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

## `protostuff.properties` ##
```
modules = foo

foo.source = foo.proto
#java_bean, gwt_overlay, java_v2protoc_schema
foo.output = java_bean
foo.outputDir = .
foo.encoding = UTF-8
foo.options = key:value,generate_field_map,separate_schema
```

## `build.xml` ##

```
<project name="foo" basedir=".">

<target name="compileproto">
  <java classname="com.dyuproject.protostuff.compiler.CompilerMain" fork="true" dir="src">
    <arg value="protostuff.properties"/>
    <classpath>
      <pathelement location="protostuff-compiler-1.0.0-jarjar.jar"/>
    </classpath>
  </java>
</target>

</project>
```

## `Execute:` ##
```
$ ant compileproto
```