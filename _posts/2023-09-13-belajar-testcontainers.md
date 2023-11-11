---
layout: post
title: Belajar Testcontainers
modified: 2023-02-01T21:08:21+07:00
categories:
description: belajar testcontainers
tags: [testcontainers, integration, container, docker]
image:
    background: abstract-2.png
comments: true
share: true
date: 2023-09-13T21:08:21+07:00
---

## Apa Itu Testcontainers ?

>>Testcontainers adalah framework yang menyediakan ketersediaan environment dan infrastruktur untuk kebutuhan automate test.

Latar belakang dibuat nya testcontainers adalah untuk mempermudah developer pada saat mejalankan unit test tanpa perlu melakukan setup infrastruktur seperti database, message broker dan lain sebagai nya. Testcontainers nanti nya akan menjalankan kebutuhan inftrastuktur aplikasi yang dibutuhkan dengan menggunakan container (biasa nya menggunakan [docker](https://rizkimufrizal.github.io/belajar-docker/)). Dengan ada nya testcontainers, maka developer dapat fokus pada pembuatan unit test dan integration test. 

Untuk saat ini, testcontainers hanya memiliki module - module tertentu saja sehingga masih terdapat keterbatasan untuk module - module yang tidak common. Contoh module yang dapat kita gunakan adalah seperti database postgresql, mysql, mongodb, kafka dan lain sebagai nya. Tetapi untuk saat ini, module - module yang terdapat pada testcontainers baru di support untuk bahasa pemrograman java, noed js, go dan .NET, berikut [list module - module nya](https://testcontainers.com/modules/).

## Setup Testcontainers Pada Spring Boot

Pada artikel ini, penulis akan mencoba mendemokan dengan menggunakan spring boot. Secara default, spring boot sudah support dengan testcontainers dan juga integration test sehingga dapat mempermudah developer dalam menggunakan module - module testcontainers. Seperti biasa, silahkan buka [spring initializr](https://start.spring.io/). lalu isi seperti berikut

![Screenshot from 2023-09-12 22-07-20.png](../images/Screenshot from 2023-09-12 22-07-20.png)

Lalu setelah selesai, silahkan download dan buka dengan IDE. Silahkan buka file pom.xml lalu tambahkan dependecy rest-assured seperti berikut

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.3</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.belajar.testcontainers</groupId>
    <artifactId>belajar-testcontainers</artifactId>
    <version>1.0.0</version>
    <name>belajar-testcontainers</name>
    <description>belajar testcontainers</description>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
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
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-testcontainers</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>rest-assured</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

Silahkan buka file `application.properties` di dalam folder `resources` lalu ubah seperti berikut

```properties
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=jdbc:postgresql://localhost:5432/belajar_testcontainers
spring.datasource.username=root
spring.datasource.password=root

spring.jpa.show-sql=true
spring.jpa.properties.jakarta.persistence.sharedCache.mode=UNSPECIFIED
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.hibernate.ddl-auto=create-drop
```

Lalu pada folder test, silahkan buat sebuah class `TestBelajarTestcontainersApplication` dan juga file `application.properties` pada folder `resources` lalu ubah seperti berikut

```properties
spring.jpa.show-sql=true
spring.jpa.properties.jakarta.persistence.sharedCache.mode=UNSPECIFIED
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.hibernate.ddl-auto=create-drop
```

### Membuat Domain Class

Domain class digunakan untuk keperluan mapping dari class ke table yang terdapat pada database. Silahkan buat sebuah package `domain` lalu silahkan buat class `Product` di dalam package tersebut dan ubah codingan nya seperti berikut.

```java
package com.belajar.testcontainers.domain;

import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import org.hibernate.annotations.UuidGenerator;

import java.io.Serializable;
import java.math.BigDecimal;
import java.util.UUID;

@Entity
@Table(name = "tb_product")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Product implements Serializable {
    @Id
    @UuidGenerator
    private UUID id;

    private String name;

    private BigDecimal price;

    private Integer quantity;
}
```

### Membuat Repository Class

Repository class digunakan untuk process query ke database. Silahkan buat sebuah package `repository` lalu silahkan buat class `ProductRepository` di dalam package tersebut dan ubah codingan nya seperti berikut.

```java
package com.belajar.testcontainers.repository;

import com.belajar.testcontainers.domain.Product;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.UUID;

public interface ProductRepository extends JpaRepository<Product, UUID> {
}
```

### Membuat Service Class

Service class digunakan untuk logic bisnis. Silahkan buat sebuah package `service` dan `Service.impl` lalu silahkan buat class `ProductService` di dalam package tersebut dan ubah codingan nya seperti berikut.

```java
package com.belajar.testcontainers.service;

import com.belajar.testcontainers.domain.Product;

import java.util.Optional;
import java.util.UUID;

public interface ProductService {
    Product save(Product product);

    Optional<Product> findOne(UUID id);
}
```

Lalu buat sebuah class `ProductServiceImpl` di dalam package `Service.impl` dan ubah codingan nya seperti berikut.

```java
package com.belajar.testcontainers.service.impl;

import com.belajar.testcontainers.domain.Product;
import com.belajar.testcontainers.repository.ProductRepository;
import com.belajar.testcontainers.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.Optional;
import java.util.UUID;

@Service
public class ProductServiceImpl implements ProductService {

    @Autowired
    private ProductRepository productRepository;

    @Override
    public Product save(Product product) {
        return productRepository.save(product);
    }

    @Override
    public Optional<Product> findOne(UUID id) {
        return productRepository.findById(id);
    }
}
```

### Membuat Controller Class

Controller class digunakan untuk menerima request atau traffic dari client. Silahkan buat sebuah package `controller` lalu silahkan buat class `ProductController` di dalam package tersebut dan ubah codingan nya seperti berikut.

```java
package com.belajar.testcontainers.controller;

import com.belajar.testcontainers.domain.Product;
import com.belajar.testcontainers.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
@RequestMapping(value = "/api")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping(value = "/products/{id}")
    public ResponseEntity<?> findOne(@PathVariable("id") UUID id) {
        return new ResponseEntity<>(productService.findOne(id), HttpStatus.OK);
    }

    @PostMapping(value = "/products")
    public ResponseEntity<?> save(@RequestBody Product product) {
        return new ResponseEntity<>(productService.save(product), HttpStatus.OK);
    }

}
```

## Membuat Testcontainers Class

Silahkan buka class `TestBelajarTestcontainersApplication` di dalam folder test, lalu tambahkan code berikut

```java
package com.belajar.testcontainers;

import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestConfiguration(proxyBeanMethods = false)
public class TestBelajarTestcontainersApplication {

}
```

fungsi dari annotation `@SpringBootTest` untuk menjalankan test pada spring boot, lalu spring boot akan dijalankan dengan menggunakan port random, sedangkan fungsi dari annotation `@TestConfiguration` digunakan untuk membuat unit test pada spring boot, melakukan override pada bean spring boot dan membuat bean baru sesuai dengan kebutuhan unit test. Selanjutnya untuk testcontainers pada artikel ini, kita akan menggunakan module postgresql, untuk dapat menjalankan module tersebut silahkan tambahkan code berikut.

```java
package com.belajar.testcontainers;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeAll;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestConfiguration(proxyBeanMethods = false)
public class TestBelajarTestcontainersApplication {

    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>(
            "postgres:alpine"
    );

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @BeforeAll
    static void beforeAll() {
        postgres.start();
    }

    @AfterAll
    static void afterAll() {
        postgres.stop();
    }
}
```

Pada codingan diatas dapat dilihat bahwa penulis mendeklarasikan `PostgreSQLContainer` dimana image yang digunakan adalah `postgres:alpine`. Kemudian karena postgresql ini dijalankan pada testcontainer maka spring membutuhkan properties seperti url, username dan password sehingga class `DynamicPropertyRegistry` dapat mereplace properties tersebut. Pada annotation `@BeforeAll` dan `@AfterAll` berfungsi untuk menjalankan dan mematikan postgresql. Selanjutkan kita memerlukan setup untuk kebutuhan [REST Assured](https://rest-assured.io/) dengan menambahkan code berikut

```java
package com.belajar.testcontainers;

import io.restassured.RestAssured;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestConfiguration(proxyBeanMethods = false)
public class TestBelajarTestcontainersApplication {

    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>(
            "postgres:alpine"
    );

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @BeforeAll
    static void beforeAll() {
        postgres.start();
    }

    @LocalServerPort
    private Integer port;

    @BeforeEach
    void setUp() {
        RestAssured.baseURI = "http://localhost:" + port;
    }

    @AfterAll
    static void afterAll() {
        postgres.stop();
    }
}
```

Pada bagian annotation `@LocalServerPort` berfungsi untuk mengambil port spring boot yang akan berjalan pada saat unit test berjalan. Annotation `@BeforeEach` berfungsi untuk melakukan setup pada `RestAssured` setelah spring boot running dengan port random. Langkah selanjutnya silahkan tambahkan code berikut untuk melakukan test untuk save dan findOne ke database.

```java
package com.belajar.testcontainers;

import com.belajar.testcontainers.domain.Product;
import com.belajar.testcontainers.repository.ProductRepository;
import io.restassured.RestAssured;
import io.restassured.http.ContentType;
import org.hamcrest.Matchers;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.boot.test.web.server.LocalServerPort;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;

import java.math.BigDecimal;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestConfiguration(proxyBeanMethods = false)
public class TestBelajarTestcontainersApplication {

    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>(
            "postgres:alpine"
    );

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @LocalServerPort
    private Integer port;

    @BeforeAll
    static void beforeAll() {
        postgres.start();
    }

    @BeforeEach
    void setUp() {
        RestAssured.baseURI = "http://localhost:" + port;
    }

    @AfterAll
    static void afterAll() {
        postgres.stop();
    }

    @Autowired
    private ProductRepository productRepository;

    @Test
    void save() {
        var product = Product.builder()
                .name("aqua")
                .price(BigDecimal.valueOf(1000L))
                .quantity(5)
                .build();

        RestAssured.given()
                .contentType(ContentType.JSON)
                .body(product)
                .post("/api/products")
                .then()
                .statusCode(200)
                .body("name", Matchers.is(product.getName()));
    }


    @Test
    void testFindOne() {
        var product = Product.builder()
                .name("aqua")
                .price(BigDecimal.valueOf(1000L))
                .quantity(5)
                .build();

        var result = productRepository.save(product);

        RestAssured.given()
                .contentType(ContentType.JSON)
                .when()
                .get("/api/products/" + result.getId())
                .then()
                .statusCode(200)
                .body("name", Matchers.is(result.getName()));
    }
}
```

Lalu selanjutnya silahkan jalankan test nya dengan menggunakan command

```shell
mvn clean test
```

maka hasil nya akan seperti berikut

```shell
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------< com.belajar.testcontainers:belajar-testcontainers >----------
[INFO] Building belajar-testcontainers 1.0.0
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- clean:3.2.0:clean (default-clean) @ belajar-testcontainers ---
[INFO] Deleting /home/rizki/Documents/project/belajar-project/spring-project/belajar-testcontainers/target
[INFO] 
[INFO] --- resources:3.3.1:resources (default-resources) @ belajar-testcontainers ---
[INFO] Copying 1 resource from src/main/resources to target/classes
[INFO] Copying 0 resource from src/main/resources to target/classes
[INFO] 
[INFO] --- compiler:3.11.0:compile (default-compile) @ belajar-testcontainers ---
[INFO] Changes detected - recompiling the module! :source
[INFO] Compiling 6 source files with javac [debug release 17] to target/classes
[INFO] 
[INFO] --- resources:3.3.1:testResources (default-testResources) @ belajar-testcontainers ---
[INFO] Copying 1 resource from src/test/resources to target/test-classes
[INFO] 
[INFO] --- compiler:3.11.0:testCompile (default-testCompile) @ belajar-testcontainers ---
[INFO] Changes detected - recompiling the module! :dependency
[INFO] Compiling 1 source file with javac [debug release 17] to target/test-classes
[INFO] 
[INFO] --- surefire:3.0.0:test (default-test) @ belajar-testcontainers ---
[INFO] Using auto detected provider org.apache.maven.surefire.junitplatform.JUnitPlatformProvider
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.belajar.testcontainers.TestBelajarTestcontainersApplication
13:36:39.856 [main] INFO org.springframework.test.context.support.AnnotationConfigContextLoaderUtils -- Could not detect default configuration classes for test class [com.belajar.testcontainers.TestBelajarTestcontainersApplication]: TestBelajarTestcontainersApplication does not declare any static, non-private, non-final, nested classes annotated with @Configuration.
13:36:39.931 [main] INFO org.springframework.boot.test.context.SpringBootTestContextBootstrapper -- Found @SpringBootConfiguration com.belajar.testcontainers.BelajarTestcontainersApplication for test class com.belajar.testcontainers.TestBelajarTestcontainersApplication
13:36:40.050 [main] INFO org.testcontainers.utility.ImageNameSubstitutor -- Image name substitution will be performed by: DefaultImageNameSubstitutor (composite of 'ConfigurationFileImageNameSubstitutor' and 'PrefixingImageNameSubstitutor')
13:36:40.188 [main] INFO org.testcontainers.dockerclient.DockerClientProviderStrategy -- Loaded org.testcontainers.dockerclient.UnixSocketClientProviderStrategy from ~/.testcontainers.properties, will try it first
13:36:40.358 [main] INFO org.testcontainers.dockerclient.DockerClientProviderStrategy -- Found Docker environment with local Unix socket (unix:///var/run/docker.sock)
13:36:40.359 [main] INFO org.testcontainers.DockerClientFactory -- Docker host IP address is localhost
13:36:40.372 [main] INFO org.testcontainers.DockerClientFactory -- Connected to docker: 
  Server Version: 24.0.6
  API Version: 1.43
  Operating System: Ubuntu 22.04.3 LTS
  Total Memory: 14812 MB
13:36:40.412 [main] INFO tc.testcontainers/ryuk:0.5.1 -- Creating container for image: testcontainers/ryuk:0.5.1
13:36:40.513 [main] INFO tc.testcontainers/ryuk:0.5.1 -- Container testcontainers/ryuk:0.5.1 is starting: 5bd8ac42d26ceb6ccbf70fef204729d955ad56c08787726138805e80d5c10639
13:36:40.848 [main] INFO tc.testcontainers/ryuk:0.5.1 -- Container testcontainers/ryuk:0.5.1 started in PT0.470571487S
13:36:40.852 [main] INFO org.testcontainers.utility.RyukResourceReaper -- Ryuk started - will monitor and terminate Testcontainers containers on JVM exit
13:36:40.852 [main] INFO org.testcontainers.DockerClientFactory -- Checking the system...
13:36:40.853 [main] INFO org.testcontainers.DockerClientFactory -- ✔︎ Docker server version should be at least 1.6.0
13:36:40.856 [main] INFO tc.postgres:alpine -- Pulling docker image: postgres:alpine. Please be patient; this may take some time but only needs to be done once.
13:36:43.815 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Starting to pull image
13:36:43.826 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  0 pending,  0 downloaded,  0 extracted, (0 bytes/0 bytes)
13:36:44.968 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  7 pending,  1 downloaded,  0 extracted, (34 KB/? MB)
13:36:45.018 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  6 pending,  2 downloaded,  0 extracted, (34 KB/? MB)
13:36:46.230 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  5 pending,  3 downloaded,  0 extracted, (2 MB/? MB)
13:36:46.828 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  4 pending,  4 downloaded,  0 extracted, (4 MB/? MB)
13:36:46.885 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  4 pending,  4 downloaded,  1 extracted, (4 MB/? MB)
13:36:46.895 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  4 pending,  4 downloaded,  2 extracted, (4 MB/? MB)
13:36:46.902 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  4 pending,  4 downloaded,  3 extracted, (4 MB/? MB)
13:36:47.400 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  3 pending,  5 downloaded,  3 extracted, (5 MB/? MB)
13:36:50.222 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  2 pending,  6 downloaded,  3 extracted, (9 MB/? MB)
13:37:16.964 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  1 pending,  7 downloaded,  3 extracted, (88 MB/? MB)
13:37:18.064 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  1 pending,  7 downloaded,  4 extracted, (89 MB/? MB)
13:37:18.074 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  1 pending,  7 downloaded,  5 extracted, (89 MB/? MB)
13:37:18.081 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  1 pending,  7 downloaded,  6 extracted, (89 MB/? MB)
13:37:18.089 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  1 pending,  7 downloaded,  7 extracted, (89 MB/? MB)
13:37:18.097 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pulling image layers:  1 pending,  7 downloaded,  8 extracted, (89 MB/? MB)
13:37:18.106 [docker-java-stream-1567680985] INFO tc.postgres:alpine -- Pull complete. 8 layers, pulled in 34s (downloaded 89 MB at 2 MB/s)
13:37:18.111 [main] INFO tc.postgres:alpine -- Creating container for image: postgres:alpine
13:37:18.280 [main] INFO tc.postgres:alpine -- Container postgres:alpine is starting: de1c5c36fa428724411fda32a1d53051304d5001d630431dd0427957152c58e0
13:37:19.447 [main] INFO tc.postgres:alpine -- Container postgres:alpine started in PT38.594374044S
13:37:19.449 [main] INFO tc.postgres:alpine -- Container is started (JDBC URL: jdbc:postgresql://localhost:32773/test?loggerLevel=OFF)

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.1.3)

2023-09-13T13:37:19.624+07:00  INFO 18120 --- [           main] b.t.TestBelajarTestcontainersApplication : Starting TestBelajarTestcontainersApplication using Java 17.0.8 with PID 18120 (started by rizki in /home/rizki/Documents/project/belajar-project/spring-project/belajar-testcontainers)
2023-09-13T13:37:19.625+07:00  INFO 18120 --- [           main] b.t.TestBelajarTestcontainersApplication : No active profile set, falling back to 1 default profile: "default"
2023-09-13T13:37:20.112+07:00  INFO 18120 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFAULT mode.
2023-09-13T13:37:20.150+07:00  INFO 18120 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 34 ms. Found 1 JPA repository interfaces.
2023-09-13T13:37:20.515+07:00  INFO 18120 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 0 (http)
2023-09-13T13:37:20.527+07:00  INFO 18120 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-09-13T13:37:20.527+07:00  INFO 18120 --- [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.12]
2023-09-13T13:37:20.591+07:00  INFO 18120 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-09-13T13:37:20.593+07:00  INFO 18120 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 841 ms
2023-09-13T13:37:20.708+07:00  INFO 18120 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
2023-09-13T13:37:20.733+07:00  INFO 18120 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 6.2.7.Final
2023-09-13T13:37:20.735+07:00  INFO 18120 --- [           main] org.hibernate.cfg.Environment            : HHH000406: Using bytecode reflection optimizer
2023-09-13T13:37:20.811+07:00  INFO 18120 --- [           main] o.h.b.i.BytecodeProviderInitiator        : HHH000021: Bytecode provider name : bytebuddy
2023-09-13T13:37:20.882+07:00  INFO 18120 --- [           main] o.s.o.j.p.SpringPersistenceUnitInfo      : No LoadTimeWeaver setup: ignoring JPA class transformer
2023-09-13T13:37:20.890+07:00  INFO 18120 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2023-09-13T13:37:20.989+07:00  INFO 18120 --- [           main] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection org.postgresql.jdbc.PgConnection@5dd6f517
2023-09-13T13:37:20.990+07:00  INFO 18120 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2023-09-13T13:37:21.107+07:00  INFO 18120 --- [           main] o.h.b.i.BytecodeProviderInitiator        : HHH000021: Bytecode provider name : bytebuddy
2023-09-13T13:37:21.420+07:00  INFO 18120 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
Hibernate: 
    drop table if exists tb_product cascade
2023-09-13T13:37:21.430+07:00  WARN 18120 --- [           main] o.h.engine.jdbc.spi.SqlExceptionHelper   : SQL Warning Code: 0, SQLState: 00000
2023-09-13T13:37:21.430+07:00  WARN 18120 --- [           main] o.h.engine.jdbc.spi.SqlExceptionHelper   : table "tb_product" does not exist, skipping
Hibernate: 
    create table tb_product (
        price numeric(38,2),
        quantity integer,
        id uuid not null,
        name varchar(255),
        primary key (id)
    )
2023-09-13T13:37:21.439+07:00  INFO 18120 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2023-09-13T13:37:21.612+07:00  WARN 18120 --- [           main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2023-09-13T13:37:21.879+07:00  INFO 18120 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 36553 (http) with context path ''
2023-09-13T13:37:21.887+07:00  INFO 18120 --- [           main] b.t.TestBelajarTestcontainersApplication : Started TestBelajarTestcontainersApplication in 2.421 seconds (process running for 42.534)
OpenJDK 64-Bit Server VM warning: Sharing is only supported for boot loader classes because bootstrap classpath has been appended
Hibernate: 
    insert 
    into
        tb_product
        (name,price,quantity,id) 
    values
        (?,?,?,?)
2023-09-13T13:37:22.878+07:00  INFO 18120 --- [o-auto-1-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2023-09-13T13:37:22.878+07:00  INFO 18120 --- [o-auto-1-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2023-09-13T13:37:22.881+07:00  INFO 18120 --- [o-auto-1-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 2 ms
Hibernate: 
    select
        p1_0.id,
        p1_0.name,
        p1_0.price,
        p1_0.quantity 
    from
        tb_product p1_0 
    where
        p1_0.id=?
Hibernate: 
    insert 
    into
        tb_product
        (name,price,quantity,id) 
    values
        (?,?,?,?)
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 44.121 s - in com.belajar.testcontainers.TestBelajarTestcontainersApplication
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  46.919 s
[INFO] Finished at: 2023-09-13T13:37:23+07:00
[INFO] ------------------------------------------------------------------------
```

Dari log diatas, dapat dilihat bahwa test dijalankan secara sukses dan dapat dilihat terdapat log dimana maven melakukan pull docker image lalu menjalankan testcontainer image tersebut dan kemudian unit test dapat dijalankans secara lancar.

Sekian artikel mengenai Belajar testcontainers, untuk source code diatas dapat anda akses di [belajar testcontainers](https://github.com/RizkiMufrizal/Belajar-Testcontainers). Jika ada saran dan komentar silahkan isi dibawah dan terima kasih :).