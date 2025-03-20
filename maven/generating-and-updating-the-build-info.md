# Generating and Updating the Build Info of a Package

You can use the Templating Maven Plugin to replace build properties values inside the template of a Java class. That Java class is automatically compiled and included in the application in the standard build.

To do this follow the steps below.

## 1. Create the Java template file

Create the file `src/main/java-templates/my/package/BuildInformation.java` and add the template content:

```java
package my.package;

public class BuildInformation {

  public static final String NAME = "${project.name}";
  public static final String VERSION = "${project.version}";
  public static final String BUILD_ID = "${timestamp}";

}
```

**Note**: Replace `my/package` with the specific package you want this class to be in.

**Note**: This generic source file should be stored in the SCM repository. Conversely, the generated source code (Java class with replaced values) will be compiled and included in the jar package, but won't be stored in the repository (since it will be placed inside the `target` directory).

## 2. Produce a Custom-formatted Build ID

By default, the format of the timestamp is fully ISO format (and is provided as the property `maven.build.timestamp`). This may not suit your needs.

To produce a custom-formatted build ID generate another property (`timestamp` in this case) that will produce the formatted value. Include the following tag inside the `<properties>` tag:

```xml
   <!-- creates a build property "timestamp" -->
   <timestamp>${maven.build.timestamp}</timestamp>
   <maven.build.timestamp.format>yyyyMMdd-HHmmss</maven.build.timestamp.format>
```

**Note**: In Maven, the build timestamp is always in UTC, not the local time zone. There's no way around this when using the standard functionality, and it actually makes sense when you have developers in different time zones; it avoids confusion. Nevetheless, it's technically possible to generate it in the local time zone (or any time zone), by using extra plugins; this is not covered here.

## 3. Enable the Templating Maven Plugin

Add the following configuration to the `pom.xml` file, inside the `<project>/<build>/<plugins>` tag:

```xml
      <!-- Enable the templating plugin to generate the BuildInformation class with build info -->
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>templating-maven-plugin</artifactId>
        <version>1.0.0</version>
        <executions>
          <execution>
            <id>filtering-java-templates</id>
            <goals>
              <goal>filter-sources</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
```

## 4. Run the Templating for the First Time

Use any maven goal that includes the "generate-source" phase. For example:

```bash
mvn package
```

This will generate the new source file `BuildInformation.java`, will compile it, and will include it in the generated jar/war.

## 5. Include the Newly Generated Class in Eclipse (optional)

To allow Eclipse to see the generated/updated class (so you can easily import it in any class) you'll need to add the folder as a new source folder in Eclipse.

First, refresh the project in Eclipse. The new folder `target/generated-sources/java-templates/my/package` will show up. Find it in Eclipse and:

```
-> Right-click on the new folder
-> Build Path -> Use as Source Folder
```

That's it.

From now on, every time you build using Maven, this file will be updated with the version coming from the `pom.xml` file and the current timestamp.

