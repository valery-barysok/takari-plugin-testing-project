## Takari Maven Plugin Testing Framework

Small, cohesive, one-stop library for developing unit and integration tests for 
Maven plugins. Provides alternative to, and arguably supersedes, 
maven-plugin-testing-harness and maven-verifier.

Features and benefits

* Convenient junit4-based API
* Flexible unit test mojo configuration API simplifies test project setup 
  and maintenance
* Collocate main and test code in the same build module
* No need to install or deploy plugins to run tests
* Run plugins integration tests against multiple Maven versions
* Integration with takari-lifecycle and incrementalbuild library
* Fully supported by Maven Development Tools m2e extension
* [2.1.0+] full support for all maven versions starting with 3.0

### Unit testing

pom.xml

    <packaging>takari-maven-plugin</packaging>
    
    <dependency>
      <groupId>io.takari.maven.plugins</groupId>
      <artifactId>takari-plugin-testing</artifactId>
      <version>2.0.0</version>
      <scope>test</scope>
    </dependency>
    
    <!-- required if not already present in main dependencies -->
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-core</artifactId>
      <version>${mavenVersion}</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-compat</artifactId>
      <version>${mavenVersion}</version>
      <scope>test</scope>
    </dependency>

test code

    public class PluginUnitTest {
      @Rule
      public final TestResources resources = new TestResources();
    
      @Rule
      public final TestMavenRuntime maven = new TestMavenRuntime();
    
      @Test
      public void test() throws Exception {
        File basedir = resources.getBasedir("testproject");
        maven.executeMojo(basedir, "mymojo", newParameter("name", "value");
        assertFilesPresent(basedir, "target/output.txt");
      }
    }

### Integration testing

pom.xml

    <packaging>takari-maven-plugin</packaging>
        
    <dependency>
      <groupId>io.takari.maven.plugins</groupId>
      <artifactId>takari-plugin-testing</artifactId>
      <version>2.0.0</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>io.takari.maven.plugins</groupId>
      <artifactId>takari-plugin-integration-testing</artifactId>
      <version>2.0.0</version>
      <type>pom</type>
      <scope>test</scope>
    </dependency>


test

    @RunWith(MavenJUnitTestRunner.class)
    @MavenVersions({"3.2.3", "3.2.5"})
    public class PluginIntegrationTest {
      @Rule
      public final TestResources resources = new TestResources();
    
      public final MavenRuntime maven;
    
      public PluginIntegrationTest(MavenRuntimeBuilder mavenBuilder) {
        this.maven = mavenBuilder.withCliOptions("-B", "-U").build();
      }
    
      @Test
      public void test() throws Exception {
        File basedir = resources.getBasedir("basic");
        maven.forProject(basedir)
          .withCliOption("-Dproperty=value")
          .withCliOption("-X")
          .execute("deploy")
          .assertErrorFreeLog()
          .assertLogText("some build message");
      }
    }

### Hudson users beware
 
Hudson Maven 3 does not use -s/--settings standard maven command
option to enable "managed" settings.xml files in maven builds. Because of this, test 
JVMs launched by Maven builds are not able to use the settings.xml files will 
fail with artifact resolution errors. To workaround, use filesystem-based settings xml
files and pass them using -s/--settings options.
