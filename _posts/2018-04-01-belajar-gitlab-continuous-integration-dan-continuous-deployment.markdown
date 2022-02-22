---
layout: post
title: Belajar Gitlab Continuous Integration Dan Continuous Deployment
modified:
categories:
description: Belajar Gitlab Continuous Integration Dan Continuous Deployment
tags: [docker, deployment, heroku, java, spring, gitlab, continuous integration, continuous deployment]
image:
  background: abstract-2.png
comments: true
share: true
date: 2018-04-01T20:15:28+07:00
---

Pada artikel [sebelumnya](https://rizkimufrizal.github.io/belajar-deployment-docker-pada-heroku/), penulis telah menjelaskan bagaimana cara melakukan deployment docker ke heroku, akan tetapi cara tersebut bisa dibilang sangat melelahkan ketika kita harus memanage banyak service. Untuk dapat melakukan otomatisasi keseluruhan service maka kita akan menggunakan bantuan CI/CD yang ditawarkan oleh gitlab. Untuk penjelasan mengenai CI/CD, anda dapat membaca artikel [Belajar Melakukan Integrasi Jenkins Dan Gitlab Pada Docker](https://rizkimufrizal.github.io/belajar-melakukan-integrasi-jenkins-dan-gitlab-pada-docker/). Pada artikel ini, kita akan mencoba membuat CI/CD untuk kebutuhan deployment service yang terdapat pada artikel [Belajar Deployment Docker Pada Heroku](https://rizkimufrizal.github.io/belajar-deployment-docker-pada-heroku/).

## Arsitektur Deployment Dengan Gitlab CI/CD

Berikut adalah arsitektur yang akan kita gunakan untuk CI/CD pada gitlab.

![Arsitektur Gitlab CI_CD.png](../images/Arsitektur Gitlab CI_CD.png)

Berikut adalah penjelasan dari gambar diatas.

1. Developer akan melakukan pull code terlebih dahulu yang berasal dari gitlab
2. Jika telah selesai mengubah atau menambahkan code, maka developer melakukan push code ke gitlab
3. Jika terdapat perubahan, gitlab akan melakukan build source code lalu melakukan testing, biasanya trigger perubahan ini dapat dilakukan pada branch tertentu.
4. Jika proses build source code dan testing berhasil maka gitlab akan melakukan build docker image, lalu docker image ini akan di push ke docker hub.
5. Setelah selesai melakukan push ke docker hub, gitlab akan melakukan deployment ke heroku. Pada saat proses deployment, gitlab akan melakukan pull image terlebih dahulu dari docker hub, lalu image terebut nantinya akan di push ke registry heroku.
6. Jika proses push docker image ke registry heroku berhasil, maka heroku akan menjalankan image tersebut.

## Import Project Dari Github Ke Gitlab

Salah satu alasan menggunakan gitlab adalah gitlab memberikan banyak fitur terutama untuk penggunaan devops. Untuk melakukan import project dari github ke gitlab, silahkan lakukan langkah - langkah berikut.

1. Silahkan login pada website [gitlab](https://gitlab.com). Lalu silahkan buat sebuah project, lalu pilih tab import project seperti gambar berikut.

![Screen Shot 2018-03-03 at 7.20.05 PM](../images/Screen Shot 2018-03-03 at 7.20.05 PM.png)

2. Lalu pilih import dari github, maka akan muncul output seperti berikut.

![Screen Shot 2018-03-03 at 7.29.22 PM.png](../images/Screen Shot 2018-03-03 at 7.29.22 PM.png)

Lalu klik authorize sehingga gitlab dapat mengakses repo anda yang ada di github.

3. Lalu pilih repo seperti berikut.

![Screen Shot 2018-03-03 at 7.31.37 PM.png](../images/Screen Shot 2018-03-03 at 7.31.37 PM.png)

4. Setelah selesai, silahkan clone kembali repo `Heroku-Container` yang berasal dari gitlab seperti berikut

{% highlight bash %}
git clone git@gitlab.com:RizkiMufrizal/Heroku-Container.git
{% endhighlight %}

## Membuat Unit Test Pada Spring Boot

Silahkan buka class `HerokuContainerApplicationTests` yang terdapat di dalam package `org.rizki.mufrizal.heroku.container`. Kemudian silahkan ubah source code nya menjadi seperti berikut untuk kebutuhan testing.

{% highlight kotlin %}
package org.rizki.mufrizal.heroku.container

import com.fasterxml.jackson.databind.ObjectMapper
import org.hamcrest.Matchers
import org.junit.Test
import org.junit.runner.RunWith
import org.rizki.mufrizal.heroku.container.domain.Barang
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc
import org.springframework.boot.test.context.SpringBootTest
import org.springframework.http.MediaType
import org.springframework.test.context.TestPropertySource
import org.springframework.test.context.junit4.SpringRunner
import org.springframework.test.web.servlet.MockMvc
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders
import org.springframework.test.web.servlet.result.MockMvcResultHandlers
import org.springframework.test.web.servlet.result.MockMvcResultMatchers
import java.math.BigDecimal

@RunWith(SpringRunner::class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, classes = [HerokuContainerApplication::class])
@AutoConfigureMockMvc
@TestPropertySource(locations = ["classpath:application.properties"])
class HerokuContainerApplicationTests {

    @Autowired
    lateinit var mockMvc: MockMvc

    @Test
    @Throws(Exception::class)
    fun getBarangsTest() {
        mockMvc
                .perform(MockMvcRequestBuilders.get("/api/barang").contentType(MediaType.APPLICATION_JSON))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(MockMvcResultMatchers.status().isOk)
                .andExpect(MockMvcResultMatchers.jsonPath("$._embedded.barang", Matchers.hasSize<Int>(0)))
    }

    @Test
    @Throws(Exception::class)
    fun saveBarangTest() {
        val barang = Barang(namaBarang = "rinso", hargabarang = BigDecimal.valueOf(5000), jumlahBarang = 100)
        mockMvc
                .perform(MockMvcRequestBuilders.post("/api/barang").contentType(MediaType.APPLICATION_JSON).content(ObjectMapper().writeValueAsString(barang)).accept(MediaType.APPLICATION_JSON))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(MockMvcResultMatchers.status().isCreated)
    }

}
{% endhighlight %}

Lalu buat sebuah file `application.properties` di dalam folder `resources`, lalu tambahkan konfigurasinya seperti berikut.

{% highlight properties %}
spring.profiles=heroku

spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=postgres://postgresqldb:postgresqldb@postgres:5432/heroku_container_test
spring.datasource.poolName=SpringBootHikariCP
spring.datasource.maximumPoolSize=3
spring.datasource.minimumIdle=5
spring.datasource.maxLifetime=2000000
spring.datasource.connectionTimeout=30000
spring.datasource.idleTimeout=30000

spring.jpa.generate-ddl=true
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQL9Dialect

spring.hateoas.use-hal-as-default-json-media-type=true

endpoints.cors.allowed-methods = POST, GET, OPTIONS, DELETE, PUT
endpoints.cors.allowed-origins = '*'
endpoints.cors.exposed-headers = accept, authorization, x-requested-with, content-type
endpoints.cors.max-age = 3600

spring.jackson.serialization.indent-output=true

spring.data.rest.base-path=/api
{% endhighlight %}

Maka struktur project akan berubah menjadi seperti berikut.

![Screen Shot 2018-03-03 at 8.00.33 PM.png](../images/Screen Shot 2018-03-03 at 8.00.33 PM.png)

## Setup CI/CD Gitlab

Pada tahapan ini, kita akan mencoba setup CI/CD pada gitlab. Untuk membuat CI/CD pada gitlab sangatlah mudah yaitu dengan membuat sebuah file di dalam root project dengan nama `.gitlab-ci.yml`, lalu tambahkan source code berikut

{% highlight yaml %}
image: docker:latest

stages:
 - build
 - publish-image
 - deploy-heroku-container

build:
 stage: build
 image: gradle:alpine
 variables:
  POSTGRES_DB: heroku_container_test
  POSTGRES_USER: postgresqldb
  POSTGRES_PASSWORD: postgresqldb
 services:
   - postgres:alpine
 script:
  - gradle clean test build
 artifacts:
  paths:
    - build/libs/*.jar

publish-image:
 stage: publish-image
 variables:
  POSTGRES_DB: heroku_container_test
  POSTGRES_USER: postgresqldb
  POSTGRES_PASSWORD: postgresqldb
  DOCKER_DRIVER: overlay2
 services:
  - docker:dind
  - postgres:alpine
 before_script:
  - docker info
 script:
 - echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin
 - apk add --no-cache openjdk8
 - java -version
 - ./gradlew clean test build docker dockerPush

deploy-heroku-container:
 stage: deploy-heroku-container
 image: rizkimufrizal/docker-node
 variables:
  DOCKER_DRIVER: overlay2
 services:
  - docker:dind
 before_script:
  - docker info
 script:
   - npm install -g heroku-cli
   - cd heroku-docker
   - echo $HEROKU_API_KEY | docker login --username=$HEROKU_USERNAME --password-stdin registry.heroku.com
   - heroku container:push web --app heroku-docker
{% endhighlight %}

Pada konfigurasi diatas terdapat 3 proses yang akan dijalankan oleh gitlab yaitu

1. `build` yaitu proses test dan build project menjadi file jar, nantinya kita dapat mendownload file jar yang telah dibuild oleh gitlab.
2. `publish-image` yaitu proses test build dan juga sekaligus proses push docker image ke docker hub.
3. `deploy-heroku-container` yaitu proses deployment docker image ke heroku.

Setelah selesai, tahap selanjutnya adalah kita perlu melakukan konfigurasi `Variables` yang dibutuhkan. Berikut adalah variabel yang dibutuhkan untuk konfigurasi gitlab diatas.

* `DOCKER_USERNAME` yaitu username docker hub
* `DOCKER_PASSWORD` yaitu password docker hub
* `HEROKU_USERNAME` yaitu username heroku biasanya menggunakan email
* `HEROKU_API_KEY` yaity api key sebagai pengganti password heroku

Untuk melakukan konfigurasi variabel diatas, silahkan akses project di gitlab anda. Lalu pilih menu setting dan pilih menu CI / CD. Lalu expand `Secret variables` dan isikan konfigurasi nya seperti berikut.

![Screen Shot 2018-03-03 at 8.15.20 PM.png](../images/Screen Shot 2018-03-03 at 8.15.20 PM.png)

Untuk melihat API Key nya heroku, silahkan buka dashboard heroku, klik profile lalu pilih menu `account settings`, lalu scroll kebawah sehingga akan muncul menu API Key seperti berikut.

![Screen Shot 2018-03-03 at 8.17.01 PM.png](../images/Screen Shot 2018-03-03 at 8.17.01 PM.png)

Silahkan pilih menu `reveal` untuk melihat API Key tersebut.

## Mengecek Hasil Deployment

Setelah konfigurasi diatas selesai, silahkan commit lalu push source code anda, jika berhasil maka muncul proses pipeline yang sedang berjalan pada project gitlab anda seperti berikut.

![Screen Shot 2018-03-03 at 8.21.11 PM.png](../images/Screen Shot 2018-03-03 at 8.21.11 PM.png)

Lalu silahkan pilih menu CI / CD di menu project anda, nantinya akan muncul pipeline yang sedang berjalan seperti berikut.

![Screen Shot 2018-03-03 at 8.22.29 PM.png](../images/Screen Shot 2018-03-03 at 8.22.29 PM.png)

Jika ingin lebih detail, silahkan klik menu jobs, maka akan muncul seperti berikut.

![Screen Shot 2018-03-03 at 8.23.09 PM.png](../images/Screen Shot 2018-03-03 at 8.23.09 PM.png)

Pada gambar diatas dapat dilihat bahwa jobs yang pertama yaitu `build` telah berhasil dikerjakan dan anda dapat melakukan download artifact jar melalui menu berikut.

![Screen Shot 2018-03-03 at 8.24.29 PM.png](../images/Screen Shot 2018-03-03 at 8.24.29 PM.png)

Berikut adalah gambar jika pipeline nya berhasil dijalankan

![Screen Shot 2018-03-03 at 8.28.10 PM.png](../images/Screen Shot 2018-03-03 at 8.28.10 PM.png)

Dan berikut adalah gambar jika semua jobs berhasil dijalankan

![Screen Shot 2018-03-03 at 8.28.19 PM.png](../images/Screen Shot 2018-03-03 at 8.28.19 PM.png)

Bagi anda yang ingin melihat source code codingan, silahkan lihat di [Heroku-Container](https://gitlab.com/RizkiMufrizal/Heroku-Container). Sekian artikel mengenai Belajar Gitlab Continuous Integration Dan Continuous Deployment dan terima kasih :).