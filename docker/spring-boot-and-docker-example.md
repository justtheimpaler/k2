# Hello World Docker!

This guide runs a Spring Boot project with Docker. 

It includes a REST API, database connection (SQL Server), and serves static HTTP resources (HTML, images, CSS, etc.).

To run this sample app you'll need:

- Java.
- Maven.
- Docker installed on a server.
- A text editor. `Notepad` or `vi` will do, but you can also use your favorite IDE as well.

This guide sets up the Maven project with a very simple app. It first builds it and runs it locally. Then, it builds a docker image and runs it as a docker container.


## Part 1 &mdash; Setting Up the Project

In this part we create the Maven project.


### Set Up a Maven Project

If you are using a plain text editor (such as Notepad) you can create an empty folder and add the files as described in the 
steps below. Alternatively, you can use your favorite IDE to create a blank Maven project.

In your brand new project add a `pom.xml` file with the content:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.app</groupId>
  <artifactId>app</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>jar</packaging>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
  </properties>

  <dependencies>

    <dependency> <!-- The Spring Boot Dependency -->
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>2.3.4.RELEASE</version>
    </dependency>
    
    <!-- The libraries included below will change according to the persistence layer, such as 
         plain JDBC, JPA, JdbcTemplate, MyBatis, etc. Change accordingly -->

    <dependency> <!-- Logging dependency -->
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-simple</artifactId>
      <version>2.0.5</version>
    </dependency>

    <dependency> <!-- Logging dependency -->
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-api</artifactId>
      <version>2.17.2</version>
    </dependency>

    <dependency> <!-- Logging dependency -->
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
      <version>2.17.2</version>
    </dependency>

    <dependency> <!-- Logging dependency -->
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-log4j2</artifactId>
      <version>2.3.4.RELEASE</version>
    </dependency>

    <dependency> <!-- The JDBC driver -->
      <groupId>com.microsoft.sqlserver</groupId>
      <artifactId>mssql-jdbc</artifactId>
      <version>10.2.1.jre8</version>
    </dependency>

    <dependency> <!-- The DataSource Connection Pool Implementation -->
      <groupId>com.zaxxer</groupId>
      <artifactId>HikariCP</artifactId>
      <version>5.0.1</version>
    </dependency>

  </dependencies>

  <build>

    <finalName>app</finalName>

    <plugins>
    
      <!-- Repackage as a single big Spring-Boot jar -->

      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>2.3.4.RELEASE</version>
        <executions>
          <execution>
            <goals>
              <goal>repackage</goal>
            </goals>
          </execution>
        </executions>
      </plugin>

    </plugins>

  </build>

  <profiles>

    <profile>

      <id>docker</id>

      <build>

        <plugins>

          <!-- 1. Copy files to the docker context (for the image build) -->

          <plugin>
            <artifactId>maven-resources-plugin</artifactId>
            <version>3.1.0</version>
            <executions>
              <execution>
                <id>copy-resource-two</id>
                <phase>package</phase>
                <goals>
                  <goal>copy-resources</goal>
                </goals>
                <configuration>
                  <outputDirectory>${basedir}/src/docker/context/target</outputDirectory>
                  <resources>
                    <resource>
                      <directory>${basedir}/target</directory>
                    </resource>
                  </resources>
                </configuration>
              </execution>
            </executions>
          </plugin>

          <!-- 2. Assemble the docker image -->

          <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.4.13</version>
            <executions>
              <execution>
                <id>default</id>
                <goals>
                  <goal>build</goal>
                </goals>
              </execution>
            </executions>
            <configuration>
              <contextDirectory>src/docker/context</contextDirectory>
              <repository>192.168.56.92/dockeraws/daws</repository>
            </configuration>
          </plugin>

        </plugins>

      </build>

    </profile>

  </profiles>

