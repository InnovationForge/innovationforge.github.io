---
layout: post
---

In a large organization where multiple teams are developing microservices using Spring Boot, maintaining seamless interactions between services can be challenging. Data synchronization can be expensive, and ensuring that changes in one service do not break others is critical. Implementing Consumer Driven Contract Testing (CDCT) can address these challenges by providing a structured approach to API interaction testing, increasing confidence in production deployments, and offering flexibility for developers in unit and integration testing.

## Strategy for Implementing CDCT

### 1. Establish Clear Ownership and Responsibilities

- **Service Owners:** Assign clear ownership of each microservice to specific teams. These teams will be responsible for creating and maintaining the contracts for their services.
- **Contract Managers:** Designate individuals or a team to oversee the contract repository and ensure consistency and compliance with organizational standards.

### 2. Define and Document Contract Guidelines

- **Contract Creation:** Establish guidelines for writing contracts using Spring Cloud Contract DSL in Groovy. This includes naming conventions, directory structure, and best practices.
- **Contract Review:** Implement a review process to ensure that contracts are accurate and comprehensive. This can involve code reviews and automated checks.

### 3. Centralize Contract Repository

- **Version Control:** Use a version control system (e.g., Git) to store contracts. This allows for tracking changes, versioning, and collaboration.
- **Accessibility:** Ensure the contract repository is accessible to all teams. This can be done via a central Git repository, where each service has a dedicated directory for its contracts.

### 4. Automate Contract Testing

- **CI/CD Integration:** Integrate contract testing into your CI/CD pipeline. Ensure that contracts are verified against the provider during the build process.
- **Automated Stubs Generation:** Use Spring Cloud Contract to automatically generate stubs from contracts, enabling consumers to test against these stubs.

### 5. Provide Training and Support

- **Training Sessions:** Conduct training sessions for teams to understand CDCT concepts, how to write contracts, and how to integrate contract testing into their workflows.
- **Documentation:** Maintain comprehensive documentation on the CDCT process, tools, and best practices.
- **Support Channels:** Set up support channels (e.g., Slack, email) for teams to get help and share knowledge.


## Implementing Centralized Contract Repository for Consumer Driven Contract Testing (CDCT)

In a large organization, managing contract testing across multiple teams requires a well-defined strategy. A centralized contract repository helps maintain consistency and ease of access to contract definitions. Below is a comprehensive guide on how to implement this from both the provider and consumer perspectives, including CI/CD integration.

### Centralized Contract Repository Solution

1. **Setup a Version Control Repository (e.g., GitHub, GitLab):**
    - Create a dedicated repository for contracts, e.g., `contracts-repo`.
    - Organize directories by service names, each containing their respective contracts.

   ```
   contracts-repo/
   ├── provider-service-1/
   │   ├── contract1.groovy
   │   ├── contract2.groovy
   ├── provider-service-2/
   │   ├── contract1.groovy
   │   ├── contract2.groovy
   ```

### Provider Implementation

#### 1. Adding Dependencies and Plugins

Update the provider application's `pom.xml` to include dependencies for Spring Cloud Contract Verifier and the plugin configuration.

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
                    com.example.BaseTestClass
                </baseClassForTests>
                <contractsRepository>
                    https://<your-git-repo-url>/contracts-repo/provider-service-1
                </contractsRepository>
                <contractsPath>/</contractsPath>
            </configuration>
        </plugin>
    </plugins>
