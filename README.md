# test
Want to learn how to manage your application configuration in a distributed environment? Learn how to do it with Spring Cloud Config.

## Contribute Code
If you would like to become an active contributor to this project, please follow these simple steps:

*   Fork it
*   Create your feature branch
*   Commit your changes
*   Push to the branch
*   Create new Pull Request

Source code can be downloaded from github.

## What you’ll need
*   About 30 minutes
*   A favorite IDE or Spring Tool Suite™ already install
*   JDK 6 or later

## Introduction
In a distributed application environment, there can be several isolated microservices. Each one of them can have different configurations, depending on the environment in which it is running. For instance, typically, a microservice could be tested in a development environment first. Once this testing is done, it can be moved to pre-production environment, where it is tested together with other microservices. And finally, it is moved to production. In each these environments, the same microservice needs different configurations.

One of the most challenging issues that developer face in a microservice architecture, is deciding how to maintain and manage the configuration of the microservices. This is where **Spring Cloud Config** comes very handy.

In this article, we’re going to introduce **Spring Cloud Config**, by first covering the basic and afterwards by learning how to configure both a configuration server and the clients that connect to it and use the configuration provided by the server.

## Overview
**Spring Boot** is a framework aimed to help developers to easily create and build stand-alone, production-grade Spring based Applications that you can "just run".

**Spring Cloud Config** provides server and client-side support for externalized configuration in a distributed system. With the **Spring Cloud Config Server** you have a central place to manage external properties for applications across all environments.

