# `Using the code generator via maven ` #

Currently there are 4 generators:
  * json
  * numeric\_json
  * gwt\_json
  * gwt\_numeric\_json

The steps involved in generating the code are:
  * compile the .proto into a .java file
  * compile the .java file into a .class file
  * use **protostuff-maven-plugin** to generate code.

Try out the [maven archetypes](MavenArchetypes.md) to visualize the project layout for rapid development.

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

2 Modules:

  * ## `model/pom.xml` ##
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <parent>
    <artifactId>foo</artifactId>
    <groupId>com.example.foo</groupId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example.foo</groupId>
  <artifactId>foo-model</artifactId>
  <name>foo :: model</name>
  <packaging>jar</packaging>
  
  <build>
    <defaultGoal>install</defaultGoal>
    <plugins>
      <plugin>
        <artifactId>maven-antrun-plugin</artifactId>
        <executions>
          <execution>
            <id>generate-sources</id>
            <phase>generate-sources</phase>
            <configuration>
              <tasks>
                <exec executable="protoc">
                  <arg value="--java_out=src/main/java" />
                  <arg value="src/main/resources/com/example/foo/model/Model.proto" />
                </exec>
              </tasks>
              <sourceRoot>target/generated-sources</sourceRoot>
            </configuration>
            <goals>
              <goal>run</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  
  <dependencies>
    <dependency>
      <groupId>com.google.protobuf</groupId>
      <artifactId>protobuf-java</artifactId>
    </dependency>
  </dependencies>

</project>
```

  * ## `json/pom.xml` ##

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <parent>
    <artifactId>foo</artifactId>
    <groupId>com.example.foo</groupId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath>
  </parent>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example.foo</groupId>
  <artifactId>foo-json</artifactId>
  <name>foo :: json</name>
  <packaging>jar</packaging>

  <build>
    <defaultGoal>install</defaultGoal>
    <plugins>
      <plugin>
        <groupId>com.dyuproject.protostuff</groupId>
        <artifactId>protostuff-maven-plugin</artifactId>
        <version>${protostuff.version}</version>
        <configuration>
          <modules>
            <module>
              <fullClassname>com.example.foo.model.Model</fullClassname>
              <outputPackage>com.example.foo.json</outputPackage>
              <outputDir>src/main/java</outputDir>
              <generator>json</generator>
              <encoding>UTF-8</encoding>
            </module>
          </modules>
        </configuration>
        <dependencies>
          <dependency>
            <groupId>com.example.foo</groupId>
            <artifactId>foo-model</artifactId>
            <version>${project.version}</version>
          </dependency>
        </dependencies>
        <executions>
          <execution>
            <id>generate-sources</id>
            <phase>generate-sources</phase>
            <goals>
              <goal>codegen</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  
  <dependencies>
    <dependency>
      <groupId>com.example.foo</groupId>
      <artifactId>foo-model</artifactId>
      <version>${project.version}</version>
    </dependency>
    <dependency>
      <groupId>com.dyuproject.protostuff</groupId>
      <artifactId>protostuff-json</artifactId>
    </dependency>
  </dependencies>

</project>

```

Eexecute:
```
$ mvn install
```