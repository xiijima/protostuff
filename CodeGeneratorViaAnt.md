# `Using the code generator via Ant` #

Currently there are 4 generators:
  * json
  * numeric\_json
  * gwt\_json
  * gwt\_numeric\_json

The steps involved in generating the code are:
  * compile the .proto into a .java file
  * compile the .java file into a .class file
  * jar the compiled classes
  * generate the code using the protostuff-codegen jar

## `Model.proto` ##
```
package foo;

option optimize_for = LITE_RUNTIME;
option java_package = "com.example.foo.model";
option java_outer_classname = "Model";

message Greet {
  optional int32 id = 1;
  optional string name = 2;
  optional string message = 3;
  enum Status {
    NEW = 0;
    ACKNOWLEDGED = 1;
  }
  optional Status status = 4;
}
```

## `modules.properties` ##
```
modules = foo

foo.fullClassname = com.example.foo.model.Model
foo.outputPackage = com.example.foo.json
foo.outputDir = generated
foo.generator = json // json/numeric_json/gwt_json/gwt_numeric_json
foo.encoding = UTF-8

```

## `build.xml` ##

```
<project name="foo" basedir=".">

<target name="protoc">
  <exec executable="protoc">
    <arg value="--java_out=src/main/java" />
    <arg value="src/main/resources/Model.proto" />
  </exec>
</target>

<target name="javac" depends="protoc">
  <javac srcdir="src/main/java" destdir="target/classes" source="1.5" target="1.5" classpath="lib/protobuf-java-2.3.0.jar"/>
</target>

<target name="jar" depends="javac">
  <jar destfile="dist/model-1.0.0.jar" basedir="target/classes"/>
</target>

<target name="codegen" depends="jar">
  <java classname="com.dyuproject.protostuff.codegen.GeneratorMain">
    <arg value="src/main/resources/modules.properties"/>
    <classpath>
      <pathelement location="dist/model-1.0.0.jar"/>
      <pathelement location="lib/protostuff-codegen.jar"/>
    </classpath>
  </java>
</target>

</project>
```

Execute:
```
$ ant codegen
```