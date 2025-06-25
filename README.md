# mvnize-maven-plugin
Maven plugin to "mavenize" a set of files by generating a pom.xml that attaches these files.

The `mvnize-maven-plugin` features two goals with the following purposes:
* `attach`: Attaches a file to the `pom.xml` in order to prepare it for **publishing**. The POM is generated if it does not yet exist. Typically, the file extension becomes the Maven type, and the file name the Maven classifier. This goal updates the `pom.xml` with a `build-helper-maven-plugin` section that declares the appropriate file-to-artifact mappings.
* `consumer`: Reads in a `pom.xml`  (typically one generated using the `attach` goal) and derives a `consumer.pom.xml` for **consumption**. This goal reads the `build-helper-maven-plugin` configuration and adds all artifacts as **dependencies** to the `consumer.pom.xml`.

### Goal: attach

```bash
mvn org.aksw.maven.plugins:mvnize-maven-plugin:attach \     
  -DparentId=my.org:my-data-deployment-setup:0.0.1 \
  -DartifactId=my.org:pokedex-rdf-2023:1:ttl.bz2:data \
  -Dfile=pokedex-data-rdf.ttl.bz2
```

Notes:
* The GAV (groupId, artifact, version) of the `artifactId` must match that of an existing `pom.xml`.
* that Maven does not allow dependencies of multiple versions. So if we want to use datasets that were published at different dates,
its best to encode the publishing data in the artifact name, such as `pokedex-rdf-2023`. The version field can then be used to adress fixes of historic data, i.e. the version field refers to changes of data published at that time. Since such changes happen very rarely, a simple sequential id should be sufficient rather than the typical 3-component semantic version.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>my.org</groupId>
    <artifactId>my-data-deployment-setup</artifactId>
    <version>0.0.1</version>
  </parent>
  <groupId>my.org</groupId>
  <artifactId>pokedex-rdf-2023</artifactId>
  <version>1</version>
  <packaging>pom</packaging>
  <properties>
    <build-helper-maven-plugin.version>3.3.0</build-helper-maven-plugin.version>
  </properties>
  <build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>build-helper-maven-plugin</artifactId>
        <version>${build-helper-maven-plugin.version}</version>
        <executions>
          <execution>
            <id>attach-artifacts</id>
            <phase>package</phase>
            <goals>
              <goal>attach-artifact</goal>
            </goals>
            <configuration>
              <artifacts>
                <artifact>
                  <file>pokedex-data-rdf.ttl.bz2</file>
                  <type>ttl.bz2</type>
                  <classifier>data</classifier>
                </artifact>
              </artifacts>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>


```

### Goal: consume


```bash
mvn org.aksw.maven.plugins:mvnize-maven-plugin:consume
```

There now exists a `consumer.pom.xml` with the following content:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>my-group</groupId>
  <artifactId>my-artifact</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>
  <dependencies>
    <dependency>
      <groupId>my.org</groupId>
      <artifactId>pokedex-rdf-2023</artifactId>
      <version>1</version>
      <type>ttl.bz2</type>
      <classifier>data</classifier>
    </dependency>
  </dependencies>
</project>
```