</project>
```

Also, create the empty source folders, if they are not yet created. In linux you can do:

```bash
mkdir -p src/main/java/com/app
mkdir -p src/main/resources/static
mkdir -p src/docker/context
```

Change the commands above accordingly for Windows or other OS as needed, or use your IDE to create them.

To check the `pom.xml` file is correct, run Maven once using:

```bash
mvn clean compile
```

It should report `BUILD SUCCESS` at the end.


## Part 2 &mdash; The Database

In this part we prepare a very simple SQL Server database with one table.

Run the following SQL script in your SQL Server database:

```sql
create table product (
  id varchar(8) primary key not null,
  name varchar(20) not null
);

insert into product (id, name) values ('1015', 'Theremin');
insert into product (id, name) values ('4407', 'Daguerrotype');
insert into product (id, name) values ('5003', 'Gramophone');
```


## Part 3 &mdash; The Application

In this part we write a simple app that implements a REST service that provides information about products.


### A Simple Spring Boot Application

Let's write a simple application that given a product ID return the product information. Create the application 
class `src/main/java/com/app/App.java` as:

```java
package com.app;

import javax.sql.DataSource;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import com.zaxxer.hikari.HikariDataSource;

@Configuration
@SpringBootApplication
@ComponentScan
public class App {

  public static void main(String[] args) {
    SpringApplication.run(App.class, args);
  }

  @Bean
  @ConfigurationProperties("spring.datasource")
  public DataSource dataSource() {
    return DataSourceBuilder.create().type(HikariDataSource.class).build();
  }

}
```

Then, create the REST service class `src/main/java/com/app/ProductService.java`:

```java
package com.app;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

@RequestMapping("/product")
@RestController
public class ProductService {

  @Autowired
  private DataSource datasource;

  @GetMapping("/{id}")
  @ResponseBody
  public ResponseEntity<?> getProduct(@PathVariable String id) throws SQLException {
    try (Connection conn = this.datasource.getConnection();
        PreparedStatement st = conn.prepareStatement("select * from product where id = ?");) {
      st.setString(1, id);
      try (ResultSet rs = st.executeQuery()) {
        if (rs.next()) {
          Product p = new Product(id, rs.getString(2));
          return new ResponseEntity<Product>(p, HttpStatus.OK);
        } else {
          return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }
      }
    }
  }

