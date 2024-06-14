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
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
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
   package com.example.demo.controller;

   import com.example.demo.service.DataService;
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
   package com.example.demo.service;

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
   package com.example.demo.service;

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
   package com.example.demo.controller;

   import com.example.demo.service.DataService;
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
   package com.example.demo;

   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.boot.test.web.client.TestRestTemplate;
   import org.springframework.http.ResponseEntity;
   import org.springframework.test.context.ActiveProfiles;

   import static org.junit.jupiter.api.Assertions.assertEquals;

   @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
   @ActiveProfiles("test")
   public class DataIntegrationTest {

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

### Conclusion
In this blog post, we've covered how to set up unit and integration tests in a Spring Boot application, along with generating and publishing test coverage reports using Maven. Implementing thorough testing and monitoring coverage helps ensure your application is robust and secure.

Happy testing!

---

### Additional Tips
- Regularly run tests and review coverage reports to maintain high code quality.
- Use more advanced features of testing frameworks and coverage tools as your project grows.
- Integrate other tools like SonarQube for comprehensive code analysis and quality checks.

By following this guide, you'll be well-equipped to implement effective testing strategies in your Spring Boot applications.
