---
layout: post
---

**Testing** is a crucial part of software development, ensuring the correctness and reliability of your application. In this blog post, we'll demonstrate how to configure and run **unit** and **integration tests** in a Spring Boot application using Maven. We'll also cover how to generate and publish test coverage reports.

### Prerequisites
Before we start, ensure you have the following:
- A basic understanding of Spring Boot and Maven.
- Maven installed.
- A development environment set up for Java.

### Setting Up a Simple Spring Boot REST API

1. **Create a Spring Boot Application:**
   Generate a new Spring Boot project using [Spring Initializr](https://start.spring.io/).

2. **Add Dependencies in `pom.xml`:**
   Include necessary dependencies for Spring Boot, testing, and coverage reporting.

   ```xml
    <dependencies>
        <!-- Spring Boot Web Starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Boot Starter Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
   ```

3. **Create a Simple REST Controller and Service:**

   **Controller:**
   ```java
   package com.github.innovationforge.controller;
   
   import com.github.innovationforge.service.DataService;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   import java.util.List;
   
   @RestController
   @RequestMapping("/api/data")
   public class DataController {
   
       @Autowired
       private DataService dataService;
   
       @GetMapping
       public List<String> getData() {
           return dataService.getData();
       }
   }

   ```

   **Service:**
   ```java
   package com.github.innovationforge.service;
   
   import org.springframework.stereotype.Service;
   
   import java.util.Arrays;
   import java.util.List;
   
   @Service
   public class DataService {
   
       private final List<String> dataList = Arrays.asList("Data1", "Data2", "Data3");
   
       public List<String> getData() {
           return dataList;
       }
   }
   ```

### Writing Unit Tests

1. **Unit Test for Service:**
   ```java
   package com.github.innovationforge.service;
   
   import org.junit.jupiter.api.Test;
   import java.util.List;
   import static org.junit.jupiter.api.Assertions.assertEquals;
   
   public class DataServiceTest {
   
       private final DataService dataService = new DataService();
   
       @Test
       public void testGetData() {
           List<String> data = dataService.getData();
           assertEquals(3, data.size());
           assertEquals("Data1", data.get(0));
       }
   }
   ```

2. **Unit Test for Controller:**
   ```java
   package com.github.innovationforge.controller;
   
   import com.github.innovationforge.service.DataService;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
   import org.springframework.boot.test.mock.mockito.MockBean;
   import org.springframework.test.web.servlet.MockMvc;
   
   import java.util.Arrays;
   
   import static org.mockito.Mockito.when;
   import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
   
   @WebMvcTest(DataController.class)
   public class DataControllerTest {
   
       @Autowired
       private MockMvc mockMvc;
   
       @MockBean
       private DataService dataService;
   
       @Test
       public void testGetData() throws Exception {
           when(dataService.getData()).thenReturn(Arrays.asList("Data1", "Data2", "Data3"));
   
           mockMvc.perform(get("/api/data"))
                   .andExpect(status().isOk())
                   .andExpect(jsonPath("$[0]").value("Data1"))
                   .andExpect(jsonPath("$[1]").value("Data2"))
                   .andExpect(jsonPath("$[2]").value("Data3"));
       }
   }
   ```

### Writing Integration Tests

1. **Integration Test:**
   ```java
   package com.github.innovationforge;
   
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.boot.test.web.client.TestRestTemplate;
   import org.springframework.http.ResponseEntity;
   import org.springframework.test.context.ActiveProfiles;
   
   import static org.junit.jupiter.api.Assertions.assertEquals;
   
   @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
   @ActiveProfiles("test")
   public class DataIT {
   
       @Autowired
       private TestRestTemplate restTemplate;
   
       @Test
       public void testGetData() {
           ResponseEntity<String[]> response = restTemplate.getForEntity("/api/data", String[].class);
           String[] data = response.getBody();
           assertEquals(3, data.length);
           assertEquals("Data1", data[0]);
       }
   }

   ```

### Generating and Publishing Test Coverage Reports

1. **Add JaCoCo Plugin to `pom.xml`:**
   ```xml
   <build>
       <plugins>
           <plugin>
               <groupId>org.jacoco</groupId>
               <artifactId>jacoco-maven-plugin</artifactId>
               <version>0.8.7</version>
               <executions>
                   <execution>
                       <goals>
                           <goal>prepare-agent</goal>
                       </goals>
                   </execution>
                   <execution>
                       <id>report</id>
                       <phase>test</phase>
                       <goals>
                           <goal>report</goal>
                       </goals>
                   </execution>
               </executions>
           </plugin>
       </plugins>
   </build>
   ```

2. **Run Tests and Generate Report:**
   ```sh
   mvn clean test
   mvn jacoco:report
   ```

3. **Publishing Test Coverage Reports:**
   - Configure your CI/CD pipeline to run the above Maven commands and publish the generated reports.
   
   **Example with GitHub Actions:**
   ```yaml
   name: CI

   on:
     push:
       branches:
         - main
     pull_request:
       branches:
         - main

   jobs:
     build:
       runs-on: ubuntu-latest

       steps:
       - name: Checkout code
         uses: actions/checkout@v2

       - name: Set up JDK 11
         uses: actions/setup-java@v2
         with:
           java-version: '11'

       - name: Build with Maven
         run: mvn clean test jacoco:report

       - name: Upload coverage report
         uses: actions/upload-artifact@v2
         with:
           name: coverage-report
           path: target/site/jacoco
   ```


### Skipping Tests with Maven Profiles
In real-life scenarios, running all unit and integration tests for a large application can be time-consuming. You can use Maven profiles to provide developers with options to skip tests when necessary. Here's how you can configure your Spring Boot application with Maven profiles to skip tests:

#### Add Maven Profiles to `pom.xml`
```xml
<project>
    ...
    <profiles>
        <!-- Profile to skip all tests -->
        <profile>
            <id>skip-tests</id>
            <properties>
                <maven.test.skip>true</maven.test.skip>
                <skip.unit.tests>true</skip.unit.tests>
                <skip.integration.tests>true</skip.integration.tests>
            </properties>
        </profile>

        <!-- Profile to run only unit tests -->
        <profile>
            <id>unit-tests-only</id>
            <properties>
                <skip.unit.tests>false</skip.unit.tests>
                <skip.integration.tests>true</skip.integration.tests>
            </properties>
        </profile>

        <!-- Profile to run only integration tests -->
        <profile>
            <id>integration-tests-only</id>
            <properties>
                <skip.unit.tests>true</skip.unit.tests>
                <skip.integration.tests>false</skip.integration.tests>
            </properties>
        </profile>
    </profiles>
    ...
</project>
```

#### Configure Surefire and Failsafe Plugins
```xml
 <build>
     <plugins>
         <!-- Maven Surefire Plugin for unit tests -->
         <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-surefire-plugin</artifactId>
             <version>${maven-surefire-plugin.version}</version>
             <configuration>
                 <skipTests>${maven.test.skip}</skipTests>
                 <excludes>
                     <exclude>**/*IT.java</exclude>
                 </excludes>
                 <skip>${skip.unit.tests}</skip>
             </configuration>
         </plugin>

         <!-- Maven Failsafe Plugin for integration tests -->
         <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-failsafe-plugin</artifactId>
             <version>${maven-failsafe-plugin.version}</version>
             <executions>
                 <execution>
                     <goals>
                         <goal>integration-test</goal>
                         <goal>verify</goal>
                     </goals>
                 </execution>
             </executions>
             <configuration>
                 <skip>${maven.test.skip}</skip>
                 <skipTests>${maven.test.skip}</skipTests>
                 <skipITs>${skip.integration.tests}</skipITs>
                 <includes>
                     <include>**/*IT.java</include>
                 </includes>
             </configuration>
         </plugin>
     </plugins>
 </build>
```

#### Running with Maven Profiles
```sh
mvn clean verify
mvn clean verify -Pskip-tests
mvn clean verify -Punit-tests-only
mvn clean verify -Pintegration-tests-only
```
By configuring Maven profiles, you provide developers with flexible options to control the execution of unit and integration tests. This approach is particularly useful for larger projects where running all tests can be time-consuming. Developers can choose to run only the necessary tests based on their current needs, improving productivity and efficiency.

The Surefire and Failsafe plugins serve distinct purposes and are integral to a well-structured testing strategy in a Maven project. Here's a detailed explanation of why they are used alongside the JaCoCo plugin:

### Maven Surefire Plugin

- **Purpose**: The Surefire plugin is designed to run unit tests. These tests are typically fast and do not require the application to be fully started or to interact with external resources like databases or web services.
- **Configuration**: It is configured to run tests in the `src/test/java` directory.
- **Usage**: It executes tests that follow a naming convention like `*Test.java`.

### Maven Failsafe Plugin

- **Purpose**: The Failsafe plugin is specifically designed to run integration tests. These tests usually require the application to be started and often involve external systems and resources.
- **Configuration**: It is configured to run tests in the `src/test/java` directory but only those that follow a naming convention like `*IT.java`.
- **Usage**: It runs in the `integration-test` and `verify` phases, allowing the application to be packaged and possibly deployed before the tests are executed.

### JaCoCo Plugin

- **Purpose**: The JaCoCo plugin is used to measure code coverage of the tests run by Surefire and Failsafe plugins.
- **Configuration**: It can be configured to run during the test phases and to generate reports based on the coverage data.
- **Usage**: It can aggregate coverage data from both unit and integration tests to provide a comprehensive coverage report.

### Integration of Plugins

To achieve a comprehensive testing strategy, all three plugins are used together:

1. **Surefire for Unit Tests**: Ensures that unit tests are run quickly during the `test` phase.
2. **Failsafe for Integration Tests**: Ensures that integration tests are run during the `integration-test` and `verify` phases, allowing for proper application lifecycle management.
3. **JaCoCo for Coverage**: Collects coverage data from both Surefire and Failsafe test runs and generates a combined report.

### Example Configuration

Here’s an example of how you can configure these plugins in your `pom.xml`:

```xml
<project>
    ...
    <build>
        <plugins>
            ...
            <!-- Maven Surefire Plugin for unit tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${maven-surefire-plugin.version}</version>
                <configuration>
                    <skipTests>${maven.test.skip}</skipTests>
                    <excludes>
                        <exclude>**/*IT.java</exclude>
                    </excludes>
                    <skip>${skip.unit.tests}</skip>
                </configuration>
            </plugin>

            <!-- Maven Failsafe Plugin for integration tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>${maven-failsafe-plugin.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <skip>${maven.test.skip}</skip>
                    <skipTests>${maven.test.skip}</skipTests>
                    <skipITs>${skip.integration.tests}</skipITs>
                    <includes>
                        <include>**/*IT.java</include>
                    </includes>
                </configuration>
            </plugin>

            <!-- JaCoCo Maven Plugin for coverage reports -->
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>${jacoco-maven-plugin.version}</version>
                <executions>
                    <!-- Phase 'initialize' is typical for prepare-agent goal -->
                    <execution>
                        <id>jacoco-initialize</id>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                        <phase>initialize</phase>
                    </execution>
                    <!-- Generate report after unit tests -->
                    <execution>
                        <id>jacoco-report-unit-tests</id>
                        <phase>test</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                    <!-- Generate report after integration tests -->
                    <execution>
                        <id>jacoco-report-integration-tests</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    ...
</project>
```

### Running Tests and Generating Reports

1. **Run All Tests**:
    ```sh
    mvn clean verify
    ```

2. **Generate Coverage Report**:
    ```sh
    mvn jacoco:report
    ```
Your JaCoCo Maven plugin configuration looks generally correct for generating coverage reports for both unit tests and integration tests. However, there are a few aspects to double-check and adjust if necessary:

### Points to Verify and Adjust:

1. **Execution Order and Phases**:
   - Ensure that the `jacoco-maven-plugin` executions are correctly aligned with the Maven build phases where you expect coverage reports (`test` phase for unit tests and `verify` phase for integration tests).
   - The `<phase>` element specifies when the execution should occur relative to the Maven lifecycle phases (`initialize`, `test`, `verify`, etc.). Ensure that `prepare-agent` runs in the `initialize` phase, and `report` goals run in the appropriate phases (`test` for unit tests and `verify` for integration tests).

2. **Coverage Data Collection**:
   - Verify that the `prepare-agent` goal is properly configuring JaCoCo to collect coverage data for both unit and integration tests. This goal instruments your classes during the build to gather coverage information.

3. **Execution of Maven Commands**:
   - After making sure your `pom.xml` is correctly configured, run the following Maven command to execute your tests and generate the JaCoCo coverage reports:
     ```bash
     mvn clean verify
     ```
4. **Inspecting Reports**:
   - After the Maven build completes, navigate to the generated JaCoCo HTML reports (`target/site/jacoco/index.html`).
   - Check if the `DataIT` integration test class is listed in the classes with coverage data. If it's not listed, there may be issues with how the test is configured or executed.

### Debugging and Troubleshooting:

- **Debug Output**: Use `-X` flag with Maven commands (`mvn clean verify -X`) to enable debug output. This can help identify any issues or errors related to JaCoCo instrumentation or coverage reporting during the build process.

- **Reviewing Logs**: Examine Maven build logs for any warnings or errors related to JaCoCo plugin executions. Look for messages that indicate if `DataIT` or other integration tests are being processed for coverage.

### Conclusion

Using the Surefire and Failsafe plugins alongside JaCoCo ensures a well-structured and efficient testing strategy. Surefire handles fast unit tests, Failsafe handles more extensive integration tests, and JaCoCo provides a comprehensive coverage report for both, helping maintain high code quality and reliability.

While JaCoCo could technically run with just one type of test, combining all three plugins allows for better separation of concerns, better lifecycle management, and more accurate and comprehensive coverage reporting.

Happy coding and testing!

---

### Additional Tips
- Regularly run tests and review coverage reports to maintain high code quality.
- Use more advanced features of testing frameworks and coverage tools as your project grows.
- Integrate other tools like SonarQube for comprehensive code analysis and quality checks.

By following this guide, you'll be well-equipped to implement effective testing strategies in your Spring Boot applications.
