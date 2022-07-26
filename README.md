# Spring Context and Beans

## Learning Goals

- Define Spring context and beans.
- Create Spring context.
- Add Beans to the Spring context.

## Introduction

The application context in Spring is a memory space where all the objects that
needs to be managed by the framework is added. Spring is not aware of objects
unless it’s explicitly told to keep track of them. We can also define
relationships between objects in the context.

Beans are object instances that are managed by Spring. We can add beans using in
the context by using the `@Bean` annotation or stereotype annotations provided
by Spring.

## Create Maven Project

We will be using Maven to add the Spring context dependency to our project.

1. Open IntelliJ and create a new project. Name the project
   `spring-context-example`.

   ![IntelliJ create project screen](https://curriculum-content.s3.amazonaws.com/java-spring-1/create-project-spring-context.png)

2. Add the `spring-context` dependency in the `pom.xml` file and reload the
   project.

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-context</artifactId>
           <version>5.3.15</version>
       </dependency>
   </dependencies>
   ```

3. Create a `main` and `config` package in `src/main/java`. Your directory
   structure should look like this:

   ```
   ├── pom.xml
   └── src
       ├── main
       │   ├── java
       │   │   ├── config
       │   │   └── main
       │   └── resources
       └── test
           └── java
   ```

## Add Bean using `@Bean` Annotation

The `@Bean` annotation is applied to methods. We will create a `Dog` class and
add an instance of it to the Spring context.

![Spring context diagram with a single dog bean](https://curriculum-content.s3.amazonaws.com/java-spring-1/spring-context-dog.png)

Let’s create the `Dog` class before we configuring Spring and adding the bean.
Create the `Dog` class in the `main` and add the following code:

```java
package main;

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
2. Add a method with the `@Bean` annotation to the configuration class which
   should return the object instance that needs to be added to the context.
3. Initialize Spring context with the configuration class.

### Create Configuration Class

A configuration class is defined in Spring using the `@Configuration`
annotation. These classes are used to define Spring related configurations.

Create a `MyConfig` class in the `config` package and the following code:

```java
package config;

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
package config;

import main.Dog;
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

Create a `Main` class in the `main` package and add the following code:

```java
package main;

import config.MyConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
    public static void main(String[] args) {
				// create Spring context instance
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
				// access the bean
        Dog luna = context.getBean(Dog.class);
        System.out.println(luna);
    }
}
```

We call the instantiate `AnnotationConfigApplicationContext` class with the
`MyConfig` class. This causes Spring to initialize the `Dog` instance and add it
to its context which makes it a bean (any object managed by Spring).

![Spring context diagram with a single dog bean](https://curriculum-content.s3.amazonaws.com/java-spring-1/spring-context-dog.png)

The `getBean` method gets an instance of the `Dog` class from the Spring
context. Since there is a single `Dog` instance in the context we only need to
specify the class of the instance we want to access.

## Add Beans Using Stereotype Annotation (@Component)

Spring provides class-level annotations in the `org.springframework.stereotype`
package for configuring applications. We will be using `@Component` annotation
to add a bean to the Spring context. We need to follow these steps to add the
beans using the `@Component` annotation:

1. Annotate the class for which the instance needs to be added to the context.
   In this case, we will annotate the `Dog` class.

   ```java
   package main;

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

2. Use the `@ComponentScan` annotation over the configuration class and tell it
   where to look for classes annotated with stereotype annotations.

   ```java
   package config;

   import org.springframework.context.annotation.ComponentScan;
   import org.springframework.context.annotation.Configuration;

   @Configuration
   @ComponentScan(basePackages = "main")
   public class MyConfig {

   }
   ```

Spring will create the instance for us which means we can’t set the `name`
property on the `Dog` instance during instantiation. Instead, we will get the
`Dog` instance after Spring creates it and set the `name` property. Update the
`Main` class code:

```java
package main;

import config.MyConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
    public static void main(String[] args) {
        // create context
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
        // access bean
        Dog luna = context.getBean(Dog.class);
        luna.setName("Luna");
        System.out.println(luna);
    }
}
```

Running the `main` method will print out the `Dog` instance in the Spring
context. The stereotype annotations are primarily used over the `@Bean`
annotations since it is more simple.

As mentioned earlier, the `org.springframework.stereotype` package has other
annotations:

- `@Service`: applied to classes used for business logic.
- `@Controller`: applied to classes used in a web environment.
- `@Repository`: applied to classes used for interacting with external data
  storage.

## `@Bean` vs `@Component`

Now that we have used both of these annotations for adding beans let’s look at
their differences:

1. `@Bean` is a method-level annotation and `@Component` is a class-level
   annotation.
2. `@Bean` gives us full control over the instance that’s added to the context
   since we have access to the instance in the method body. For the `@Component`
   annotation, we only get access to the instance once Spring creates it.
3. We can annotate multiple methods with the `@Bean` annotation and use them to
   create multiple instances of the same class. The `@Component` annotation only
   allows one instance of the annotated class.
4. `@Bean` can be used to add any objects to the Spring context including
   instances of classes that we did not create. For example, we can use the
   following method to add a `String` to the context:

   ```java
   @Bean
       public String cat() {
           return "Cat";
       }
   ```

   `@Component` annotation can only be used for classes defined in the
   application.

## Conclusion

We have learned about the Spring context which is a core part of the Spring
framework. We also looked at how to use both the `@Bean` and `@Component`
annotations to add beans to the context.
