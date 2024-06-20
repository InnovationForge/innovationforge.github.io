---
layout: post
---
In today’s interconnected digital landscape, ensuring secure communication between services is paramount. Mutual TLS (mTLS) offers a robust solution by enabling two-way authentication, where both the client and server verify each other's digital certificates. This post explores the design and implementation of a secure mTLS Client and Server application using Spring Boot and associated technologies, with a focus on practical use cases in enterprise application integration.

#### What is Mutual TLS (mTLS)?

Mutual TLS (mTLS) is an extension of the standard TLS (Transport Layer Security) protocol. While TLS ensures secure communication from a server to a client, mTLS adds another layer by requiring the client to also authenticate itself to the server. This bidirectional authentication enhances security, particularly in closed network environments and application integration ecosystems where trust needs to be meticulously managed.

#### Industry Use Cases for mTLS

1. **Internal Microservices Communication**: In a microservices architecture, mTLS can be used to secure communication between internal services, ensuring that only authorized services can communicate with each other.

2. **API Security**: Enterprises expose APIs to partners and customers. By implementing mTLS, businesses can ensure that only authenticated clients can access sensitive APIs.

3. **Application Integration**: For integrating third-party applications or services, mTLS provides a secure method of communication, verifying the identity of both communicating parties.

4. **Compliance and Regulatory Requirements**: Certain industries, such as finance and healthcare, have stringent security requirements. mTLS helps in meeting these regulations by ensuring encrypted and authenticated communication.

#### Project Structure

Our mTLS project is divided into two main applications: `client-app` and `server-app`.

```
springboot-mtls
├── client-app
│   ├── src
│   │   ├── main
│   │   │   ├── java
│   │   │   │   └── com
│   │   │   │       └── example
│   │   │   │           └── client
│   │   │   │               ├── ClientAppApplication.java
│   │   │   │               ├── config
│   │   │   │               │   ├── ConnectionPoolConfig.java
│   │   │   │               │   ├── RestClientConfig.java
│   │   │   │               │   └── SecretsConfig.java
│   │   │   │               ├── controller
│   │   │   │               │   └── HelloController.java
│   │   │   ├── resources
│   │   │   │   ├── application.properties
│   │   │   │   ├── certs
│   │   │   │   │   ├── client.p12
│   │   │   │   │   └── client-truststore.p12
│   │   ├── test
│   │       ├── java
│   │       │   └── com
│   │       │       └── example
│   │       │           └── client
│   │       │               ├── ClientAppApplicationTests.java
│   │       └── resources
│   ├── pom.xml
├── server-app
│   ├── src
│   │   ├── main
│   │   │   ├── java
│   │   │   │   └── com
│   │   │   │       └── example
│   │   │   │           └── server
│   │   │   │               ├── ServerAppApplication.java
│   │   │   │               └── controller
│   │   │   │                   └── HelloController.java
│   │   ├── resources
│   │   │   ├── application.properties
│   │   │   ├── certs
│   │   │   │   ├── server.p12
│   │   │   │   └── server-truststore.p12
│   │   ├── test
│   │       ├── java
│   │       │   └── com
│   │       │       └── example
│   │       │           └── server
│   │       │               ├── ServerAppApplicationTests.java
│   │       └── resources
│   ├── pom.xml
├── pom.xml
```

### Explanation of the Structure:

- **Root Directory (`springboot-mtls`)**:
    - Contains two main subprojects: `client-app` and `server-app`.
    - Each subproject has its own `pom.xml` for Maven configuration.
    - The root `pom.xml` could be a parent POM that includes both subprojects.

- **Client Application (`client-app`)**:
    - **`src/main/java/com/example/client`**:
        - `ClientAppApplication.java`: The main class for the client application.
        - `config`: Contains configuration classes such as `ConnectionPoolConfig`, `RestClientConfig`, and `SecretsConfig`.
        - `controller`: Contains the `HelloController` which handles incoming HTTP requests.
    - **`src/main/resources`**:
        - `application.properties`: Configuration properties for the client application.
        - `certs`: A subdirectory to hold certificate files (`client.p12` and `client-truststore.p12`).
    - **`src/test/java/com/example/client`**:
        - Contains test classes, e.g., `ClientAppApplicationTests.java`.

- **Server Application (`server-app`)**:
    - **`src/main/java/com/example/server`**:
        - `ServerAppApplication.java`: The main class for the server application.
        - `controller`: Contains the `HelloController` which handles incoming HTTP requests.
    - **`src/main/resources`**:
        - `application.properties`: Configuration properties for the server application.
        - `certs`: A subdirectory to hold certificate files (`server.p12` and `server-truststore.p12`).
    - **`src/test/java/com/example/server`**:
        - Contains test classes, e.g., `ServerAppApplicationTests.java`.

This structure enhances readability and maintainability by clearly separating different aspects of the application such as configuration, controllers, and certificates.


#### Client Application

The client application is responsible for sending authenticated requests to the server application.

##### Generating Self-Signed Certificates

To enable mTLS, we need to create self-signed certificates and keystores for both the client and server.

