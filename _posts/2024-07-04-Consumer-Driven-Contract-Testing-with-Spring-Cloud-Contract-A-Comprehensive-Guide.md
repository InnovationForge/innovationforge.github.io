---
layout: post
---

Consumer Driven Contract Testing (CDCT) is an effective approach for ensuring that microservices can communicate with each other seamlessly. This testing methodology focuses on defining the interactions between services from the consumer's perspective, thereby ensuring that changes in the provider do not break the consumer's functionality.

## Introduction to Consumer Driven Contract Testing

In microservice architectures, services often interact over APIs. Ensuring these interactions remain consistent and reliable is crucial for system stability. Consumer Driven Contract Testing (CDCT) addresses this need by allowing consumers to define their expectations of the provider's API. These expectations are then used to generate tests for the provider, ensuring compliance with the consumer's requirements.

### Key Benefits of CDCT
1. **Decoupling Services:** CDCT enables independent development and deployment of services by establishing clear contracts.
2. **Early Detection of Issues:** Providers can test against consumer contracts early in the development cycle, preventing integration issues.
3. **Improved Collaboration:** Clear contracts enhance understanding and collaboration between teams responsible for different services.

## Spring Cloud Contract

Spring Cloud Contract is a powerful framework that facilitates CDCT by providing tools to define and verify contracts. It supports the creation of contracts using a Domain Specific Language (DSL) in Groovy, making it easy to write and maintain contracts.

### Key Components
- **Contract Definition Language (DSL):** Allows defining contracts in Groovy.
- **Spring Cloud Contract Verifier:** Generates tests from contracts to verify provider compliance.
- **Stub Runner:** Generates stubs from contracts for consumer testing.

## Example Project Setup

Let's walk through a demo project with two Spring Boot applications: `provider-application` and `consumer-application`. The `provider-application` serves an API, while the `consumer-application` consumes it.

### Provider Application

#### Dependencies and Plugins

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
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

<build>
    <plugins>
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
```

#### Controller and Contract

The `ProviderController` serves the API endpoint:

```java
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

Contracts are defined to specify the expected behavior:

```groovy
// ProviderControllerShouldReturnDefaultGreeting.groovy
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

// ProviderControllerShouldReturnGreetingWithName.groovy
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

#### Dependencies and Plugins

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-contract-wiremock</artifactId>
        <version>4.1.3</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.github.innovationforge</groupId>
        <artifactId>provider-application</artifactId>
        <version>1.0.0</version>
        <classifier>stubs</classifier>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

#### Controller and Integration Tests

The `ConsumerController` makes requests to the provider:

```java
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
                .body(ConsumerResponse.class);
        return ResponseEntity.ok(consumerResponse);
    }
}
```

Integration tests validate the contract:

```java
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

1. **Generate Provider Stubs:** Navigate to the provider application directory and run:
   ```sh
   ./mvnw clean install
   ```
2. **Run Consumer Tests:** In the consumer application directory, execute:
   ```sh
   ./mvnw test
   ```

## Conclusion

Consumer Driven Contract Testing with Spring Cloud Contract ensures robust and reliable API interactions in microservices architectures. By defining clear contracts and verifying compliance, CDCT facilitates smooth integration and enhances collaboration between teams. The demo project showcases the practical implementation of CDCT, providing a solid foundation for integrating this approach into your projects.