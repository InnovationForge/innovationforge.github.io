---
layout: post
---

### Building a PDF Generation Service with Spring Boot

In this blog post, we'll walk you through the design and implementation of a PDF generation service using Spring Boot. This service takes a template and data to create the requested PDF, ideal for generating documents like invoices, reports, or any other structured document. We'll cover the project structure, key technologies used, and the code implementation.

#### Project Overview

The primary goal of this project is to create a RESTful service that generates PDF files based on HTML templates and input data. We'll use the following technologies:
- **Spring Boot** for building the REST API.
- **Thymeleaf** for template rendering.
- **Open HTML to PDF** for converting HTML content to PDF format.

#### Project Structure

Here's a quick overview of the project's structure:

```
src
└── main
    ├── java
    │   └── com.github.innovationforge
    │       ├── PdfController.java
    │       ├── PdfService.java
    │       └── PdfServiceApplication.java
    └── resources
        ├── templates
        │   ├── invoice.html
        │   └── invoice1.html
        └── application.properties
```

#### Dependencies

The `pom.xml` file includes the necessary dependencies for Spring Boot, Thymeleaf, and Open HTML to PDF:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>com.openhtmltopdf</groupId>
        <artifactId>openhtmltopdf-pdfbox</artifactId>
        <version>1.0.10</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

#### Configuration

The `application.properties` file configures the application:

```properties
spring.application.name=pdf-service
server.port=8080
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
spring.thymeleaf.mode=HTML5
```

#### HTML Template

We use Thymeleaf templates to define the structure of the PDFs. Here's an example template (`invoice.html`):

```html
<!DOCTYPE html>
<html lang="NL">
<head>
    <meta charset="UTF-8"/>
    <title>Invoice</title>
    <style>
        body { font-family: Arial, sans-serif; }
        .invoice-box { max-width: 800px; margin: auto; padding: 30px; border: 1px solid #eee; box-shadow: 0 0 10px rgba(0, 0, 0, 0.15); }
        .invoice-box table { width: 100%; line-height: inherit; text-align: left; }
        .invoice-box table td { padding: 5px; vertical-align: top; }
        .invoice-box table tr td:nth-child(2) { text-align: right; }
        .invoice-box table tr.top table td { padding-bottom: 20px; }
        .invoice-box table tr.information table td { padding-bottom: 40px; }
        .invoice-box table tr.heading td { background: #eee; border-bottom: 1px solid #ddd; font-weight: bold; }
        .invoice-box table tr.details td { padding-bottom: 20px; }
        .invoice-box table tr.item td { border-bottom: 1px solid #eee; }
        .invoice-box table tr.item.last td { border-bottom: none; }
        .invoice-box table tr.total td:nth-child(2) { border-top: 2px solid #eee; font-weight: bold; }
    </style>
</head>
<body>
<div class="invoice-box">
    <table>
        <tr class="top">
            <td colspan="2">
                <table>
                    <tr>
                        <td class="title">
                            <h2>Invoice</h2>
                        </td>
                        <td>
                            Invoice #: <span th:text="${invoiceNumber}"></span><br/>
                            Created: <span th:text="${createdDate}"></span><br/>
                            Due: <span th:text="${dueDate}"></span>
                        </td>
                    </tr>
                </table>
            </td>
        </tr>
        <tr class="information">
            <td colspan="2">
                <table>
                    <tr>
                        <td>
                            <span th:text="${companyName}"></span><br/>
                            <span th:text="${companyAddress}"></span><br/>
                            <span th:text="${companyEmail}"></span>
                        </td>
                        <td>
                            <span th:text="${clientName}"></span><br/>
                            <span th:text="${clientAddress}"></span><br/>
                            <span th:text="${clientEmail}"></span>
                        </td>
                    </tr>
                </table>
            </td>
        </tr>
        <tr class="heading">
            <td>Item</td>
            <td>Price</td>
        </tr>
        <tr class="item" th:each="item : ${items}">
            <td th:text="${item.description}">Item 1</td>
            <td th:text="${item.price}">$0.00</td>
        </tr>
        <tr class="total">
            <td></td>
            <td>Total: <span th:text="${total}"></span></td>
        </tr>
    </table>
</div>
</body>
</html>
```

#### Controller

The `PdfController` handles incoming requests to generate PDFs:

```java
package com.github.innovationforge;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.io.IOException;
import java.util.Map;

@RestController
@RequestMapping("/api/pdf")
public class PdfController {

    @Autowired
    private PdfService pdfService;

    @PostMapping(value = "/generate", produces = "application/pdf")
    public ResponseEntity<byte[]> generatePdf(@RequestParam("template") String templateName,
                                              @RequestBody Map<String, Object> data) {
        try {
            byte[] pdfContent = pdfService.generatePdf(templateName, data);
            HttpHeaders headers = new HttpHeaders();
            headers.add("Content-Disposition", "inline; filename=invoice.pdf");

            return new ResponseEntity<>(pdfContent, headers, HttpStatus.OK);
        } catch (IOException e) {
            return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}
```

#### Service

The `PdfService` contains the logic to render the HTML template and convert it to a PDF:

```java
package com.github.innovationforge;

import com.openhtmltopdf.pdfboxout.PdfRendererBuilder;
import org.springframework.stereotype.Service;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.util.Map;

@Service
public class PdfService {

    private final TemplateEngine templateEngine;

    public PdfService(TemplateEngine templateEngine) {
        this.templateEngine = templateEngine;
    }

    public byte[] generatePdf(String templateName, Map<String, Object> data) throws IOException {
        // Merge data into the HTML template
        Context context = new Context();
        context.setVariables(data);
        String htmlContent = templateEngine.process(templateName, context);

        // Convert HTML to PDF
        try (ByteArrayOutputStream os = new ByteArrayOutputStream()) {
            PdfRendererBuilder builder = new PdfRendererBuilder();
            builder.withHtmlContent(htmlContent, null);
            builder.toStream(os);
            builder.run();
            return os.toByteArray();
        } catch (Exception e) {
            throw new IOException("Error generating PDF", e);
        }
    }
}
```

#### Main Application

The `PdfServiceApplication` class is the entry point for the Spring Boot application:

```java
package com.github.innovationforge;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class PdfServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(PdfServiceApplication.class, args);
    }

}
```

### Conclusion

In this blog post, we've walked through the design and implementation of a PDF generation service using Spring Boot. By leveraging Thymeleaf for template rendering and Open HTML to PDF for PDF conversion, we've created a flexible and robust service capable of generating PDFs based on dynamic data. This approach can be easily extended to support various document types and templates, making it a valuable addition to any enterprise application.