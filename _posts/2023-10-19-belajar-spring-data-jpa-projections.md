---
layout: post
title: Belajar Projections Pada Spring Data JPA
modified: 2023-02-01T21:08:21+07:00
categories:
description: belajar projections pada spring data JPA
tags: [projections, spring framework, spring data JPA, hibernate]
image:
    background: abstract-2.png
comments: true
share: true
date: 2023-10-18T21:08:21+07:00
---

# Apa Itu Projections ?

>>Projections Pada Spring Data JPA adalah salah satu fitur yang ditawarkan oleh spring yang memudahkan developer untuk membuat permodelan dengan tipe tertentu atau biasa nya lebih disebut dengan permodelan dengan custom DTO.

Biasa nya projections akan digunakan ketika developer membutuhkan field - field yang tidak tersedia pada class entity, misal nya seperti hasil dari sum(), avg() dan lain sebagai nya. Projections juga bisa digunakan untuk custom query seperti sub query, dimana hasil dari query tersebut tidak dapat di mapping ke class entity. Projections pada spring data jpa dapat di implementasikan dengan 3 metode yaitu interface projections, class projections (DTO) dan dynamic projections.

Contoh penggunaan projections, misal kita mempunyai sebuah class entity seperti berikut.

```java
public class Product {

    @Id
    private UUID id;

    private String name;

    private BigDecimal price;

    private Integer quantity;
}
```

Dari class entity diatas, misal kita ingin menampilkan nama dan harga product saja. Sebenarnya kita bisa menggunakan class DTO secara manual cuma dibutuhkan proses mapping terlebih dahulu, sehingga dibutuhkan mapping dari class entity ke class DTO. Tapi di case tertentu seperti process sum terhadap suatu column misal jika kita menggunakan entity diatas, maka kita ingin mengetahui berapa total price. Hasil dari sum tersebut tidak mempunyai attribute pada class entity diatas sehingga dibutuhkan proses projections.

## Setup Kebutuhan Belajar Projections

