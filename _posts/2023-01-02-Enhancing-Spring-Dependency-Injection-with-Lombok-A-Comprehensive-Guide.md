---
layout: post
---

In modern software development, maintaining clean and readable code is crucial. One of the key design patterns promoting this practice is Dependency Injection (DI), especially in the Spring Framework. However, even with DI, developers often find themselves writing repetitive boilerplate code. This is where Lombok comes into play, helping to reduce boilerplate and improve code readability. In this blog post, we'll explore how to leverage Lombok for optimizing DI in Spring applications, complete with examples.

**What is Dependency Injection?**

Dependency Injection (DI) is a design pattern that allows a class to receive its dependencies from an external source rather than creating them internally. This promotes loose coupling and enhances testability and maintainability.

**Types of Dependency Injection in Spring**

1. **Constructor Injection**: Dependencies are provided through a class constructor.
2. **Setter Injection**: Dependencies are provided through setter methods.

**Using Lombok to Reduce Boilerplate Code**

Lombok is a Java library that automatically plugs into your editor and build tools, generating boilerplate code such as constructors, getters, setters, and more at compile time. This allows developers to focus on business logic rather than repetitive code.

**Constructor Injection Example**

**Without Lombok**

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.stereotype.Component;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
}

@Component
class Service {
    private final Repository repository;

    public Service(Repository repository) {
        this.repository = repository;
    }

    public void performService() {
        repository.save();
    }
}

@Component
class Repository {
    public void save() {
        System.out.println("Saving data...");
    }
}

public class MainApp {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        Service service = context.getBean(Service.class);
        service.performService();
    }
}
```

**With Lombok**

```java
import lombok.RequiredArgsConstructor;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.stereotype.Component;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
}

@Component
@RequiredArgsConstructor
class Service {
    private final Repository repository;

    public void performService() {
        repository.save();
    }
}

@Component
class Repository {
    public void save() {
        System.out.println("Saving data...");
    }
}

public class MainApp {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        Service service = context.getBean(Service.class);
        service.performService();
    }
}
```

By using `@RequiredArgsConstructor`, Lombok automatically generates a constructor for all final fields, significantly reducing boilerplate code.

**Setter Injection Example**

**Without Lombok**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.stereotype.Component;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
}

@Component
class Service {
    private Repository repository;

    @Autowired
    public void setRepository(Repository repository) {
        this.repository = repository;
    }

    public void performService() {
        repository.save();
    }
}

@Component
class Repository {
    public void save() {
        System.out.println("Saving data...");
    }
}

public class MainApp {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        Service service = context.getBean(Service.class);
        service.performService();
    }
}
```

**With Lombok**

```java
import lombok.Setter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.stereotype.Component;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
}

@Component
class Service {
    @Setter(onMethod = @__(@Autowired))
    private Repository repository;

    public void performService() {
        repository.save();
    }
}

@Component
class Repository {
    public void save() {
        System.out.println("Saving data...");
    }
}

public class MainApp {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        Service service = context.getBean(Service.class);
        service.performService();
    }
}
```

Using `@Setter` with `onMethod` allows Lombok to generate a setter method with the `@Autowired` annotation, reducing the amount of code written manually.

**Which One to Use and Why**

- **Constructor Injection**: Preferred for mandatory dependencies. It ensures that the dependency is injected when the object is created, making the object fully initialized and immutable.
- **Setter Injection**: Suitable for optional dependencies or when there's a need for re-injection. It provides the flexibility to change dependencies after the object is created.

**Conclusion**

Lombok is a powerful tool that can help reduce boilerplate code in Spring applications, especially when working with Dependency Injection. By using annotations like `@RequiredArgsConstructor` and `@Setter`, developers can write cleaner, more maintainable code. Constructor Injection is generally preferred for its advantages in ensuring immutability and completeness of the object, while Setter Injection offers flexibility for optional dependencies.

Implementing Lombok in your Spring projects can lead to more readable and maintainable code, allowing you to focus on the business logic rather than the repetitive tasks of writing constructors and setters.

**Try It Out**

Add Lombok to your Spring project and see how it simplifies your codebase. Happy coding!

