---
layout: post
---

In today's software development landscape, open source libraries are essential for speeding up development. However, they also introduce potential security vulnerabilities. Nexus IQ is a powerful tool that helps you manage and mitigate risks associated with using open source libraries. This blog post will demonstrate how to integrate Nexus IQ with a simple Spring Boot application using Maven profiles, enabling you to perform security scans both locally and in a CI/CD pipeline.

### Prerequisites
Before we start, ensure you have the following:
- A basic understanding of Maven and Spring Boot.
- Nexus IQ server up and running (local or hosted).
- Maven installed.
- A simple Spring Boot application.

### Setting Up Nexus IQ
1. **Download and Install Nexus IQ:**
   - Follow the instructions on the [Nexus IQ download page](https://www.sonatype.com/products/nexus-iq-server) to install and start Nexus IQ.

2. **Create an Application in Nexus IQ:**
   - Log in to your Nexus IQ instance.
   - Create a new application and generate the necessary credentials.

### Configuring Maven for Nexus IQ
1. **Add Nexus IQ Plugin to `pom.xml`:**
   ```xml
   <project>
       ...
       <build>
           <plugins>
               ...
               <plugin>
                   <groupId>org.sonatype.plugins</groupId>
                   <artifactId>nexus-clm-maven-plugin</artifactId>
                   <version>2.10.0-01</version>
               </plugin>
               ...
           </plugins>
       </build>
       ...
   </project>
   ```

2. **Define Maven Profiles:**
   We'll create two Maven profiles—one for local scans and one for CI/CD.

   ```xml
   <project>
       ...
       <profiles>
           <profile>
               <id>nexus-iq-local</id>
               <activation>
                   <activeByDefault>false</activeByDefault>
               </activation>
               <properties>
                   <nexus.iq.server.url>http://localhost:8070</nexus.iq.server.url>
                   <nexus.iq.application.publicId>your_application_public_id</nexus.iq.application.publicId>
                   <nexus.iq.username>your_local_nexus_iq_username</nexus.iq.username>
                   <nexus.iq.password>your_local_nexus_iq_password</nexus.iq.password>
               </properties>
           </profile>

           <profile>
               <id>nexus-iq-ci</id>
               <activation>
                   <activeByDefault>false</activeByDefault>
               </activation>
               <properties>
                   <nexus.iq.server.url>http://your-ci-nexus-iq-url</nexus.iq.server.url>
                   <nexus.iq.application.publicId>your_application_public_id</nexus.iq.application.publicId>
                   <nexus.iq.username>your_ci_nexus_iq_username</nexus.iq.username>
                   <nexus.iq.password>your_ci_nexus_iq_password</nexus.iq.password>
               </properties>
           </profile>
       </profiles>
       ...
   </project>
   ```

### Externalizing Credentials
Instead of hardcoding credentials in `pom.xml`, use environment variables or a properties file. Here's how:

#### Using Environment Variables:
1. **Set Environment Variables:**
   ```sh
   export NEXUS_IQ_SERVER_URL=http://your-ci-nexus-iq-url
   export NEXUS_IQ_APPLICATION_PUBLIC_ID=your_application_public_id
   export NEXUS_IQ_USERNAME=your_nexus_iq_username
   export NEXUS_IQ_PASSWORD=your_nexus_iq_password
   ```

2. **Reference Environment Variables in `pom.xml`:**
   ```xml
   <project>
       ...
       <profiles>
           <profile>
               <id>nexus-iq-local</id>
               <activation>
                   <activeByDefault>false</activeByDefault>
               </activation>
               <properties>
                   <nexus.iq.server.url>${env.NEXUS_IQ_SERVER_URL}</nexus.iq.server.url>
                   <nexus.iq.application.publicId>${env.NEXUS_IQ_APPLICATION_PUBLIC_ID}</nexus.iq.application.publicId>
                   <nexus.iq.username>${env.NEXUS_IQ_USERNAME}</nexus.iq.username>
                   <nexus.iq.password>${env.NEXUS_IQ_PASSWORD}</nexus.iq.password>
               </properties>
           </profile>

           <profile>
               <id>nexus-iq-ci</id>
               <activation>
                   <activeByDefault>false</activeByDefault>
               </activation>
               <properties>
                   <nexus.iq.server.url>${env.NEXUS_IQ_SERVER_URL}</nexus.iq.server.url>
                   <nexus.iq.application.publicId>${env.NEXUS_IQ_APPLICATION_PUBLIC_ID}</nexus.iq.application.publicId>
                   <nexus.iq.username>${env.NEXUS_IQ_USERNAME}</nexus.iq.username>
                   <nexus.iq.password>${env.NEXUS_IQ_PASSWORD}</nexus.iq.password>
               </properties>
           </profile>
       </profiles>
       ...
   </project>
   ```

### Running Nexus IQ Analysis Locally
To run the Nexus IQ analysis locally, use the following command:
```sh
mvn clean verify org.sonatype.plugins:nexus-clm-maven-plugin:2.10.0-01:scan -Pnexus-iq-local
```

### Integrating Nexus IQ in CI/CD Pipeline
For CI/CD, you'll configure your pipeline to trigger the Nexus IQ scan using the `nexus-iq-ci` profile. Here's an example using GitHub Actions:

1. **GitHub Actions Workflow (`.github/workflows/ci.yml`):**
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
         run: mvn clean verify

       - name: Nexus IQ Scan
         env:
           NEXUS_IQ_SERVER_URL: ${{ secrets.NEXUS_IQ_SERVER_URL }}
           NEXUS_IQ_APPLICATION_PUBLIC_ID: ${{ secrets.NEXUS_IQ_APPLICATION_PUBLIC_ID }}
           NEXUS_IQ_USERNAME: ${{ secrets.NEXUS_IQ_USERNAME }}
           NEXUS_IQ_PASSWORD: ${{ secrets.NEXUS_IQ_PASSWORD }}
         run: mvn org.sonatype.plugins:nexus-clm-maven-plugin:2.10.0-01:scan -Pnexus-iq-ci
   ```

### Conclusion
Integrating Nexus IQ with your Spring Boot application using Maven profiles allows you to proactively manage and mitigate security risks associated with open source libraries. By following the steps outlined in this post, you can set up a robust system for continuous security monitoring both locally and in your CI/CD pipeline.

Happy coding!

---

### Additional Tips
- Regularly update your Nexus IQ server and plugins to benefit from the latest features and security updates.
- Monitor Nexus IQ reports and promptly address any identified vulnerabilities.
- Explore Nexus IQ's advanced features such as policy management and detailed reports for comprehensive security management.

By following this guide, you'll be well on your way to integrating Nexus IQ effectively into your Java development workflow.
