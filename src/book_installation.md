# Installation

## Install Java

Torq for Java requires JDK 17+. Use your favorite JDK provider or one of the following:

* AWS: <https://aws.amazon.com/corretto/>
* Microsoft: <https://learn.microsoft.com/en-us/java/openjdk/download>

## Install Torq JARs

Torq consists of four core JARs, one optional server JAR, and one examples JAR. The core jars are 100% Java with no external dependencies. The server JAR provides dynamic API support and depends on Jetty 12. The examples JAR depends on all the other JARs.

### Core JARs

```xml
<dependency>
    <groupId>org.torqlang</groupId>
    <artifactId>torqlang-local</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

```xml
<dependency>
    <groupId>org.torqlang</groupId>
    <artifactId>torqlang-lang</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

```xml
<dependency>
    <groupId>org.torqlang</groupId>
    <artifactId>torqlang-klvm</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

```xml
<dependency>
    <groupId>org.torqlang</groupId>
    <artifactId>torqlang-util</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

### Server JAR

```xml
<dependency>
    <groupId>org.torqlang</groupId>
    <artifactId>torqlang-server</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

### Examples JAR

```xml
<dependency>
    <groupId>org.torqlang</groupId>
    <artifactId>torqlang-examples</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

## Install Torq JARs using Maven

To use Maven, you need to:

1. Configure your Maven settings file `settings.xml`
    1. Add a GitHub profile
    2. Add your GitHub identity and personal access token
2. Configure your project POM file `pom.xml`
    1. Add the Torq GitHub repository

For detailed guidance, see
GitHub, [Learn more about Maven](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry).

### Configure `settings.xml`

#### Add a GitHub profile

Add a GitHub profile to your Maven settings. Replace `OWNER` with `torq-lang` and `REPOSITORY` with `torq-jv`.

~~~xml
<profile>
    <id>github</id>
    <repositories>
        <repository>
            <id>central</id>
            <url>https://repo1.maven.org/maven2</url>
        </repository>
        <repository>
            <id>github</id>
            <url>https://maven.pkg.github.com/OWNER/REPOSITORY</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>
</profile>
~~~

#### Add your GitHub identity and personal access token

Add a GitHub server to your list of servers. Replace `USERNAME` with your GitHub user ID and replace `TOKEN` with your
GitHub personal access token.

~~~xml
<servers>
    <server>
        <id>github</id>
        <username>USERNAME</username>
        <password>TOKEN</password>
    </server>
</servers>
~~~

### Configure `pom.xml`

Your POM file must contain two declarations:

1. A repository declaration
2. A dependency for each Torq JAR being used

~~~xml
<project>

    <!-- Declare the GitHub repository for Torq -->
    <repositories>
        <repository>
            <id>github</id>
            <url>https://maven.pkg.github.com/torq-lang/torq-jv</url>
        </repository>
    </repositories>

    <!-- Declare a dependency for each Torq JAR used -->
    <dependencies>
        <dependency>
           <groupId>org.torqlang</groupId>
           <artifactId>torqlang-server</artifactId>
           <version>1.0-SNAPSHOT</version>
        </dependency>       
        <dependency>
            <groupId>org.torqlang</groupId>
            <artifactId>torqlang-local</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.torqlang</groupId>
            <artifactId>torqlang-lang</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.torqlang</groupId>
            <artifactId>torqlang-util</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.torqlang</groupId>
            <artifactId>torqlang-klvm</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.torqlang</groupId>
            <artifactId>torqlang-examples</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
~~~