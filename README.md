# Spring Boot Maven plugin demonstration of problematic argument quoting

I have run into [an issue](https://github.com/spring-projects/spring-boot/issues/46555) with Spring AOT. My application uses variables to include
further profiles.

[`application.yaml`](./src/main/resources/application.yaml):

```yaml
spring.application.name: demo
spring.profiles.include: ${EXTRA:NOT-SET}
```

The intention is that the application starts fine when `EXTRA` is not set.
When `EXTRA` is set to an existing value (here, `hello`), then the profile
is loaded (as an example, it turns of the banner of the demo application).

When I run `mvn package` this includes `process-aot` in my example.
I am setting the `EXTRA` system property in the Maven [`pom.xml`](./pom.xml) like:

```xml
<plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
  <configuration>
    <systemPropertyVariables>
      <EXTRA>hello</EXTRA>
    </systemPropertyVariables>
  </configuration>
  ...
```

Running `mvn package` works fine on Windows, but will not work on Linux.

The issue is caused by `org.springframework.boot.maven.CommandLineBuilder`, which [adds quotes](https://github.com/spring-projects/spring-boot/blob/f278b1944c32b4a71641c0b5936067d4f3c8994d/build-plugin/spring-boot-maven-plugin/src/main/java/org/springframework/boot/maven/CommandLineBuilder.java#L103) around the
system property:

```java
return String.format("-D%s=\"%s\"", key, value);
```

This can be verified by running on Linux, for example

```sh
podman run -it --rm -v /c/Users/.../demo:/workspace maven:3.9.8-eclipse-temurin-21 sh -c "cd /workspace && mvn package"
```

