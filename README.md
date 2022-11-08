# Spring Context and Beans

## Learning Goals

- Define Spring context.
- Add beans to the application context.
- Discuss the differences between a bean and a component.

## Introduction

Now that we have a very simple working MVC, let's back up a bit and look
at some more fundamentals of the Spring framework.

In this lesson, we'll talk about what an application context is and how we can
add objects to the Spring application context.

## What is the Spring Context?

A fundamental element to the Spring framework is the **application context**. The
context is a memory space in our application in which we can add objects to that
need to be managed by the framework. Spring is not aware of objects unless it's
explicitly told to keep track of them. We can also define relationships between
objects in the context. This is an important concept to using Spring, because
otherwise, we cannot turn our Java objects into something that Spring recognizes.

So, how do we tell Spring to add our Java objects into the Spring application
context?

This is where we will introduce the concept of a bean and a component.

## Add Bean using `@Bean` Annotation

**Beans** are object instances that are managed by Spring. We can add beans into
the context by using the `@Bean` annotation or stereotype annotations provided
by Spring.

The `@Bean` annotation is applied to methods. We will create a `Dog` class and
add an instance of it to the Spring context.

![Spring context diagram with a single dog bean](https://curriculum-content.s3.amazonaws.com/java-spring-1/spring-context-dog.png)

Let’s create a `Dog` class before we configure Spring and add the bean to the
application context.

In a new Spring Boot project, we'll create a new package called `domain`. Then we
will create the `Dog` class in the `domain` package with the following code:

```java
package com.example.demo.domain;

public class Dog {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

We will need to follow these steps for adding a bean to the context:

1. Create a configuration class.
2. Add a method with the `@Bean` annotation to the configuration class, which
   should return the object instance that needs to be added to the context.
3. Initialize the application context with the configuration class.

### Create Configuration Class

A **configuration class** is defined in Spring using the `@Configuration`
annotation. These classes are used to define Spring related configurations and
indicate that a class declares one or more `@Bean` methods.

To create a configuration class, we'll create a new package called `config`. Then
we will create a `MyConfig` class in the `config` package with the following code:

```java
package com.example.demo.config;

import org.springframework.context.annotation.Configuration;

@Configuration
public class MyConfig {

}
```

### Add Method for Bean Creation

When a method in a configuration class returns an object instance and is
annotated with `@Bean`, Spring calls the method and manages it in its context.
The methods used for adding beans are named using nouns instead of verbs since
these methods are used for representing the object instances Spring manages.

Add a `dog()` method in the `MyConfig` class:

```java
package com.example.demo.config;

import com.example.demo.domain.Dog;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyConfig {
    @Bean
    public Dog dog() {
        Dog d = new Dog();
        d.setName("Luna");
        return d;
    }
}
```

The `@Bean` annotation tells Spring to call this method when it initializes the
context and store the returned value in its context.

### Initialize Spring Context

We will be using the `AnnotationConfigApplicationContext` class to create the
Spring context instance since we are using annotations for configurations.

In the default class, add the following code:

```java
package com.example.demo;

import com.example.demo.config.MyConfig;
import com.example.demo.domain.Dog;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
        
        // create Spring context instance
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
        
        // access the bean
        Dog luna = context.getBean(Dog.class);
        System.out.println(luna);
    }
}
```

We instantiate an `AnnotationConfigApplicationContext` instance with the
`MyConfig` class. This causes Spring to initialize the `Dog` instance and add it
to its context, which makes it a bean (any object managed by Spring).

![Spring context diagram with a single dog bean](https://curriculum-content.s3.amazonaws.com/java-spring-1/spring-context-dog.png)

The `getBean` method gets an instance of the `Dog` class from the Spring
context. Since there is a single `Dog` instance in the context we only need to
specify the class of the instance we want to access.

## Add Beans Using Stereotype Annotation (@Component)

Spring provides class-level annotations in the `org.springframework.stereotype`
package for configuring applications. These are called **components**. We will be
using the `@Component` annotation to add a bean to the Spring context. We need to
follow these steps to add the beans using the `@Component` annotation:

1. Annotate the class for which the instance needs to be added to the context.
   In this case, we will annotate the `Dog` class definition.

```java
package com.example.demo.domain;

import org.springframework.stereotype.Component;

@Component
public class Dog {
   private String name;

   public String getName() {
      return name;
   }

   public void setName(String name) {
      this.name = name;
   }
}
```

2. To look for component classes, we can use the `@ComponentScan` annotation over
   the configuration class to tell Spring where to look for classes annotated with
   stereotype annotations. We'll pass in the `domain` package to tell Spring to
   scan that as the base package.

```java
   package config;

   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.context.annotation.Configuration;

   @Configuration
   @ComponentScan(basePackages = "com.example.demo.domain")
   public class MyConfig {
       
   }
```
Spring will create the instance for us which means we can’t set the `name`
property on the `Dog` instance during instantiation. Instead, we will get the
`Dog` instance after Spring creates it and set the `name` property. Update the
`Main` class code:

```java
package com.example.demo;

import com.example.demo.config.MyConfig;
import com.example.demo.domain.Dog;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

@SpringBootApplication
public class DemoApplication {
   public static void main(String[] args) {
      SpringApplication.run(DemoApplication.class, args);

      // create Spring context instance
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);

      // access the bean
      Dog luna = context.getBean(Dog.class);
      luna.setName("Luna");
      System.out.println(luna);
   }
}
```

Running the `main` method will print out the `Dog` instance in the Spring
context. The stereotype annotations are primarily used over the `@Bean`
annotations since it is more simple.

So far, we have been using the `@Service` and `@Controller` annotations, which
are part of the `org.springframework.stereotype` package, and are applied to
classes. Both of these annotations serve as a specialization of `@Component`,
allowing for implementation classes to be autodetected through classpath scanning.
This is how we were able to add them as beans to the application context in the
previous lessons!

## `@Bean` vs `@Component`

Now that we have used both of these annotations for adding beans let’s look at
their differences:

| `@Bean`                                                                                                            | `@Component`                                                                |
|--------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| Method-level annotation                                                                                            | Class-level annotation                                                      |
 | Full control over the instance that's added to the context since we have access to the instance in the method body | Only gets access to the instance once Spring creates it                     |
 | Annotate multiple methods with the `@Bean` annotation and use them to create multiple instances of the same class  | The `@Component` annotation only allows one instance of the annotated class |
 | Used to add any objects to the Spring context, including instances of classes that we did not create               | Used for classes defined in the application only                            |

## Conclusion

We have learned about the Spring context which is a core part of the Spring
framework. We also looked at how to use both the `@Bean` and `@Component`
annotations to add beans to the context.

## References

- [Configuration Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)
- [Bean Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html)
- [Component Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html)
- [ComponentScan Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.html)
- [Service Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html)
- [Controller Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Controller.html)
