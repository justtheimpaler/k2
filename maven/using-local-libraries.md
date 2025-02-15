# Using Local Libraries in Maven

Even though the vast majority of libraries used in a Maven java application are published in [Central](https://central.sonatype.com/?smo=true) or other repositories directly accessible to Maven, there are exceptions to the rule.

Some libraries are not publicly available but only through a license agreement; other libraries had never been published to any public package repository for a variety of reasons.

If we need to use a library in our application that cannot be published globally we can either publish it in a organization-wide repository/registry or store it in inside our project using a local repository. This article covers the latter. Follow the steps described below to do so.

## 1. Preparing a Maven Repository Local to the Project

You can create a Maven repository local to the project as a plain folder structure included in your project. The main benefit of this solution is that you don't need any extra server to act as a package repository. The library is stored in the git project along with the source code of the project and can be directly referenced by your build.

Create the local directory `local-maven-repository` (or any other name) in your project that will include the local libraries. As you see this folder will be stored in the source code repository as any other folder in the project.

```bash
cd $PROJECT_HOME
mkdir local-maven-repository
```

Now you have an empty local repository.

## 2. Install Your Local Libraries in the Local Maven Repository

You can now *install* the local libraries in this local repository. You'll need to define the Maven *coordinates* for each library you want to install in here. That is, you need to define the following three values for each library. Mind that these coordinates will be used to reference them from your build.

- groupId
- artifactId
- version

For example, if you want to use the library `/home/myself/libs/charting-2d.jar` in your project using the coordinates:

- groupId: **local**
- artifactId: **charting-2d**
- version: **1.0.0**

Install them using:

```bash
cd $PROJECT_HOME
mvn org.apache.maven.plugins:maven-install-plugin:2.5.2:install-file -Dfile=/home/myself/libs/charting-2d.jar -DgroupId=local -DartifactId=charting-2d -Dversion=1.0.0 -Dpackaging=jar -DlocalRepositoryPath=local-maven-repository
```
**Note**: Note all four parameters used in the command above: the library full path and the three coordinates. You need to replace these values accordingly for your specific library(ies).

**Note**: The full absolute path of the JAR file is typically needed, not a relative one.

Run the command above once per library that you need to install. Once completed, all libraries will be stored inside this folder. You can even install multiple versions of each library if you need to; as long as the coordinates are different and unique, all should be good.

## 3. Reference the Local Maven Repository

To use this repository from the Maven build you'll need to declare it in your `pom.xml` file. Right after the `<dependencies>` section add:

```xml
  <repositories>
    <repository>
      <id>local-maven-repository</id>
      <url>file:///${project.basedir}/local-maven-repository</url>
    </repository>
  </repositories>
```

## 4. Use the Libraries in the Build

Finally, you can reference the local libraries from the build as you would do with any other library.
For example, to use the library defined in the example above you can declare the dependency:

```xml
  <dependency>
    <groupId>local</groupId>
    <artifactId>charting-2d</artifactId>
    <version>1.0.0</version>
  </dependency>
```

That's it! Happy building...





