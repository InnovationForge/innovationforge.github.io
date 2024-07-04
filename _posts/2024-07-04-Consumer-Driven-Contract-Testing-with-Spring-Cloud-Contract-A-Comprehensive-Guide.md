---
layout: post
---

In the world of microservices, ensuring smooth communication between various services is crucial. One effective way to achieve this is through Consumer-Driven Contract Testing (CDCT). This approach focuses on defining the interactions between services from the consumer's perspective, thereby ensuring that any changes in the provider do not break the consumer's functionality. In this blog post, we'll delve into the concept of CDCT and demonstrate how to implement it using Spring Cloud Contract.

## Understanding Consumer-Driven Contract Testing (CDCT)

Consumer-Driven Contract Testing is a technique where the consumer of a service defines the expectations for the provider. These expectations are written in the form of contracts, which are then shared with the provider. The provider uses these contracts to ensure that any changes made do not violate the consumer's expectations.

The main advantages of CDCT include:
- **Early Detection of Integration Issues**: By defining contracts early, potential integration issues are identified before deployment.
- **Clear Communication**: Contracts serve as a clear and precise way to communicate expectations between teams.
- **Continuous Integration**: CDCT can be integrated into the CI/CD pipeline, ensuring that any changes are automatically verified.

## Spring Cloud Contract

Spring Cloud Contract is a project within the Spring ecosystem that provides tools to implement CDCT. It includes:
- **Contract Definition Language (DSL)**: A DSL written in Groovy to define contracts.
- **Spring Cloud Contract Verifier**: A tool that verifies if the provider meets the consumer's expectations based on the defined contracts.

## Implementing CDCT with Spring Cloud Contract

Let's walk through a demo where we have a provider and a consumer application, both built using Spring Boot.

### Provider Application

#### POM File
First, we need to set up the dependencies in the `pom.xml` of the provider application.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.github.innovationforge</groupId>
    <artifactId>provider-application</artifactId>
    <version>1.0.0</version>
    <name>provider-application</name>
    <description>provider-application</description>
    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2023.0.2</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.4.0</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-contract-verifier</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-contract-maven-plugin</artifactId>
                <version>4.1.3</version>
                <extensions>true</extensions>
                <configuration>
                    <testFramework>JUNIT5</testFramework>
                    <baseClassForTests>
                        com.github.innovationforge.BaseTestClass
                    </baseClassForTests>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

#### Controller
Next, we define a simple controller that returns a greeting message.

```java
package com.github.innovationforge;

import ch.qos.logback.core.util.StringUtil;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;

@RestController
@RequestMapping("/provider")
public class ProviderController {

    @GetMapping(produces = "application/json")
    public ResponseEntity<ProviderResponse> getProviderMessage(@RequestParam(value = "name", required = false) String name){
        return ResponseEntity.ok(ProviderResponse.builder()
                .message(name != null ? "Hello " + StringUtil.capitalizeFirstLetter(name) : "Hello World!")
                .timestamp(LocalDateTime.now())
                .build());
    }
}
```

#### Base Test Class
We need to set up a base test class for the contract tests.

```java
package com.github.innovationforge;

import io.restassured.module.mockmvc.RestAssuredMockMvc;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.cloud.contract.verifier.messaging.boot.AutoConfigureMessageVerifier;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
@AutoConfigureMessageVerifier
@DirtiesContext
public class BaseTestClass {

    @Autowired
    private WebApplicationContext webApplicationContext;

    private MockMvc mockMvc;

    @BeforeEach
    public void setup() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
        RestAssuredMockMvc.mockMvc(mockMvc);
    }
}
```

#### Contracts
Contracts are defined in the `contracts` directory. Here are two example contracts:

**ProviderControllerShouldReturnDefaultGreeting.groovy**
```groovy
package contracts

import org.springframework.cloud.contract.spec.Contract

Contract.make {
    description "should return 'Hello World!' when no name is provided"

    request {
        method GET()
        url("/provider")
    }

    response {
        status 200
        body(
                message: "Hello World!",
                timestamp: $(regex('.*'))
        )
        headers {
            contentType(applicationJson())
        }
    }
}
```

**ProviderControllerShouldReturnGreetingWithName.groovy**
```groovy
package contracts

import org.springframework.cloud.contract.spec.Contract

Contract.make {
    description "should return a greeting message with the name capitalized if provided, otherwise 'Hello World!'"

    request {
        method GET()
        url("/provider") {
            queryParameters {
                parameter("name", "john")
            }
        }
    }

    response {
        status 200
        body(
                message: "Hello John",
                timestamp: $(regex('.*'))
        )
        headers {
            contentType(applicationJson())
        }
    }
}
```

### Consumer Application

#### Controller
In the consumer application, we define a controller that consumes the provider service.

```java
package com.github.innovationforge;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestClient;

@RestController
@RequestMapping("/consumer")
@RequiredArgsConstructor
@Slf4j
public class ConsumerController {

    private final RestClient restClient;

    @GetMapping
    public ResponseEntity<ConsumerResponse> getConsumer(@RequestParam(value = "name", required = false) String name){
        ConsumerResponse consumerResponse = restClient.get()
                .uri(name != null ? "http://localhost:8081/provider?name=" + name : "http://localhost:8081/provider")
                .accept(MediaType.APPLICATION_JSON)
                .retrieve()
                .body(ConsumerResponse

.class);
        return ResponseEntity.ok(consumerResponse);
    }
}
```

#### Integration Tests
We then set up integration tests to verify the consumer's expectations.

```java
package com.github.innovationforge;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.cloud.contract.stubrunner.spring.AutoConfigureStubRunner;
import org.springframework.cloud.contract.stubrunner.spring.StubRunnerProperties;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;

@ExtendWith(SpringExtension.class)
@SpringBootTest
@AutoConfigureMockMvc
@AutoConfigureStubRunner(
        stubsMode = StubRunnerProperties.StubsMode.CLASSPATH,
        ids = "com.github.innovationforge:provider-application:+:stubs:8081"
)
public class ConsumerControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void givenValidName_whenGetConsumer_thenReturnsMessage() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/consumer")
                .param("name", "john")
                .contentType("application/json"))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.jsonPath("$.message").value("Hello John"));
    }

    @Test
    public void givenNoName_whenGetConsumer_thenReturnsDefaultMessage() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/consumer")
                .contentType("application/json"))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.jsonPath("$.message").value("Hello World!"));
    }
}
```

### Running the Tests

1. **Generate the Stub in the Provider**: Run the following command in the provider application to generate the stubs:
    ```sh
    ./mvnw clean install
    ```

2. **Run the Consumer Tests**: In the consumer application, run the tests to verify the contracts:
    ```sh
    ./mvnw test
    ```

## Conclusion

Consumer-Driven Contract Testing is a powerful approach to ensure seamless integration between microservices. By leveraging Spring Cloud Contract, we can define, share, and verify contracts effectively, leading to more robust and reliable applications. This blog post provided a comprehensive guide to implementing CDCT using Spring Cloud Contract with a practical example. Happy coding!