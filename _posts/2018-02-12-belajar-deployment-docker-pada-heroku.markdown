---
layout: post
title: Belajar Deployment Docker Pada Heroku
modified: 2018-02-14T20:15:28+07:00
categories:
description: Belajar Deployment Docker Pada Heroku
tags: [docker, deployment, heroku, java, spring]
image:
  background: abstract-2.png
comments: true
share: true
date: 2018-02-12T20:15:28+07:00
---

Setelah sekian lama tidak aktif untuk menulis artikel, akhirnya hari ini penulis menyempatkan waktu untuk menulis artikel baru :D. Pada artikel ini, penulis akan membahas mengenai bagaimana teknik deployment docker pada heroku. Secara default, heroku sebenarnya mendukung deploment untuk docker akan tetapi hanya sedikit artikel dan tutorial mengenai deployment pada heroku.

Mungkin ada yang bertanya, mengapa kita menggunakan heroku untuk melakukan deployment docker ? sebenarnya untuk melakukan deployment docker dapat menggunakan provider lain misalnya [google cloud](https://cloud.google.com/), akan tetapi google cloud mewajibkan kita menggunakan kartu kredit untuk menggunakan versi free :(, untuk mengatasi hal tersebut, kita dapat menggunakan heroku :D. Proses deployment ini juga penulis gunakan untuk deployment aplikasi - aplikasi microservice yang penulis bangun untuk kebutuhan tesis :), jadi dapat dipastikan bahwa deployment docker pada heroku merupakan alternative yang tepat bagi developer yang ingin mencoba teknologi docker.

## Setup Project Spring Boot

Pada artikel ini, penulis akan membuat sebuah project spring boot dengan bahasa pemrograman kotlin yang nantinya akan di deploy ke container. Silahkan akses [start.spring.io](https://start.spring.io/), lalu isi seperti berikut.

![Screen Shot 2018-02-11 at 9.56.34 PM.png](../images/Screen Shot 2018-02-11 at 9.56.34 PM.png)

Silahkan download lalu extract project tersebut dan import ke dalam IDE Intellij IDEA. Lalu silahkan buka file `build.gradle` dan ubah menjadi seperti berikut.

{% highlight groovy %}
buildscript {
    ext {
        kotlinVersion = '1.2.21'
        springBootVersion = '1.5.10.RELEASE'
        gradleDockerPlugin = '0.13.0'
    }
    repositories {
        maven {
            url "https://plugins.gradle.org/m2/"
        }
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:${kotlinVersion}")
        classpath("org.jetbrains.kotlin:kotlin-allopen:${kotlinVersion}")
        classpath("gradle.plugin.com.palantir.gradle.docker:gradle-docker:${gradleDockerPlugin}")
    }
}

apply plugin: 'kotlin'
apply plugin: 'kotlin-spring'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'com.palantir.docker'

group = 'org.rizki.mufrizal.heroku.container'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

jar {
    baseName = 'heroku-container'
    version = '0.0.1'
}

docker {
    name "rizkimufrizal/${jar.baseName}"
    tags 'latest'
    files jar.archivePath
    buildArgs(['JAR_FILE': "${jar.archiveName}"])
}

compileKotlin {
    kotlinOptions {
        freeCompilerArgs = ["-Xjsr305=strict"]
        jvmTarget = "1.8"
    }
}
compileTestKotlin {
    kotlinOptions {
        freeCompilerArgs = ["-Xjsr305=strict"]
        jvmTarget = "1.8"
    }
}

repositories {
    mavenCentral()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('org.springframework.boot:spring-boot-starter-data-rest')
    compile('org.springframework.boot:spring-boot-starter-hateoas')
    compile('org.springframework.boot:spring-boot-starter-web')
    compile("org.jetbrains.kotlin:kotlin-stdlib-jdk8:${kotlinVersion}")
    compile("org.jetbrains.kotlin:kotlin-reflect:${kotlinVersion}")
    compile('com.zaxxer:HikariCP')
    runtime('org.postgresql:postgresql')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
{% endhighlight %}

Dapat dilihat pada bagian berikut.

{% highlight groovy %}
jar {
    baseName = 'heroku-container'
    version = '0.0.1'
}

docker {
    name "rizkimufrizal/${jar.baseName}"
    tags 'latest'
    files jar.archivePath
    buildArgs(['JAR_FILE': "${jar.archiveName}"])
}
{% endhighlight %}

merupakan konfigurasi plugin docker pada gradle yang nantinya kita gunakan untuk melakukan proses pembuatan image. `rizkimufrizal` adalah nama dari account [docker hub](https://hub.docker.com/). Langkah selanjutnya silahkan buka file `application.properties` di dalam folder `src/main/resources` lalu ubah konfigurasinya menjadi seperti berikut.

{% highlight properties %}
spring.profiles=heroku

spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.url=${DATABASE_URL}
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

`${DATABASE_URL}` merupakan variabel yang ada pada heroku, variabel ini nantinya kita gunakan untuk mengambil url database. Lalu silahkan buat sebuah package `configuration` di dalam package `org.rizki.mufrizal.heroku.container` lalu buatlah sebuah class kotlin `DataSourceConfiguration` untuk keperluan konfigurasi datasource, dan ubah codingan nya seperti berikut.

{% highlight kotlin %}
package org.rizki.mufrizal.heroku.container.configuration

import com.zaxxer.hikari.HikariConfig
import com.zaxxer.hikari.HikariDataSource
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.core.env.Environment
import java.net.URI
import javax.sql.DataSource

/**
 *
 * @Author Rizki Mufrizal <mufrizalrizki@gmail.com>
 * @Web <https://RizkiMufrizal.github.io>
 * @Since 11 February 2018
 * @Time 10:31 PM
 * @Project Heroku-Container
 * @Package org.rizki.mufrizal.heroku.container.configuration
 * @File DataSourceConfiguration
 *
 */

@Configuration
class DataSourceConfiguration @Autowired constructor(val environment: Environment) {

    @Bean(destroyMethod = "close")
    fun dataSource(): DataSource {
        val databaseUrl = URI(environment.getRequiredProperty("spring.datasource.url"))
        val dataSourceConfig = HikariConfig()
        dataSourceConfig.driverClassName = environment.getRequiredProperty("spring.datasource.driver-class-name")
        dataSourceConfig.jdbcUrl = "jdbc:postgresql://${databaseUrl.host}:${databaseUrl.port}${databaseUrl.path}"
        dataSourceConfig.username = databaseUrl.userInfo.split(":")[0]
        dataSourceConfig.password = databaseUrl.userInfo.split(":")[1]
        dataSourceConfig.maximumPoolSize = environment.getRequiredProperty("spring.datasource.maximumPoolSize").toInt()
        dataSourceConfig.minimumIdle = environment.getRequiredProperty("spring.datasource.minimumIdle").toInt()
        dataSourceConfig.connectionTimeout = environment.getRequiredProperty("spring.datasource.connectionTimeout").toLong()
        dataSourceConfig.idleTimeout = environment.getRequiredProperty("spring.datasource.idleTimeout").toLong()
        dataSourceConfig.addDataSourceProperty("poolName", environment.getRequiredProperty("spring.datasource.poolName"))
        dataSourceConfig.addDataSourceProperty("cachePrepStmts", true)
        dataSourceConfig.addDataSourceProperty("prepStmtCacheSize", 250)
        dataSourceConfig.addDataSourceProperty("prepStmtCacheSqlLimit", 2048)
        return HikariDataSource(dataSourceConfig)
    }
}
{% endhighlight %}

Langkah selanjutnya silahkan buat package `domain` pada package `org.rizki.mufrizal.heroku.container` lalu buat sebuah class kotlin dengan nama `Barang` lalu masukkan codingan seperti berikut.

{% highlight kotlin %}
package org.rizki.mufrizal.heroku.container.domain

import org.hibernate.annotations.GenericGenerator
import java.math.BigDecimal
import javax.persistence.Column
import javax.persistence.Entity
import javax.persistence.GeneratedValue
import javax.persistence.Id
import javax.persistence.Table

/**
 *
 * @Author Rizki Mufrizal <mufrizalrizki@gmail.com>
 * @Web <https://RizkiMufrizal.github.io>
 * @Since 11 February 2018
 * @Time 10:34 PM
 * @Project Heroku-Container
 * @Package org.rizki.mufrizal.heroku.container.domain
 * @File Barang
 *
 */

@Entity
@Table(name = "tb_barang")
data class Barang(
        @Id
        @GeneratedValue(generator = "uuid2")
        @GenericGenerator(name = "uuid2", strategy = "uuid2")
        @Column(name = "id_barang", length = 36)
        val idBarang: String? = null,

        @Column(name = "nama_barang", length = 50)
        val namaBarang: String? = null,

        @Column(name = "jumlah_barang")
        val jumlahBarang: Int? = null,

        @Column(name = "harga_barang")
        val hargabarang: BigDecimal? = null
)
{% endhighlight %}

Setelah selesai silahkan buat package `repository` di dalam package `org.rizki.mufrizal.heroku.container` lalu buat sebuah class kotlin dengan nama `BarangRepostory` lalu ubah codingan menjadi seperti berikut.

{% highlight kotlin %}
package org.rizki.mufrizal.heroku.container.repository

import org.rizki.mufrizal.heroku.container.domain.Barang
import org.springframework.data.repository.PagingAndSortingRepository
import org.springframework.data.rest.core.annotation.Description
import org.springframework.data.rest.core.annotation.RepositoryRestResource

/**
 *
 * @Author Rizki Mufrizal <mufrizalrizki@gmail.com>
 * @Web <https://RizkiMufrizal.github.io>
 * @Since 11 February 2018
 * @Time 10:38 PM
 * @Project Heroku-Container
 * @Package org.rizki.mufrizal.heroku.container.repository
 * @File BarangRepostory
 *
 */

@RepositoryRestResource(collectionResourceRel = "barang", path = "barang", collectionResourceDescription = Description("API Barang"))
interface BarangRepostory : PagingAndSortingRepository<Barang, String>
{% endhighlight %}

Lalu ubah main class `HerokuContainerApplication` menjadi seperti berikut.

{% highlight kotlin %}
package org.rizki.mufrizal.heroku.container

import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.hateoas.config.EnableHypermediaSupport

/**
 *
 * @Author Rizki Mufrizal <mufrizalrizki@gmail.com>
 * @Web <https://RizkiMufrizal.github.io>
 * @Since 11 February 2018
 * @Time 10:31 PM
 * @Project Heroku-Container
 * @Package org.rizki.mufrizal.heroku.container
 * @File HerokuContainerApplication
 *
 */

@SpringBootApplication
@EnableHypermediaSupport(type = [EnableHypermediaSupport.HypermediaType.HAL])
class HerokuContainerApplication

fun main(args: Array<String>) {
    SpringApplication.run(HerokuContainerApplication::class.java, *args)
}
{% endhighlight %}

Langkah selanjutnya silahkan buat file `Dockerfile` pada root project lalu tambahkan codingan berikut.

{% highlight bash %}
FROM openjdk:8-jdk-alpine
ARG JAR_FILE
ADD ${JAR_FILE} app.jar
RUN sh -c 'touch /app.jar'
{% endhighlight %}

Setelah selesai, selanjutnya kita akan membuat image berdasarkan project tersebut, silahkan jalankan command berikut untuk login ke docker hub.

{% highlight bash %}
docker login
{% endhighlight %}

Setelah selesai login, lalu jalankan perintah gradle berikut.

{% highlight bash %}
gradle clean build docker dockerPush -x test
{% endhighlight %}

Jika telah selesai, maka anda dapat melihat image anda telah tersedia di docker hub seperti berikut.

![Screen Shot 2018-02-11 at 11.00.13 PM.png](../images/Screen Shot 2018-02-11 at 11.00.13 PM.png)

## Deploy Docker Container Ke Heroku

Silahkan buat sebuah folder di dalam root project dengan nama `heroku-docker` lalu buat sebuah file `Dockerfile` di dalamnya dan tambahkan konfigurasi docker seperti berikut.

{% highlight bash %}
FROM rizkimufrizal/heroku-container
CMD java -Xmx300m -Xss512k -Dspring.profiles.active=heroku -jar /app.jar --server.port=$PORT
{% endhighlight %}

Container yang ada pada heroku wajib menggunakan perintah `CMD` untuk menjalankan service nya, dikarenakan ini adalah project java maka penulis menggunakan command java untuk menjalankan aplikasi web tersebut. `-Xmx300m -Xss512k` merupakan perintah untuk membatasi penggunaan memory dikarenakan memory untuk heroku versi gratis dibatasi hanya sampai [512 MB](https://devcenter.heroku.com/articles/limits).

Silahkan login pada web heroku di `https://dashboard.heroku.com`, lalu buat sebuah aplikasi dengan nama `heroku-docker`, untuk nama silahkan gunakan nama yang anda inginkan. Setelah selesai, silahkan masuk ke dalam detail aplikasi tersebut lalu pilih menu `resources`, lalu pada bagian add-ons silahkan ketika `postgres` maka akan muncul seperti berikut.

![Screen Shot 2018-02-11 at 11.09.14 PM.png](../images/Screen Shot 2018-02-11 at 11.09.14 PM.png)

Nantinya akan muncul seperti gambar berikut dan pilih `provision`

![Screen Shot 2018-02-11 at 11.09.23 PM.png](../images/Screen Shot 2018-02-11 at 11.09.23 PM.png)

Nantinya akan muncul seperti berikut.

![Screen Shot 2018-02-11 at 11.09.29 PM.png](../images/Screen Shot 2018-02-11 at 11.09.29 PM.png)

Lalu silahkan akses folder `heroku-docker` melalui terminal lalu jalankan perintah berikut untuk melakukan login ke heroku.

{% highlight bash %}
heroku login
{% endhighlight %}

silahkan isikan email dan password anda, lalu login ke registry docker heroku dengan perintah

{% highlight bash %}
heroku container:login
{% endhighlight %}

Kemudian untuk melakukan deploy ke heroku, silakan jalankan perintah berikut. Perintah berikut menggunakan `heroku-cli`, silahkan lihat cara instalasi `heroku-cli` di [heroku-cli](https://devcenter.heroku.com/articles/heroku-cli).

{% highlight bash %}
heroku container:push web --app heroku-docker
{% endhighlight %}

`heroku-docker` silahkan gunakan nama aplikasi yang anda buat di heroku. Jika berhasil maka akan muncul output seperti berikut pada terminal anda.

![Screen Shot 2018-02-11 at 11.22.15 PM.png](../images/Screen Shot 2018-02-11 at 11.22.15 PM.png)

## Test REST API Spring Boot

Untuk melakukan test terhadap REST API yang telah kita bangun, silahkan jalankan command berikut untuk melakukan proses save data.

{% highlight bash %}
curl -X POST -H "Content-type: application/json" -d '{
  "namaBarang": "rinso",
  "jumlahBarang": 100,
  "hargabarang": 5000
}' 'https://heroku-docker.herokuapp.com/api/barang'
{% endhighlight %}

Lalu silahkan akses data dengan command berikut

{% highlight bash %}
curl -X GET https://heroku-docker.herokuapp.com/api/barang
{% endhighlight %}

Untuk mematikan container pada heroku, anda dapat melakukan scale down menjadi 0 dengan perintah

{% highlight bash %}
heroku ps:scale web=0 --app heroku-docker
{% endhighlight %}

Bagi anda yang ingin melihat source code codingan, silahkan lihat di [Heroku-Container](https://github.com/RizkiMufrizal/Heroku-Container). Sekian artikel mengenai Belajar Deployment Docker Pada Heroku dan terima kasih :).