# `Using the proto compiler via maven ` #

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

## `pom.xml plugin setup` ##
```
      <plugin>
        <groupId>com.dyuproject.protostuff</groupId>
        <artifactId>protostuff-maven-plugin</artifactId>
        <version>1.0.0</version>
        <configuration>
          <protoModules>
            <protoModule>
              <source>src/main/resources/foo.proto</source>
              <outputDir>src/main/java</outputDir>
              <output>java_bean</output>
              <encoding>UTF-8</encoding>
              <options>
                <property>
                  <name>generate_field_map</name>
                </property>
              </options>
            </protoModule>
          </protoModules>
        </configuration>
        <executions>
          <execution>
            <id>generate-sources</id>
            <phase>generate-sources</phase>
            <goals>
              <goal>compile</goal>
            </goals>
          </execution>
        </executions>
      </plugin>

```


You can explicitly invoke the compiler via:
```
$ mvn protostuff:compile
```


You can skip the compilation by specifiying the property:
```
-Dprotostuff.compiler.skip=true
```

