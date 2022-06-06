---
title: "Java Maven Gae Spring and Vadin"
date: 2013-11-22T20:04:29+01:00
draft: false
author: "Bart Prokop"
description: "How to use Maven to deploy Java application on Google App Engine wiritten in Spring with Vadin"
tags: ["Java", "AspectJ", "Spring", "Vintage"]
---

In this article I will cover how to develop single-paged web applications using the techniques mentioned in title. Source code is available [there](https://github.com/bartprokop/bart-gae-poc/tree/version1). Working application is [available](http://1.bart-gae-poc.appspot.com) too.

# Problem: putting all pieces together
[Vaadin](https://vaadin.com/) is great tool for developing Web based business application. It uses GWT for rendering the server side kept UI structures. So basically the whole application - both UI and backend runs on the server, while browser merely renders it. One of the properties of Vaadin application is that UI state is always serialized and kept inside http session. This feature has it's cons and pros. If you correctly persist session, you can run application on a cluster and the app will survive the restart or outage of single node. But everything what is references by any GUI elements must implement java.io.Serializable. As such is necessary to ensure that by accident we will not serialize whole object graph to http session by keeping unnecessary references in UI object fields. The transient keyword is your friend here (but it comes with it's intrinsic limitation on deserialize).

[Spring](http://spring.io/) - It is completelly pointless to use spring-mvc or Spring Web Context or Spring request scopes with Vaadin. In fact it would duplicate Vaadin functionality (so callsed Application Session) that is based on deserialization of http request to current thread variables. It is advisable to use application (JVM instance) scoped Application Context. Also note not to accidentally try to serialize whole bean factory to http session with Vaadin GUI component (this will raise non serializable class exception).

[Google App Engine](https://developers.google.com/appengine/ - it seems it finally matured as Java PaaS solution. There is finally native, oficial Google provided Maven plugin for GAE. It is enough to add plugin definition to pom and declare project as war to actually deploy application to GAE. It even carries out Google Acccount authentication and completelly frees Maven user from downdloading/installig/using GAE SDK tools. Google App Engine uses heavilly horizontal scalling. The next http request to your application can be processed by other JVM instance, so it is bad practice to rely on any "static" or "global" variables. But Vaadin UI state serialization fluorish here.

# No problems, only solutions

## Problem 1 - Serialization, deserialization and autowired.
Lets say we are dealing with such a code:

```java
class MyVaadinButton extends Button {
   @Autowired
   private SpringOnClickHandler handler;
}
```

Problems: SpringOnClickHandler instance can reference quite deep object tree. When serialized it will cause big footprint problems. Some of those object can be not Serializabke, causing Serialize Exception. Moreover deserialization will be problematic if the next request is processed by different instance. Adding the transient keyword will result in null reference after deserialization.


## Problem 2 - Vaadin code and bean factory.

Look again for the MyVaadinButton code. For autowired to work, it is necessary that MyVaadinButton is created by bean factory. If created by new operator, Autowired annotation will have no effect. The problem is that new operator is natural for building Vaadin interfaces, while bean-factory or autowiring is not. Also it has not too much sense to try to make Vaadin framework "Spring aware".

Both problem will be addressed by using Spring AOP AspectJ compile time weaving.

# Implementation

The minimalist Maven POM for combining Spring, GAE and AspectJ:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <!-- Project description -->
    <modelVersion>4.0.0</modelVersion>

    <groupId>name.prokop.bart</groupId>
    <artifactId>bart-gae-poc</artifactId>
    <version>1</version>
    <packaging>war</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <appengine.target.version>1.8.7</appengine.target.version>
    </properties>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>aspectj-maven-plugin</artifactId>
                <version>1.5</version>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                    <complianceLevel>1.7</complianceLevel>
                    <aspectLibraries>
                        <aspectLibrary>
                            <groupId>org.springframework</groupId>
                            <artifactId>spring-aspects</artifactId>
                        </aspectLibrary>
                    </aspectLibraries>
                    <Xlint>warning</Xlint>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>test-compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>com.google.appengine</groupId>
                <artifactId>appengine-maven-plugin</artifactId>
                <version>${appengine.target.version}</version>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <dependency>
            <groupId>com.google.appengine</groupId>
            <artifactId>appengine-api-1.0-sdk</artifactId>
            <version>${appengine.target.version}</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>3.2.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>1.7.3</version>
        </dependency>
    </dependencies>
</project>
```

To deploy application on GAE you need just type: mvn clean appengine:update. Note that the POM is using Maven Central artifacts and new Google official Maven plugin. During the build the classes with @Configurable annotation are weaved with bytecode that will treat them as managed beans. The magic is they will be managed even if created with new operator or deserialized. Web application on Google App Engine needs two additional XML files - web.xml and application descriptor:

```xml
<?xml version="1.0" encoding="utf-8"?>
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xsi:schemaLocation="http://appengine.google.com/ns/1.0 http://googleappengine.googlecode.com/svn/branches/1.2.1/java/docs/appengine-web.xsd">
    <application>bart-gae-poc</application>
    <version>1</version>
    <threadsafe>true</threadsafe>
</appengine-web-app>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4" xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
    <display-name>Natan i Edwin</display-name>
    
    <listener>
        <listener-class>com.appspot.bartgaepoc.WarmupListener</listener-class>
    </listener>
    
    <welcome-file-list>
        <welcome-file>SomeServlet</welcome-file>
    </welcome-file-list>
    
    <servlet>
        <servlet-name>SomeServlet</servlet-name>
        <servlet-class>com.appspot.bartgaepoc.SomeServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>SomeServlet</servlet-name>
        <url-pattern>/SomeServlet</url-pattern>
    </servlet-mapping>
</web-app>
```

As you can see in web.xml we defined one servlet and one listener. The role of the listener will be to initialize Spring framework on application start - called warmup in GAE nomenclature.

```java
package com.appspot.bartgaepoc;

import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class WarmupListener implements ServletContextListener {

    private static AnnotationConfigApplicationContext annotationConfigApplicationContext;

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        annotationConfigApplicationContext = new AnnotationConfigApplicationContext("com.appspot.bartgaepoc");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
    }

}
```

As you see, we initialize Spring using plain annotation configuration. The most remarkable annotation is @EnableSpringConfigured that tells Spring to cooperate with instrumented code:

```
package com.appspot.bartgaepoc;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.aspectj.EnableSpringConfigured;

@Configuration
@EnableSpringConfigured
public class SpringConfiguration {
}
```

And to prove that it works, let's inject Application Context to some servlet. Please note that we have no control when and how the servlet is created. It surely is not created by Spring. But the class is instrumented and the @Autowired (and @PostConstruct, @Transactional, etc.) will work.

```java
package com.appspot.bartgaepoc;

import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.context.ApplicationContext;

@Configurable
public class SomeServlet extends HttpServlet {

    @Autowired
    private ApplicationContext applicationContext;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().println(applicationContext);
    }

}
```

In next part I will show how to take advantage of our setup in Vaadin application.

[Original Blog Post](https://java-in-cloud.blogspot.com/2013/11/java-maven-gae-spring-and-vaadin-all.html)