</build>
```

#### 2. Fetching Contracts in CI/CD Pipeline

- **CI/CD Pipeline Configuration:**
  Ensure your CI/CD pipeline fetches the latest contracts from the centralized repository.

  **Example with GitHub Actions:**

  ```yaml
  name: Build and Verify Contracts

  on: [push, pull_request]

  jobs:
    build:
      runs-on: ubuntu-latest

      steps:
        - name: Checkout Repository
          uses: actions/checkout@v2

        - name: Checkout Contracts Repository
          uses: actions/checkout@v2
          with:
            repository: <your-org>/contracts-repo
            path: contracts-repo
            token: ${{ secrets.GITHUB_TOKEN }}

        - name: Set up JDK 17
          uses: actions/setup-java@v2
          with:
            java-version: 17

        - name: Build and Verify Contracts
          run: ./mvnw clean verify
  ```

#### 3. Generate Stubs

Running the Maven build will generate the stubs and place them in the target directory.

```sh
./mvnw clean install
```

### Consumer Implementation

#### 1. Adding Dependencies

Update the consumer application's `pom.xml` to include dependencies for WireMock and Spring Cloud Contract Stub Runner.

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
        <groupId>com.example</groupId> <!-- Replace with the groupId of your provider application -->
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

#### 2. Writing Integration Tests

Use the generated stubs in your integration tests to simulate interactions with the provider.

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
@AutoConfigureMockMvc
@AutoConfigureStubRunner(
        stubsMode = StubRunnerProperties.StubsMode.CLASSPATH,
        ids = "com.example:provider-application:+:stubs:8081"
)
public class ConsumerControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void givenValidName_whenGetConsumer_thenReturnsMessage() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/consumer")
                .param("name", "John")
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

#### 3. Running Consumer Tests

Run the tests to ensure the consumer application interacts correctly with the provider stubs.

```sh
./mvnw test
```

### CI/CD Integration for Consumers

- **CI/CD Pipeline Configuration:**
  Ensure your CI/CD pipeline fetches the latest stubs and runs the integration tests.

  **Example with GitHub Actions:**

  ```yaml
  name: Test Consumer Application

  on: [push, pull_request]

  jobs:
    test:
      runs-on: ubuntu-latest

      steps:
        - name: Checkout Repository
          uses: actions/checkout@v2

        - name: Set up JDK 17
          uses: actions/setup-java@v2
          with:
            java-version: 17

        - name: Build and Test
          run: ./mvnw clean test
  ```

### Summary

By implementing a centralized contract repository and integrating CDCT into the CI/CD pipeline, you can achieve consistent and reliable interactions between microservices developed by different teams. This approach ensures that changes in one service are promptly reflected and verified against the contracts, thereby reducing the risk of integration issues and increasing confidence in production deployments.

## Solution Implementation

### Provider Application Setup

#### Dependencies and Plugins

Ensure the provider application includes the necessary dependencies and plugins for Spring Cloud Contract.

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
                    com.example.BaseTestClass
                </baseClassForTests>
            </configuration>
        </plugin>
    </plugins>
</build>
```

#### Controller and Contract Example

The provider application exposes an API, and we define contracts for expected behaviors.

```java
@RestController
@RequestMapping("/provider")
public class ProviderController {

    @GetMapping(produces = "application/json")
    public ResponseEntity<ProviderResponse> getProviderMessage(@RequestParam(value = "name", required = false) String name){
        return ResponseEntity.ok(ProviderResponse.builder()
                .message(name != null ? "Hello " + name : "Hello World!")
                .timestamp(LocalDateTime.now())
                .build());
    }
}
```

Contracts:

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
    description "should return a greeting message with the name if provided"

    request {
        method GET()
        url("/provider") {
            queryParameters {
                parameter("name", "John")
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

### Consumer Application Setup

#### Dependencies

Ensure the consumer application includes the necessary dependencies for contract testing.

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
        <groupId>com.example</groupId> <!-- Replace with the groupId of your provider application -->
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

The consumer application makes requests to the provider and verifies responses against the generated stubs.

```java
@RestController
@RequestMapping("/consumer")
@RequiredArgsConstructor
public class ConsumerController {

    private final RestTemplate restTemplate;

    @GetMapping
    public ResponseEntity<ConsumerResponse> getConsumer(@RequestParam(value = "name", required = false) String name){
        ConsumerResponse consumerResponse = restTemplate.getForObject(
                name != null ? "http://localhost:8081/provider?name=" + name : "http://localhost:8081/provider",
                ConsumerResponse.class);
        return ResponseEntity.ok(consumerResponse);
    }
}
```

Integration tests:

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
@AutoConfigureMockMvc
@AutoConfigureStubRunner(
        stubsMode = StubRunnerProperties.StubsMode.CLASSPATH,
        ids = "com.example:provider-application:+:stubs:8081"
)
public class ConsumerControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void givenValidName_whenGetConsumer_thenReturnsMessage() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/consumer")
                .param("name", "John")
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

1. **Generate Provider Stubs:**
   ```sh
   cd provider-application
   ./mvnw clean install
   ```
2. **Run Consumer Tests:**
   ```sh
   cd consumer-application
   ./mvnw test
   ```

## Conclusion

Implementing Consumer Driven Contract Testing across multiple teams in a large organization ensures that microservices interact reliably and consistently. By defining clear contracts, automating testing processes, and fostering a culture of collaboration and ownership, organizations can increase confidence in their production deployments and provide flexibility for developers to perform thorough unit and integration testing. The strategy outlined above provides a structured approach to achieving these goals, leveraging the power of Spring Cloud Contract and best practices in software development.