![CANCHITO-DEV: Spring Boot Config - Server and client-side support for externalized configuration](http://www.canchito-dev.com/public/blog/wp-content/uploads/2019/07/Spring-Boot-Config-Server-and-client-side-support-for-externalized-configuration.png) CANCHITO-DEV: Spring Boot Config - Server and client-side support for externalized configuration

**Spring Cloud Config Server** is a centralized service that via HTTP provides all the applications configuration (name-value pairs or equivalent YAML content). The server is embeddable in a **Spring Boot** application, by using the `@EnableConfigServer` annotation.

In other words, the **Spring Cloud Config Server** is simply a **Spring Boot** application, configured as a **Spring Cloud Config Server**, and that is able to retrieve the properties from the configured property source. The property source can be a **Git repository**, **svn** or **Consul service**.  In this article, we will use file-based Git repository as a property source.

A **Spring Boot** application properly configured, can take immediate advantage of the **Spring Config Server**. It also picks up some additional useful features related to `Environment` change events. Any **Spring Boot** application can easily be configured as a **Spring Cloud Config Client**.

## Explaining the demo
The below image explains the application architecture that we will be creating in this article.

![CANCHITO-DEV: Spring Boot Config - Server and client demo](http://www.canchito-dev.com/public/blog/wp-content/uploads/2019/07/Spring-Boot-Config-Server-and-client-demo.png) CANCHITO-DEV: Spring Boot Config - Server and client demo

1.  **Demo App** requests the properties from the Spring Cloud Config Server.
2.  **Spring Cloud Config Server** access the configuration found in the file-based **Git repository**.
3.  It will retrieve two properties files:
    1.  _application.properties__:_ It has the shared properties used by every app requesting the properties to this configuration server.
    2.  _spring-cloud-config.properties__:_ App specific properties.
4.  **Demo app** receives the latest version of the properties and uses them.
5.  A user modifies either the shared-, or the app specific- properties, or maybe both.
6.  User commands **Demo App** to refresh its properties by calling the endpoint `actuator/refresh`.

## File-based Git Repository
Let’s start by creating a local file-based repository. You can create one easily by making a new directory and git committing properties and YAML files to it.

```
$ cd $HOME
$ mkdir config-repo
$ cd config-repo
$ git init .
$ echo canchitodev.service-name=Spring Cloud Config Example > spring-cloud-config.properties
$ git add -A .
$ git commit -m "Add spring-cloud-config.properties"
```
Using the local filesystem for your git repository is intended for testing only. You should use a server to host your configuration repositories in production.

Anyway, once you have created it, you can go to its location, and continue modifying the _spring-cloud-config.properties_ properties file with the help of your favorite editor. It should look something like the below example:

```
# ----------------------------------------
# EMBEDDED SERVER CONFIGURATION (ServerProperties)
# server.servlet.application-display-name = Display name of the application.
# server.servlet.context-path = Context path of the application.
# server.port = Server HTTP port.
# ----------------------------------------
server.servlet.application-display-name=Demo Spring Cloud Config
server.servlet.context-path=/spring-cloud-config-rest
server.port=9876

# ----------------------------------------
# ACTUATOR PROPERTIES
# ----------------------------------------
# ----------------------------------------
# MANAGEMENT HTTP SERVER (ManagementServerProperties)
# management.server.servlet.context-path = Management endpoint context-path (for instance, `/management`). Requires a custom management.server.port.
# management.server.port = Management endpoint HTTP port. Uses the same port as the application by default. Configure a different port to use management-specific SSL.
# ----------------------------------------
management.server.servlet.context-path=/spring-cloud-config-actuator
management.server.port=9877

# ----------------------------------------
# DEMO SPECIFIC PROPERTIES
# ----------------------------------------
canchitodev.service-name=Spring Cloud Config Example
```

Afterwards, create another properties files, and call it _application.properties_. This file will have all the properties that are shared among all the microservices that will be retrieving their properties from the **Spring Cloud Config Server**.

```
# ----------------------------------------
# EMBEDDED SERVER CONFIGURATION (ServerProperties)
# server.address = Network address to which the server should bind to.
# server.connection-timeout = Time in milliseconds that connectors will wait for another HTTP request before closing the connection. When not set, the connector's container-specific default will be used. Use a value of -1 to indicate no (i.e. infinite) timeout.
# server.tomcat.max-connections = Maximum number of connections that the server will accept and process at any given time.
# server.tomcat.max-threads = Maximum amount of worker threads.
# server.tomcat.uri-encoding = Character encoding to use to decode the URI.
# ----------------------------------------
server.address=localhost
server.connection-timeout=60000
server.tomcat.max-connections=100
server.tomcat.max-threads=100
server.tomcat.uri-encoding=UTF-8

# ----------------------------------------
# JPA (JpaBaseConfiguration, HibernateJpaAutoConfiguration)
# spring.jpa.show-sql	= Enable logging of SQL statements.
# ----------------------------------------
spring.jpa.show-sql=false

# ----------------------------------------
# ACTUATOR PROPERTIES
# ----------------------------------------
# ----------------------------------------
# MANAGEMENT HTTP SERVER (ManagementServerProperties)
# management.server.address = Network address that the management endpoints should bind to.
# ----------------------------------------
management.server.address=localhost

# ----------------------------------------
# ENDPOINTS GENERAL CONFIGURATION
# management.endpoints.enabled-by-default= Whether to enable or disable all endpoints by default.
# ----------------------------------------
management.endpoints.enabled-by-default=true

# ----------------------------------------
# ENDPOINTS WEB CONFIGURATION (WebEndpointProperties)
# management.endpoints.web.exposure.include	= Endpoint IDs that should be included or '*' for all.
# management.endpoints.web.exposure.exclude	= Endpoint IDs that should be excluded.
# management.endpoints.web.base-path		= Base path for Web endpoints. Relative to server.servlet.context-path or management.server.servlet.context-path if management.server.port is configured. Default value is /actuator
# management.endpoints.web.path-mapping		= Mapping between endpoint IDs and the path that should expose them.
# ----------------------------------------management.endpoints.web.exposure.include=auditevents,health,shutdown,env,info,flowable,refresh

# ----------------------------------------
# HEALTH ENDPOINT (HealthEndpoint, HealthEndpointProperties)
# management.endpoint.health.show-details = When to show full health details. Default value is never
# ----------------------------------------
management.endpoint.health.show-details=always

# ----------------------------------------
# HEALTH INDICATORS
# management.health.db.enabled = Whether to enable database health check. Default value is true
# management.health.defaults.enabled	= Whether to enable default health indicators. Default value is true
# management.health.diskspace.enabled = Whether to enable disk space health check. Default value is true
# management.health.diskspace.path = Path used to compute the available disk space.
# management.health.diskspace.threshold = Minimum disk space, in bytes, that should be available. Default value is 0
# management.health.mail.enabled = Whether to enable Mail health check. Default value is true
# management.health.status.http-mapping = Mapping of health statuses to HTTP status codes. By default, registered health statuses map to sensible defaults (for example, UP maps to 200).
# management.health.status.order = Comma-separated list of health statuses in order of severity. Default value is DOWN,OUT_OF_SERVICE,UP,UNKNOWN
# ----------------------------------------
management.health.defaults.enabled=true
management.health.diskspace.enabled=true
management.health.diskspace.path=C:\\
management.health.diskspace.threshold=0
management.health.mail.enabled=true
management.health.status.order=UP,DOWN,OUT_OF_SERVICE,UNKNOWN
```

We have included the sample properties files in the resources folder of the config server project.

## Creating the Project with Maven
To start with, let's create the directory structure as follow:

```
spring-cloud-config
└── cloud-config-client
    └── src
        └── main
            └── java
            └── resources
└── cloud-config-server
    └── src
        └── main
            └── java
            └── resources
```
To get you started quickly, here are the complete configurations for the server and client applications:

`cloud-config-server/pom.xml`

```xml
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.6.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <groupId>com.canchitodev.spring.projects</groupId>
  <artifactId>cloud-config-server</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>cloud-config-server</name>
  <description>Demo project for Spring Boot using Spring Cloud Config Server</description>

  <url>http://www.canchito-dev.com/public/blog/2019/07/19/spring-cloud-config-server-and-client-side-support-for-externalized-configuration/</url>

  <issueManagement>
    <url>https://github.com/canchito-dev/spring-cloud-config/issues</url>
    <system>Canchito Development</system>
  </issueManagement>

  <organization>
    <name>Canchito Development</name>
    <url>http://www.canchito-dev.com</url>
  </organization>

  <properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-config-server</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <scope>provided</scope>
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
      </plugin>
    </plugins>
  </build>
</project>
```

`cloud-config-client/pom.xml`

```xml
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.6.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <groupId>com.canchitodev.spring.projects</groupId>
  <artifactId>cloud-config-client</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>cloud-config-client</name>
  <description>Canchito-Dev demo project of a Spring Boot REST API with using Spring Cloud Config Client</description>

  <url>http://www.canchito-dev.com/public/blog/2019/07/19/spring-cloud-config-server-and-client-side-support-for-externalized-configuration/</url>

  <issueManagement>
    <url>https://github.com/canchito-dev/spring-cloud-config/issues</url>
    <system>Canchito Development</system>
  </issueManagement>

  <organization>
    <name>Canchito Development</name>
    <url>http://www.canchito-dev.com</url>
  </organization>

  <properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>

    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <scope>provided</scope>
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
      </plugin>
    </plugins>
  </build>
</project>
```

## Spring Cloud Config Server Implementation
Let’s now move on to the definition the config server. As we have said, any **Spring Boot** application can be configured as a configuration server.

Under `cloud-config-server/src/main/java/`, create the package `com.canchitodev.cloud.config.server`. Here we create the class `CloudConfigServer`.

```java
package com.canchitodev.cloud.config.server;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@EnableConfigServer
@SpringBootApplication
public class CloudConfigServer {

  public static void main(String[] args) {
    SpringApplication.run(CloudConfigServer.class, args);
  }
}
```

Following, we need to tell the config server from which repository it will be retrieving the properties. We have already created a file-based local **Git** repository. And if you recall, we added a property `canchitodev.service-name`.

Add the _spring-cloud-config.properties_ properties file under `cloud-config-server/src/main/resources/`.

```
# ----------------------------------------
# WEB PROPERTIES
# ----------------------------------------
# ----------------------------------------
# EMBEDDED SERVER CONFIGURATION (ServerProperties)
# server.port	= Server HTTP port. Default is 8080
# ---------------------------------------- 
server.port=8888

# spring.cloud.config.server.git.uri	= URI of remote repository.
spring.cloud.config.server.git.uri=file:///${user.home}/git/spring-cloud-config-repo
```

Specify the path to the **Git** repository by specifying the `spring.cloud.config.server.git.uri` property. Make sure to also specify a different `server.port` value to avoid port conflicts when you run both this server and another **Spring Boot** application on the same machine.

## Testing Spring Cloud Config Server
First, start the server, as follows:

```
$ cd cloud-config-server
$ mvnw spring-boot:run
```

The server is a **Spring Boot** application, so you can run it from your **IDE** if you prefer to do so (the main class is `CloudConfigServer`).

```
_[2m2019-07-21 15:36:56.933_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mtrationDelegate$BeanPostProcessorChecker_[0;39m _[2m:_[0;39m Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$EnhancerBySpringCGLIB$a68ae6f4] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
_[32m :: Spring Boot :: _[39m      _[2m (v2.1.6.RELEASE)_[0;39m

_[2m2019-07-21 15:36:57.418_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mc.c.c.config.server.CloudConfigServer   _[0;39m _[2m:_[0;39m No active profile set, falling back to default profiles: default
_[2m2019-07-21 15:36:58.480_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.cloud.context.scope.GenericScope    _[0;39m _[2m:_[0;39m BeanFactory id=624aabd2-b8a6-3b40-b5c5-d9170132d688
_[2m2019-07-21 15:36:58.512_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mtrationDelegate$BeanPostProcessorChecker_[0;39m _[2m:_[0;39m Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$EnhancerBySpringCGLIB$a68ae6f4] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
_[2m2019-07-21 15:36:58.741_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.b.w.embedded.tomcat.TomcatWebServer _[0;39m _[2m:_[0;39m Tomcat initialized with port(s): 8888 (http)
_[2m2019-07-21 15:36:58.764_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.apache.catalina.core.StandardService  _[0;39m _[2m:_[0;39m Starting service [Tomcat]
_[2m2019-07-21 15:36:58.765_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36morg.apache.catalina.core.StandardEngine _[0;39m _[2m:_[0;39m Starting Servlet engine: [Apache Tomcat/9.0.21]
_[2m2019-07-21 15:36:58.954_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.a.c.c.C.[Tomcat].[localhost].[/]      _[0;39m _[2m:_[0;39m Initializing Spring embedded WebApplicationContext
_[2m2019-07-21 15:36:58.954_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.web.context.ContextLoader           _[0;39m _[2m:_[0;39m Root WebApplicationContext: initialization completed in 1524 ms
_[2m2019-07-21 15:36:59.888_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.s.concurrent.ThreadPoolTaskExecutor _[0;39m _[2m:_[0;39m Initializing ExecutorService 'applicationTaskExecutor'
_[2m2019-07-21 15:37:00.771_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.b.a.e.web.EndpointLinksResolver     _[0;39m _[2m:_[0;39m Exposing 2 endpoint(s) beneath base path '/actuator'
_[2m2019-07-21 15:37:00.887_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.b.w.embedded.tomcat.TomcatWebServer _[0;39m _[2m:_[0;39m Tomcat started on port(s): 8888 (http) with context path ''
_[2m2019-07-21 15:37:00.890_[0;39m _[32m INFO_[0;39m _[35m10292_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mc.c.c.config.server.CloudConfigServer   _[0;39m _[2m:_[0;39m Started CloudConfigServer in 5.331 seconds (JVM running for 6.561)
```

Next try out the server with the help of [curl](https://curl.haxx.se/) or [Postman](https://www.getpostman.com/):

**`GET`** `http://localhost:8888/spring-cloud-config/default`

In the response body you should see something like this:

```json
{
    "name": "spring-cloud-config",
    "profiles": [
        "default"
    ],
    "label": null,
    "version": "71ca4e5bcbab0a79b45d72251e4d8805cfdc9cc4",
    "state": null,
    "propertySources": [
        {
            "name": "file:///C:\\Users\\josec/git/spring-cloud-config-repo/spring-cloud-config.properties",
            "source": {
                "server.servlet.application-display-name": "Demo Spring Cloud Config",
                "server.servlet.context-path": "/spring-cloud-config-rest",
                "server.port": "9876",
                "management.server.servlet.context-path": "/spring-cloud-config-actuator",
                "management.server.port": "9877",
                "canchitodev.service-name": "Spring Cloud Config Example"
            }
        },
        {
            "name": "file:///C:\\Users\\josec/git/spring-cloud-config-repo/application.properties",
            "source": {
                "server.address": "localhost",
                "server.connection-timeout": "60000",
                "server.tomcat.max-connections": "100",
                "server.tomcat.max-threads": "100",
                "server.tomcat.uri-encoding": "UTF-8",
                "spring.jpa.show-sql": "false",
                "management.server.address": "localhost",
                "management.endpoints.enabled-by-default": "true",
                "management.endpoints.web.exposure.include": "auditevents,health,shutdown,env,info,flowable,refresh",
                "management.endpoint.health.show-details": "always",
                "management.health.defaults.enabled": "true",
                "management.health.diskspace.enabled": "true",
                "management.health.diskspace.path": "C:\\",
                "management.health.diskspace.threshold": "0",
                "management.health.mail.enabled": "true",
                "management.health.status.order": "UP,DOWN,OUT_OF_SERVICE,UNKNOWN"
            }
        }
    ]
}
```

As you can see, the response body brings two properties files (_spring-cloud-config.properties_ and _application.properties_). If you have another _application.properties_ properties file defined in your client application, it will override the properties returned by the **Spring Cloud Config Server**.

The HTTP service has resources in the following form:

```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

In which the _{label}_ placeholder refers to a **Git** branch, _{application}_ to the client’s application name and the _{profile}_ to the client’s current active application profile.

That’s it. We are done with the server implementation. Let’s move on to the client implementation.

## Spring Cloud Config Client Implementation
Let’s now move on to the definition the client application. This will be a very simple one, consisting of a REST controller with one _GET_ method.

The properties to configure the client application, must be read before the rest of the application’s configuration is read from the **Config Server**, during the bootstrap phase. Under `cloud-config-client/src/main/resources/`, we will add the _bootstrap.yml_ file.

```
# ----------------------------------------
# spring.application.name = Application name
# spring.cloud.config.uri = The URI of the remote server (default http://localhost:8888).
# spring.cloud.config.fail-fast = Flag to indicate that failure to connect to the server is fatal (default false). 
# ----------------------------------------

spring:
  application:
    name: spring-cloud-config

  cloud:
    config:
      uri: http://localhost:8888
      fail-fast: true 
server.port=8888
```

In is important to notice, that we have specified the application name and the URI of the configuration server. Without them, the client application does not know how to communicate with the server, and the server does not know which properties files to retrieve.

It is important to also enable actuator’s `/refresh` endpoint. We already enable it in the shared _application.properties_ file. Without this endpoint, we will not be able to tell the client application to request the properties again.

```
management.endpoints.web.exposure.include=auditevents,health,shutdown,env,info,flowable,refresh
```

Now, under `cloud-config-client/src/main/java/`, create the package `com.canchitodev.cloud.config.client`. Here we create the class `CloudConfigClient`. Right below this class, we will create the REST controller class. We will call it `ServiceRestController`. Notice `@RefreshScope` annotation. This is the annotation that tells the client application that in this class, there are properties that need to be reloaded whenever the `/refresh` endpoint is called.

```java
package com.canchitodev.cloud.config.client;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
public class CloudConfigClient {

  public static void main(String[] args) {
    SpringApplication.run(CloudConfigClient.class, args);
  }

}

@RefreshScope
@RestController
class ServiceRestController {

    @Value("${canchitodev.service-name:Service Name}")
    private String service;

    @RequestMapping("/service")
    String getService() {
        return this.service;
    }
}
```

## End-to-End Testing
First, start the client, as follows:

```
$ cd cloud-config-client
$ mvnw spring-boot:run
```

```
_[2m2019-07-21 21:24:36.562_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mtrationDelegate$BeanPostProcessorChecker_[0;39m _[2m:_[0;39m Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$EnhancerBySpringCGLIB$cc1ef749] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
_[32m :: Spring Boot :: _[39m      _[2m (v2.1.6.RELEASE)_[0;39m

_[2m2019-07-21 21:24:37.015_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mc.c.c.ConfigServicePropertySourceLocator_[0;39m _[2m:_[0;39m Fetching config from server at : http://localhost:8888
_[2m2019-07-21 21:24:37.955_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mc.c.c.ConfigServicePropertySourceLocator_[0;39m _[2m:_[0;39m Located environment: name=spring-cloud-config, profiles=[default], label=null, version=71ca4e5bcbab0a79b45d72251e4d8805cfdc9cc4, state=null
_[2m2019-07-21 21:24:37.956_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mb.c.PropertySourceBootstrapConfiguration_[0;39m _[2m:_[0;39m Located property source: CompositePropertySource {name='configService', propertySources=[MapPropertySource {name='configClient'}, MapPropertySource {name='file:///C:\Users\josec/git/spring-cloud-config-repo/spring-cloud-config.properties'}, MapPropertySource {name='file:///C:\Users\josec/git/spring-cloud-config-repo/application.properties'}]}
_[2m2019-07-21 21:24:37.964_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mc.c.c.config.client.CloudConfigClient   _[0;39m _[2m:_[0;39m No active profile set, falling back to default profiles: default
_[2m2019-07-21 21:24:38.842_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36m.s.d.r.c.RepositoryConfigurationDelegate_[0;39m _[2m:_[0;39m Bootstrapping Spring Data repositories in DEFAULT mode.
_[2m2019-07-21 21:24:38.864_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36m.s.d.r.c.RepositoryConfigurationDelegate_[0;39m _[2m:_[0;39m Finished Spring Data repository scanning in 13ms. Found 0 repository interfaces.
_[2m2019-07-21 21:24:39.071_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.cloud.context.scope.GenericScope    _[0;39m _[2m:_[0;39m BeanFactory id=32adeae3-60ab-3b5f-9cfd-74fd24608c13
_[2m2019-07-21 21:24:39.154_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mtrationDelegate$BeanPostProcessorChecker_[0;39m _[2m:_[0;39m Bean 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration' of type [org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration$EnhancerBySpringCGLIB$b004f44c] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
_[2m2019-07-21 21:24:39.227_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mtrationDelegate$BeanPostProcessorChecker_[0;39m _[2m:_[0;39m Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$EnhancerBySpringCGLIB$cc1ef749] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
_[2m2019-07-21 21:24:39.481_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.b.w.embedded.tomcat.TomcatWebServer _[0;39m _[2m:_[0;39m Tomcat initialized with port(s): 9876 (http)
_[2m2019-07-21 21:24:39.505_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.apache.catalina.core.StandardService  _[0;39m _[2m:_[0;39m Starting service [Tomcat]
_[2m2019-07-21 21:24:39.506_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36morg.apache.catalina.core.StandardEngine _[0;39m _[2m:_[0;39m Starting Servlet engine: [Apache Tomcat/9.0.21]
_[2m2019-07-21 21:24:39.625_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36m.a.c.c.C.[.[.[/spring-cloud-config-rest]_[0;39m _[2m:_[0;39m Initializing Spring embedded WebApplicationContext
_[2m2019-07-21 21:24:39.625_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.web.context.ContextLoader           _[0;39m _[2m:_[0;39m Root WebApplicationContext: initialization completed in 1632 ms
_[2m2019-07-21 21:24:40.104_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mcom.zaxxer.hikari.HikariDataSource      _[0;39m _[2m:_[0;39m HikariPool-1 - Starting...
_[2m2019-07-21 21:24:40.344_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mcom.zaxxer.hikari.HikariDataSource      _[0;39m _[2m:_[0;39m HikariPool-1 - Start completed.
_[2m2019-07-21 21:24:40.457_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.hibernate.jpa.internal.util.LogHelper _[0;39m _[2m:_[0;39m HHH000204: Processing PersistenceUnitInfo [
  name: default
  ...]
_[2m2019-07-21 21:24:40.564_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36morg.hibernate.Version                   _[0;39m _[2m:_[0;39m HHH000412: Hibernate Core {5.3.10.Final}
_[2m2019-07-21 21:24:40.567_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36morg.hibernate.cfg.Environment           _[0;39m _[2m:_[0;39m HHH000206: hibernate.properties not found
_[2m2019-07-21 21:24:40.799_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.hibernate.annotations.common.Version  _[0;39m _[2m:_[0;39m HCANN000001: Hibernate Commons Annotations {5.0.4.Final}
_[2m2019-07-21 21:24:41.083_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36morg.hibernate.dialect.Dialect           _[0;39m _[2m:_[0;39m HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
_[2m2019-07-21 21:24:41.515_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.h.t.schema.internal.SchemaCreatorImpl _[0;39m _[2m:_[0;39m HHH000476: Executing import script 'org.hibernate.tool.schema.internal.exec.ScriptSourceInputNonExistentImpl@6467ddc7'
_[2m2019-07-21 21:24:41.522_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mj.LocalContainerEntityManagerFactoryBean_[0;39m _[2m:_[0;39m Initialized JPA EntityManagerFactory for persistence unit 'default'
_[2m2019-07-21 21:24:41.911_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.s.concurrent.ThreadPoolTaskExecutor _[0;39m _[2m:_[0;39m Initializing ExecutorService 'applicationTaskExecutor'
_[2m2019-07-21 21:24:41.965_[0;39m _[33m WARN_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36maWebConfiguration$JpaWebMvcConfiguration_[0;39m _[2m:_[0;39m spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
_[2m2019-07-21 21:24:43.011_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.b.w.embedded.tomcat.TomcatWebServer _[0;39m _[2m:_[0;39m Tomcat started on port(s): 9876 (http) with context path '/spring-cloud-config-rest'
_[2m2019-07-21 21:24:43.177_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.b.w.embedded.tomcat.TomcatWebServer _[0;39m _[2m:_[0;39m Tomcat initialized with port(s): 9877 (http)
_[2m2019-07-21 21:24:43.178_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.apache.catalina.core.StandardService  _[0;39m _[2m:_[0;39m Starting service [Tomcat]
_[2m2019-07-21 21:24:43.179_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36morg.apache.catalina.core.StandardEngine _[0;39m _[2m:_[0;39m Starting Servlet engine: [Apache Tomcat/9.0.21]
_[2m2019-07-21 21:24:43.204_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36m.c.C.[.[.[/spring-cloud-config-actuator]_[0;39m _[2m:_[0;39m Initializing Spring embedded WebApplicationContext
_[2m2019-07-21 21:24:43.204_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.web.context.ContextLoader           _[0;39m _[2m:_[0;39m Root WebApplicationContext: initialization completed in 190 ms
_[2m2019-07-21 21:24:43.251_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.b.a.e.web.EndpointLinksResolver     _[0;39m _[2m:_[0;39m Exposing 6 endpoint(s) beneath base path '/actuator'
_[2m2019-07-21 21:24:43.341_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mo.s.b.w.embedded.tomcat.TomcatWebServer _[0;39m _[2m:_[0;39m Tomcat started on port(s): 9877 (http) with context path '/spring-cloud-config-actuator'
_[2m2019-07-21 21:24:43.343_[0;39m _[32m INFO_[0;39m _[35m5876_[0;39m _[2m---_[0;39m _[2m[           main]_[0;39m _[36mc.c.c.config.client.CloudConfigClient   _[0;39m _[2m:_[0;39m Started CloudConfigClient in 7.827 seconds (JVM running for 8.588)
```

Next try out the client with the help of [curl](https://curl.haxx.se/) or [Postman](https://www.getpostman.com/):

**`GET`** `http://localhost:9876/spring-cloud-config-rest/service`

In the response body you should see something like this:

```json
Spring Cloud Config Example
```

The response body corresponds to the property found in the _spring-cloud-config.properties_ properties file.

```
# ----------------------------------------
# DEMO SPECIFIC PROPERTIES
# ----------------------------------------
canchitodev.service-name=Spring Cloud Config Example
```

Go to the repository and modify it. Once you have done that, you need to call actuator’s `/refresh` endpoint.

**`POST`** `http://localhost:9877/spring-cloud-config-actuator/actuator/refresh`

The response body should look as follow:

```json
[
    "canchitodev.service-name"
]
```

## Summary
Thanks to **Spring Cloud Config**, we are able to share, manage and handle in an easier and more convenient way, the configuration of a distributed application environment.

We hope that, even though this was a very basic introduction, you understood how to use and configure this tool. In upcoming posts, we will be introducing **Spring Cloud Bus**. Thanks to is, we will be able to automatically refresh the properties, instead of calling the /refresh endpoint. But this is another post 😊.

Please feel free to contact us. We will gladly response to any doubt or question you might have.

The full implementation of this article can be found in the **GitHub** project – this is a **Maven**-based project, so it should be easy to import and run as it is.