```bash
# Generate client certificate and keystore
keytool -genkeypair -alias client -keyalg RSA -keysize 4096 -validity 365 -dname "CN=Client,OU=Client,O=Examples,L=CA,S=CA,C=US" -keypass changeit -keystore client.p12 -storeType PKCS12 -storepass changeit

# Export the client public key
keytool -exportcert -alias client -file client.cer -keystore client.p12 -storepass changeit

# Import server's public key into client's truststore
keytool -importcert -keystore client-truststore.p12 -alias server-public -file server.cer -storepass changeit -noprompt
```

##### Configuring SSL in `application.properties`

```properties
spring.application.name=client-app
server.port=8444

server.ssl.key-store=classpath:client.p12
server.ssl.key-store-password=changeit
server.ssl.key-store-type=PKCS12

server.ssl.trust-store=classpath:client-truststore.p12
server.ssl.trust-store-password=changeit
```

##### SecretsConfig

```java
@Configuration
@ConfigurationProperties(prefix = "server.ssl")
@Data
public class SecretsConfig {
    private Resource keyStore;
    private String keyStorePassword;
    private Resource trustStore;
    private String trustStorePassword;

    public KeyManager[] getKeyManagers() throws Exception {
        KeyStore ks = KeyStore.getInstance("PKCS12");
        try (InputStream stream = keyStore.getInputStream()) {
            ks.load(stream, keyStorePassword.toCharArray());
        }
        KeyManagerFactory kmf = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
        kmf.init(ks, keyStorePassword.toCharArray());
        return kmf.getKeyManagers();
    }

    public TrustManager[] getTrustManagers() throws Exception {
        KeyStore ts = KeyStore.getInstance("PKCS12");
        try (InputStream stream = trustStore.getInputStream()) {
            ts.load(stream, trustStorePassword.toCharArray());
        }
        TrustManagerFactory tmf = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        tmf.init(ts);
        return tmf.getTrustManagers();
    }
}
```

##### RestClientConfig

```java
@Configuration
@RequiredArgsConstructor
@Slf4j
public class RestClientConfig {
    private final SecretsConfig secretsConfig;
    private final ConnectionPoolConfig connectionPoolConfig;

    @Bean
    public RestClient restClient() throws Exception {
        SSLContext sslContext = SSLContext.getInstance("TLS");
        sslContext.init(secretsConfig.getKeyManagers(), secretsConfig.getTrustManagers(), null);

        SSLConnectionSocketFactory socketFactory = new SSLConnectionSocketFactory(sslContext, new NoopHostnameVerifier());

        Registry<ConnectionSocketFactory> socketFactoryRegistry = RegistryBuilder.<ConnectionSocketFactory>create()
                .register("https", socketFactory)
                .build();

        PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager(socketFactoryRegistry);
        connectionManager.setMaxTotal(connectionPoolConfig.getMaxTotal());
        connectionManager.setDefaultMaxPerRoute(connectionPoolConfig.getMaxPerRoute());

        CloseableHttpClient httpClient = HttpClients.custom()
                .setConnectionManager(connectionManager)
                .build();

        HttpComponentsClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory(httpClient);

        return RestClient.builder().requestFactory(requestFactory).build();
    }
}
```

##### HelloController

```java
@RestController
@Slf4j
public class HelloController {

    private final RestClient restClient;

    public HelloController(RestClient restClient) {
        this.restClient = restClient;
    }

    @GetMapping("/api/hello")
    public String hello() {
        log.info("Message received at Client App");
        return restClient.get()
                .uri("https://localhost:8443/api/hello")
                .retrieve()
                .body(String.class);
    }
}
```

##### ClientAppApplication

```java
@SpringBootApplication
public class ClientAppApplication {
    public static void main(String[] args) {
        SpringApplication.run(ClientAppApplication.class, args);
    }
}
```

#### Server Application

The server application handles incoming requests from the client and validates the client's certificate.

##### Generating Server Certificates

```bash
# Generate server certificate and keystore
keytool -genkeypair -alias server -keyalg RSA -keysize 4096 -validity 365 -dname "CN=Server,OU=Server,O=InnovationForge,L=Arnhem,S=GLD,C=NL" -keypass changeit -keystore server.p12 -storeType PKCS12 -storepass changeit

# Export the server public key
keytool -exportcert -alias server -file server.cer -keystore server.p12 -storepass changeit

# Import client's public key into server's truststore
keytool -importcert -keystore server-truststore.p12 -alias client-public -file client.cer -storepass changeit -noprompt
```

##### Configuring SSL in `application.properties`

```properties
spring.application.name=server-app
server.port=8443
server.ssl.key-store-type=PKCS12
server.ssl.key-store=classpath:server.p12
server.ssl.key-store-password=changeit

server.ssl.client-auth=need
server.ssl.trust-store=classpath:server-truststore.p12
server.ssl.trust-store-password=changeit
```

##### HelloController

```java
@RestController
@Slf4j
public class HelloController {

    @GetMapping("/api/hello")
    public String hello() {
        log.info("Message received at Server App");
        return "Hello World !! from Server App";
    }
}
```

##### ServerAppApplication

```java
@SpringBootApplication
public class ServerAppApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServerAppApplication.class, args);
    }
}
```

#### Conclusion

Implementing mTLS with Spring Boot is a powerful way to enhance security in enterprise applications. By ensuring that both the client and server authenticate each other, mTLS provides a robust mechanism for secure communication, particularly useful in internal microservices communication, API security, and application integration. This approach not only strengthens the security posture but also helps in meeting regulatory compliance requirements.