  public static class Product {
    private String id;
    private String name;
    public Product(String id, String name) { this.id = id; this.name = name; }
    public String getId() { return this.id; }
    public String getName() { return this.name; }
  }

}
```

Let's also create a static resource &ndash; an HTML page &ndash; to serve it as well. Create the file `src/main/resources/static/hello.html` with the content:

```html
<html>
<body>
<h3>Hello World Docker!</h3>
<p>This is a static page served by the Spring Boot app.</p>
<p>Any resources (html, css, images, etc.) are automatically served from <code>src/main/resources/static</code>.</p>
</body>
</html>
```

That's it! Your app is ready.


### Prepare the Dockerfile

Docker needs one file to assemble the image. Create the file `src/docker/context/Dockerfile`:

```docker
FROM amazoncorretto:11.0.17
RUN mkdir /home/user/
EXPOSE 8080
ADD target/app.jar /home/user/app.jar
CMD java -jar /home/user/app.jar 
```

**Note**: The `FROM` instruction specifies the base image for the new image we are creating. For a Spring Boot app we only need a bare-bones image with Java installed. If you need a different 
java version look for another image at [Amazon Corretto Images](https://hub.docker.com/_/amazoncorretto). Alternatively, you can use any other base image from a **reputable** source.


### Build and Run Locally

Let's set up the database connection details to run locally. Create the file `./application.properties` with the content:

```properties
spring.datasource.driver-class-name=com.microsoft.sqlserver.jdbc.SQLServerDriver
spring.datasource.jdbc-url=jdbc:sqlserver://192.168.56.51:1433;encrypt=true;trustServerCertificate=true;
spring.datasource.username=admin
spring.datasource.password=admin
```

**Note**: Change the database IP, port, and credentials according to your database.

Now, let's run the application:

```bash
mvn spring-boot:run
```

You see the Spring Boot banner and the application starting. Let's test it:

```bash
curl -w '\n%{http_code}\n' localhost:8080//product/4407
{"id":"4407","name":"Daguerrotype"}
200
```

Our app received the product id `4407`, connected to the database, retrieved the product name, and returned it to 
the caller as a JSON document. The HTTP response code is `200` (OK).

Now, commit your changes to git (if you haven't done so yet):

```bash
git add *
git commit -m 'Example 1'
git push
```


## Part 4 &mdash; Running in a Docker Container

In this part we build our app as a standalone jar file, and we assemble it as part of a docker image. Then, we run this image in a container.


### Build a Local Docker Image

Log in to the docker server and retrieve the source code:

```bash
git pull
```

Then, build docker image in the local docker repo. This time we enable the `docker` profile of our Maven build:

```bash
mvn -P docker clean package
```

The first time it runs, it'll download the base docker image you selected with the `FROM` instruction; you'll need to wait for a 
couple of minutes until all is downloaded and assembled. Once done, check the docker image is built:

```bash
$ docker image ls
REPOSITORY                     TAG       IMAGE ID       CREATED          SIZE
192.168.56.92/dockeraws/daws   latest    8d00b98fd138   19 minutes ago   464MB
$
```

### Run the Image in a Container

The database connection details are not hardcoded inside the docker image and need to be provided by DevOps when deploying the app.
We will store them in a file in the docker server and we'll make this file available to the running container.

If the deployment location in the docker server is `/home/user1/deployments/app/`, then create the file `/home/user1/deployments/app/config/application.properties` with the content:

```properties
spring.datasource.driver-class-name=com.microsoft.sqlserver.jdbc.SQLServerDriver
spring.datasource.jdbc-url=jdbc:sqlserver://192.168.56.51:1433;encrypt=true;trustServerCertificate=true;
spring.datasource.username=admin
spring.datasource.password=admin
```

Start a docker container with this image:

```bash
$ cd /home/user1/deployments/app
$ docker run -p 80:8080 -v /home/user/config:/home/user1/deployments/app/config 192.168.56.92/dockeraws/daws
```

Notice we are now using port 80. Even if the Spring Boot app listens on port 8080, as specified in the command line the container 
binds the *internal* port 8080 to the *external* port 80. If we ran multiple instances of the app, each one would need to use 
a different external port.

You'll now see the log coming from the container:

```log
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.4.RELEASE)

[main] INFO com.app.App - Starting App on terminus with PID 18894 (/dockeraws/pocs/poc-docker-build/target/classes started by valarcon in /dockeraws/pocs/poc-docker-build)
[main] INFO com.app.App - No active profile set, falling back to default profiles: default
[main] INFO org.springframework.boot.web.embedded.tomcat.TomcatWebServer - Tomcat initialized with port(s): 8080 (http)
[main] INFO org.apache.catalina.core.StandardService - Starting service [Tomcat]
[main] INFO org.apache.catalina.core.StandardEngine - Starting Servlet engine: [Apache Tomcat/9.0.38]
[main] INFO org.apache.catalina.core.ContainerBase.[Tomcat].[localhost].[/] - Initializing Spring embedded WebApplicationContext
[main] INFO org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext - Root WebApplicationContext: initialization completed in 548 ms
[main] INFO org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor - Initializing ExecutorService 'applicationTaskExecutor'
[main] INFO org.springframework.boot.autoconfigure.web.servlet.WelcomePageHandlerMapping - Adding welcome page: class path resource [static/index.html]
[main] INFO org.springframework.boot.web.embedded.tomcat.TomcatWebServer - Tomcat started on port(s): 8080 (http) with context path ''
[main] INFO com.app.App - Started App in 1.202 seconds (JVM running for 1.465)
```

The Spring Boot application is now running in the container and ready to respond to our REST calls. Open another terminal window and test it:

```bash
$ curl -w '\n%{http_code}\n' 192.168.56.94:80/product/4407
{"id":"4407","name":"Daguerrotype"}
200
$
```

Let's search for a non-existent product:

```bash
$ curl -w '\n%{http_code}\n' 192.168.56.94:80/product/123

404
$
```


Now, let's look at the static page. Open your browser and get the page `http://192.168.56.92:80/hello.html` (change the server IP to your docker server's IP).

You see the page that says:

> Hello World Docker!

That's it! You have a Spring Boot application running in a docker container.