Sebelum memulai case per case projections, kita perlu melakukan setup project untuk kebutuhan belajar projections. Silahkan akses web [spring initializer](https://start.spring.io/), lalu silahkan setup seperti berikut.

![Screenshot from 2023-10-18 23-10-07.png](../images/Screenshot from 2023-10-18 23-10-07.png)

Lalu silahkan buka project tersebut dengan menggunakan editor kesayangan anda, lalu silahkan buat 1 class entity `Product` lalu masukkan codingan seperti berikut.

```java
package org.rizki.mufrizal.belajar.projections.domain;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import org.hibernate.annotations.UuidGenerator;

import java.io.Serializable;
import java.math.BigDecimal;
import java.util.UUID;

@Entity
@Table(name = "tb_product")
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
public class Product implements Serializable {

    @Id
    @UuidGenerator(style = UuidGenerator.Style.TIME)
    @GeneratedValue
    private UUID id;

    private String name;

    private BigDecimal price;

    private Integer quantity;
}
```

Lalu silahkan buat 1 class repository yaitu `ProductRepository` dan masukkan codingan berikut

```java
package org.rizki.mufrizal.belajar.projections.repository;

import org.rizki.mufrizal.belajar.projections.domain.Product;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.UUID;

public interface ProductRepository extends JpaRepository<Product, UUID> {
}
```

Untuk inisialisasi data nya, kita dapat menggunakan annotation `EventListener`, silahkan buka class `BelajarProjectionsApplication` lalu tambahkan code berikut

```java
package org.rizki.mufrizal.belajar.projections;

import org.rizki.mufrizal.belajar.projections.domain.Product;
import org.rizki.mufrizal.belajar.projections.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.event.EventListener;

import java.math.BigDecimal;

@SpringBootApplication
public class BelajarProjectionsApplication {

    @Autowired
    private ProductRepository productRepository;

    public static void main(String[] args) {
        SpringApplication.run(BelajarProjectionsApplication.class, args);
    }

    @EventListener
    public void onApplicationEvent(ApplicationReadyEvent event) {
        for (int i = 1; i <= 5; i++) {
            var product = new Product();
            product.setName("Air" + i);
            product.setPrice(BigDecimal.valueOf(1000));
            product.setQuantity(10);

            productRepository.save(product);
        }
    }
}
```

Kemudian untuk config koneksi ke database, silahkan buka file `application.properties` lalu ubah menjadi seperti berikut

```properties
spring.datasource.url=jdbc:h2:mem:belajar_projectios
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.hibernate.ddl-auto=create-drop
```

## Interface Projections

Interface projections merupakan salah satu metode projections yang sangat mudah di implementasikan. Kita cukup membuat sebuah class interface, dimana di dalam class tersebut terdapat method dari masing - masing property yang hendak kita akses.

Pada case pertama, kita akan menggunakan projection untuk mengambil field - field tertentu, misal pada tulisan ini penulis hanya ingin melakukan select untuk nama dan price product saja. Maka yang diperlukan adalah silahkan buat 1 class interface dengan nama `ProductHQL` lalu masukkan codingan berikut.

```java
package org.rizki.mufrizal.belajar.projections.domain.hql;

import java.math.BigDecimal;

public interface ProductHQL {
    String getName();

    BigDecimal getPrice();
}
```

Bisa dilihat dari codingan diatas, untuk dapat melakukan mapping dari hasil query HQL maka diperlukan membuat method getter, dimana kita cukup membuat method untuk field - field yang diperlukan saja. Cara implementasi sangat mudah yaitu cukup menambahka code berikut pada class `ProductRepository`

```java
package org.rizki.mufrizal.belajar.projections.repository;

import org.rizki.mufrizal.belajar.projections.domain.Product;
import org.rizki.mufrizal.belajar.projections.domain.hql.ProductHQL;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;
import java.util.UUID;

public interface ProductRepository extends JpaRepository<Product, UUID> {
    List<ProductHQL> findAllBy();
}
```

Disini dapat dilihat, kita akan mencoba melakukan query ke database tapi hasil nya ingin ditampung ke dalam class interface dimana class interface tersebut hanya berisi 2 field saja. Fungsi projection disini adalah melakukan mapping dari hasil HQL tersebut ke class interface yang memiliki 2 method.

Lalu untuk dapat melakukan test, Silahkan buat 1 class controller yaitu `ProductController` lalu isi code nya seperti berikut.

```java
package org.rizki.mufrizal.belajar.projections.controller;

import org.rizki.mufrizal.belajar.projections.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ProductController {

    @Autowired
    private ProductRepository productRepository;

    @GetMapping(value = "/api/product/interface")
    public ResponseEntity<?> productInterface() {
        return new ResponseEntity<>(productRepository.findAllBy(), HttpStatus.OK);
    }

}
```

Setelah selesai, silahkan jalankan command `mvn clean spring-boot:run` untuk menjalankan spring boot nya lalu kamu bisa akses ke browser dengan url `http://localhost:8080/api/product/interface` atau dapat menggunakan curl dengan command

```shell
curl http://localhost:8080/api/product/interface -v
```

Hasil nya berupa

```shell
*   Trying 127.0.0.1:8080...
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET /api/product/interface HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 
< Content-Type: application/json
< Transfer-Encoding: chunked
< Date: Thu, 19 Oct 2023 04:25:02 GMT
< 
* Connection #0 to host localhost left intact
[{"name":"Air1","price":1000.00},{"name":"Air2","price":1000.00},{"name":"Air3","price":1000.00},{"name":"Air4","price":1000.00},{"name":"Air5","price":1000.00}]
```

Kemudian klau disisi log spring boot nya, hibernate hanya melakukan select untuk 2 field saja, berikut hasil log sql nya.

```sql
select
    p1_0.name,
    p1_0.price
from
    tb_product p1_0
```

Pertanyaan selanjutnya adalah, apakah projections dengan interface ini dapat dipadukan dengan custom HQL/SQL ? jawaban nya adalah bisa. Silahkan tambahkan code berikut pada class `ProductRepository`

```java
package org.rizki.mufrizal.belajar.projections.repository;

import org.rizki.mufrizal.belajar.projections.domain.Product;
import org.rizki.mufrizal.belajar.projections.domain.hql.ProductHQL;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

import java.util.List;
import java.util.UUID;

public interface ProductRepository extends JpaRepository<Product, UUID> {
    List<ProductHQL> findAllBy();

    @Query("select p.name as name, p.price as price from Product p")
    List<ProductHQL> findAllCustomQuery();
}
```

Kemudian tambahkan code berikut untuk `ProductController` agar kita dapat melakukan test

```java
package org.rizki.mufrizal.belajar.projections.controller;

import org.rizki.mufrizal.belajar.projections.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ProductController {

    @Autowired
    private ProductRepository productRepository;

    @GetMapping(value = "/api/product/interface")
    public ResponseEntity<?> productInterface() {
        return new ResponseEntity<>(productRepository.findAllBy(), HttpStatus.OK);
    }

    @GetMapping(value = "/api/product/interface/query")
    public ResponseEntity<?> productInterfaceQuery() {
        return new ResponseEntity<>(productRepository.findAllCustomQuery(), HttpStatus.OK);
    }

}
```

Kemudian silahkan akses url ke `http://localhost:8080/api/product/interface/query` dengan browser atau curl maka hasil nya akan sama seperti sebelum nya.