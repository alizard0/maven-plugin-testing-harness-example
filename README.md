# maven-plugin-testing-harness-example
After wasting a few hours trying to figure out why the official example (https://maven.apache.org/plugin-testing/maven-plugin-testing-harness/getting-started/index.html) never works, I decided to share my findings.

### pom.xml dependencies
**Note** don't mess up with this list of dependencies, they work for me and it took me several hours to get them together 😄
```xml
  <properties>
      <maven.compiler.target>1.8</maven.compiler.target>
      <maven.compiler.source>1.8</maven.compiler.source>
      <avro.version>1.10.0</avro.version>
      <jackson-dataformat-avro.version>2.13.0</jackson-dataformat-avro.version>
      <lombok.version>1.18.20</lombok.version>
      <commons-lang.version>3.12.0</commons-lang.version>
      <reflections.version>0.10.2</reflections.version>
      <mvn.plugin-tools.version>3.5.2</mvn.plugin-tools.version>
      <mvn.plugin.version>3.5.4</mvn.plugin.version>
      <junit.version>5.8.1</junit.version>
      <maven-compat.version>3.8.4</maven-compat.version>
      <maven-plugin-testing-harness.version>3.3.0</maven-plugin-testing-harness.version>
      <maven-resolver-api.version>1.7.2</maven-resolver-api.version>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.apache.avro</groupId>
      <artifactId>avro</artifactId>
      <version>${avro.version}</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.dataformat</groupId>
      <artifactId>jackson-dataformat-avro</artifactId>
      <version>${jackson-dataformat-avro.version}</version>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>${lombok.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>${commons-lang.version}</version>
    </dependency>
    <dependency>
      <groupId>org.reflections</groupId>
      <artifactId>reflections</artifactId>
      <version>${reflections.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-plugin-api</artifactId>
      <version>3.8.4</version>
    </dependency>
    <dependency>
      <groupId>org.apache.maven.plugin-tools</groupId>
      <artifactId>maven-plugin-annotations</artifactId>
      <scope>provided</scope>
      <version>${mvn.plugin-tools.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-core</artifactId>
      <version>${mvn.plugin.version}</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.maven/maven-compat -->
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-compat</artifactId>
      <version>${maven-compat.version}</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine -->
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-engine</artifactId>
      <version>${junit.version}</version>
      <scope>test</scope>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.maven.plugin-testing/maven-plugin-testing-harness -->
    <dependency>
      <groupId>org.apache.maven.plugin-testing</groupId>
      <artifactId>maven-plugin-testing-harness</artifactId>
      <version>${maven-plugin-testing-harness.version}</version>
      <scope>test</scope>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.maven.resolver/maven-resolver-api -->
    <dependency>
      <groupId>org.apache.maven.resolver</groupId>
      <artifactId>maven-resolver-api</artifactId>
      <version>${maven-resolver-api.version}</version>
      <scope>test</scope>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.mockito/mockito-core -->
    <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-core</artifactId>
      <version>4.1.0</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
```

### your mojo
```java
@Mojo(name = "run-mojo", defaultPhase = LifecyclePhase.COMPILE)
public class RunMojo extends AbstractMojo {
  @Parameter(defaultValue = "${project}")
  private MavenProject project;

  @Parameter(property = "param1", required = true, defaultValue = "something-you-need")
  private String param1;
  
  /**
   * Execute mojo
   */
  @SneakyThrows
  @Override
  public void execute() {
    log.info("Executing a custom maven plugin");
  }
  
  // if you need to inject the project or the parameter, create a setter
  void setProject(MavenProject project) {
    this.project = project;
  }
  
  void setParam1(String param1) {
    this.param1 = param1;
  }
}
```

### testing your mojo
```java
public class RunMojoTest extends AbstractMojoTestCase {

  private MavenProject mockProject;

  protected void setUp() throws Exception {
    super.setUp();
  }

  protected void tearDown() throws Exception {
    super.tearDown();
  }

  public void testRunMojo() throws Exception {
    File pom = getTestFile(getBasedir(), "src/test/resources/pom.xml");
    assertNotNull(pom);
    assertTrue(pom.exists());

    RunMojo mojo = (RunMojo) lookupMojo("run-mojo", pom);
    mojo.setProject(mockProject);

    mojo.execute();
  }
```

### how to use it
Let's suppose you want to use this maven plugin in a different project, so you to add the plugin in the project pom.
```xml
<build>
  <plugins>
      // you might have more plugins already here 
      <plugin>
          <groupId>com.your.group.id</groupId>
          <artifactId>run-mojo-plugin</artifactId>
          <version>1.0.0-SNAPSHOT</version>
          <executions>
              <execution>
                  <goals>
                      <goal>run-mojo</goal>
                  </goals>
              </execution>
          </executions>
          <configuration>
              <param1>some-important-parameter</param1>
              <project implementation="org.apache.maven.plugin.testing.stubs.MavenProjectStub"/>
          </configuration>
      </plugin>
  </plugins>
 </build>
```

🎉
