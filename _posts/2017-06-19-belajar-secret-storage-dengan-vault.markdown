---
layout: post
title: Belajar Secret Storage Dengan Vault
modified:
categories:
description: Belajar Secret Storage Dengan Vault
tags: [Secret Storage, Vault, HashiCorp, API Key, environment]
image:
  background: abstract-2.png
comments: true
share: true
date: 2017-06-19T20:15:28+07:00
---

## Apa Itu Secret Storage ?

>>Secret Storage adalah suatu storage yang berfungsi untuk menyimpan data - data secret / rahasia misalnya seperti password, API Key, client id, client secret dan lain - lain.

Sebagai seorang developer, terutama backend developer, mereka biasanya akan membuat sebuah environment. Environment ini biasanya digunakan untuk menyimpan data - data penting misalnya seperti IP database, nama database, username dan juga password dari database. Secara tidak langsung, membuat environment seperti hal diatas sebenarnya merupakan hal yang mengerikan jika suatu waktu server yang kita gunakan dapat dibobol oleh pihak yang tidak berkepentingan sehingga file environment tersebut dapat dibaca. Bahaya lagi jika pihak tersebut dapat mengakses database dan mengunci data - data nya sehingga kita diharuskan membayar kepada mereka agar datanya dapat dikembalikan.

Untuk mengatasi hal diatas, kita harus menggunakan suatu database, dimana pada database tersebut kita dapat menyimpan nilai - nilai secret tersebut secara aman. Pada artikel ini, penulis akan membahas salah satu secret storage yang banyak digunakan dikalangan developer yaitu [Vault](https://www.vaultproject.io/).

## Apa Itu Vault ?

>>Vault adalah tool untuk management secret suatu nilai yang dikembangkan oleh [HashiCorp](https://www.hashicorp.com/).

Biasanya vault akan digunakan untuk menyimpan data - data yang bersifat secret / rahasia misalnya seperti username, password, nama database dan IP dari suatu database. Vault memiliki banyak kelebihan diantaranya adalah Semua data yang disimpan ke dalam vault bersifat encrypted sehingga tidak dapat dibaca dan untuk dapat mengakses suatu data dari vault wajib menggunakan token.

## Instalasi Vault

Untuk melakukan instalasi vault, silahkan download vault [disini](https://www.vaultproject.io/downloads.html). Setelah download maka anda akan mendapatkan sebuah file, karena penulis menggunakan OSX maka nama file nya adalah `vault_0.7.3_darwin_amd64.zip`. Silahkan extract disuatu folder, lalu agar vault terbaca di terminal maka kita harus melakukan export path pada `bash` atau `zsh`. Silahkan jalankan perintah berikut untuk membuka file `.zshrc`.

{% highlight bash %}
vim ~/.zshrc
{% endhighlight %}

Jika menggunakan `bash` maka anda dapat menggunakan perintah.

{% highlight bash %}
vim ~/.bash_profile
{% endhighlight %}

Kemudian masukkan codingan berikut pada baris yang paling atas.

{% highlight bash %}
export PATH=$PATH:/Users/rizkimufrizal/vault/
export VAULT_ADDR='http://127.0.0.1:8200'
{% endhighlight %}

Pada codingan diatas, kita melakukan export path untuk vault, silahkan sesuaikan path vault anda. Pada baris kedua, penulis mendeklarasikan `VAULT_ADDR`, biasanya konfigurasi ini digunakan untuk vault client yang akan berkomunikasi dengan vault server. Langkah selanjutnya, silahkan update `.zshrc` dengan perintah.

{% highlight bash %}
source .zshrc
{% endhighlight %}

atau jika anda menggunakan bash :

{% highlight bash %}
source .bash_profile
{% endhighlight %}

Lalu selanjutnya jalankan server vault, karena sekarang masih tahap development, maka cara menjalankan seperti berikut.

{% highlight bash %}
vault server -dev
{% endhighlight %}

Lalu untuk mengecek apakah server vault telah jalan atau tidak, silahkan jalankan perintah berikut.

{% highlight bash %}
vault status
{% endhighlight %}

Jika berhasil maka akan muncul output seperti berikut.

{% highlight bash %}
Sealed: false
Key Shares: 1
Key Threshold: 1
Unseal Progress: 0
Unseal Nonce:
Version: 0.7.3
Cluster Name: vault-cluster-dea4df0a
Cluster ID: 4a9ba1e3-2638-e2a0-3f03-c3afc6e3c93e

High-Availability Enabled: false
{% endhighlight %}

## Menulis dan Membaca Nilai Secret Pada Vault

Silahkan buka terminal anda, lalu jalankan perintah berikut.

{% highlight bash %}
vault write secret/belajar value=rizkimufrizal
{% endhighlight %}

Pada contoh diatas, kita menulis sebuah nilai yaitu `rizkimufrizal`, dimana nilai tersebut ditulis pada path/key `secret/belajar`. Jika anda memiliki banyak nilai, lebih baik anda menggunakan file `json` yang dapat diimport ke vault, silahkan buat file `json` dengan nama file `vault.json`, lalu masukkan codingan seperti berikut.

{% highlight json %}
{
    "spring": {
        "datasource": {
            "url": "jdbc:mariadb://127.0.0.1:3306/microservice_book?autoReconnect=true",
            "username": "root",
            "password": "admin"
        }
    }
}
{% endhighlight %}

Pada codingan json diatas, dapat kita lihat bahwa kita membuat banyak object, diantaranya ada object `spring`, `datasource` dan lain - lain, dimana pada codingan diatas kita menyimpan data - data `url`, `username` dan `password`. Untuk melakukan import ke vault, silahkan jalankan perintah berikut.

{% highlight bash %}
vault write secret/belajar @vault.json
{% endhighlight %}

Untuk membaca nilai dari diatas, silahkan jalankan perintah berikut.

{% highlight bash %}
vault read secret/belajar
{% endhighlight %}

Maka hasilnya akan seperti berikut.

{% highlight bash %}
Key             	Value
---             	-----
refresh_interval	768h0m0s
spring          	map[datasource:map[password:admin url:jdbc:mariadb://127.0.0.1:3306/microservice_book?autoReconnect=true username:root]]
{% endhighlight %}

Agar mempermudah membaca value nya, silahkan jalankan perintah berikut untuk menghasilkan output json.

{% highlight bash %}
vault read -format=json secret/belajar
{% endhighlight %}

dan hasilnya akan seperti berikut.

{% highlight json %}
{
    "request_id": "5f185fff-0ec5-3974-f7a8-568c4457bc1b",
    "lease_id": "",
    "lease_duration": 2764800,
    "renewable": false,
    "data": {
        "spring": {
            "datasource": {
                "password": "admin",
                "url": "jdbc:mariadb://127.0.0.1:3306/microservice_book?autoReconnect=true",
                "username": "root"
            }
        }
    },
    "warnings": null
}
{% endhighlight %}

## Bagaimana Cara Akses Melalui Aplikasi ?

Untuk mengakses value tersebut, kita bisa menggunakan fungsi token yang disediakan oleh vault, silahkan jalankan perintah berikut untuk membuat sebuah token.

{% highlight bash %}
vault token-create
{% endhighlight %}

Jika berhasil maka akan muncul output seperti berikut.

{% highlight bash %}
Key            	Value
---            	-----
token          	09cb9604-f9df-c3bc-6e0c-dc1d3bd3a1fc
token_accessor 	8a95c580-3fb0-6551-f0ba-e77516fdd906
token_duration 	0s
token_renewable	false
token_policies 	[root]
{% endhighlight %}

Biasanya token yang digunakan adalah `09cb9604-f9df-c3bc-6e0c-dc1d3bd3a1fc`. Setiap bahasa pemrograman memiliki library tersendiri untuk dapat mengakses vault, silahkan lihat list library tersebut [disini](https://www.vaultproject.io/api/libraries.html).

## Implementasi Akses Vault Dengan Spring Framework

Pada artikel ini, penulis akan melakukan implementasi dengan menggunakan [Spring Vault](http://projects.spring.io/spring-vault/). Silahkan buat sebuah project spring boot melalui [Spring Initializr](https://start.spring.io/), Silahkan isikan property menjadi seperti berikut.

![Screen Shot 2017-06-19 at 11.23.06 AM.png](../images/Screen Shot 2017-06-19 at 11.23.06 AM.png)

Silahkan download dan import ke IDE anda. Lalu ubah codingan pada file `build.gradle` menjadi seperti berikut.

{% highlight gradle %}
buildscript {
    ext {
        kotlinVersion = "1.1.2-5"
        springBootVersion = "1.5.4.RELEASE"
    }
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:${kotlinVersion}")
        classpath("org.jetbrains.kotlin:kotlin-allopen:${kotlinVersion}")
    }
}

apply plugin: "kotlin"
apply plugin: "kotlin-spring"
apply plugin: "eclipse"
apply plugin: "org.springframework.boot"

version = "0.0.1-SNAPSHOT"
sourceCompatibility = 1.8
compileKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
compileTestKotlin {
    kotlinOptions.jvmTarget = "1.8"
}

repositories {
    mavenCentral()
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter")
    compile("org.jetbrains.kotlin:kotlin-stdlib-jre8:${kotlinVersion}")
    compile("org.jetbrains.kotlin:kotlin-reflect:${kotlinVersion}")
    compile("org.springframework.vault:spring-vault-core:1.0.2.RELEASE")
    testCompile("com.winterbe:expekt:0.5.0")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
{% endhighlight %}

Setelah selesai, silahkan buka file `application.properties` yang ada di dalam folder resources, lalu masukkan konfigurasi vault seperti berikut.

{% highlight properties %}
vault.token = 09cb9604-f9df-c3bc-6e0c-dc1d3bd3a1fc
vault.url = http://127.0.0.1:8200
{% endhighlight %}

Selanjutnya silahkan buat sebuah package `configuration` lalu buat sebuah class kotlin didalam nya dengan nama `VaultConfiguration` lalu masukkan codingan seperti berikut.

{% highlight kotlin %}
package org.rizki.mufrizal.belajar.spring.vault.configuration

import org.springframework.context.annotation.Configuration
import org.springframework.context.annotation.PropertySource
import org.springframework.vault.authentication.ClientAuthentication
import org.springframework.vault.authentication.TokenAuthentication
import org.springframework.vault.client.VaultEndpoint
import org.springframework.vault.config.AbstractVaultConfiguration
import java.net.URI

/**
 * Created by rizkimufrizal on 6/19/17.
 */
@Configuration
@PropertySource("classpath:application.properties")
class VaultConfiguration : AbstractVaultConfiguration() {
    override fun clientAuthentication(): ClientAuthentication = TokenAuthentication(environment.getRequiredProperty("vault.token"))

    override fun vaultEndpoint(): VaultEndpoint = VaultEndpoint.from(URI(environment.getRequiredProperty("vault.url")))
}
{% endhighlight %}

Codingan diatas berfungsi sebagai konfigurasi dari vault, dimana kita melakukan set token terlebih dahulu agar dapat nilai secret nya dapat diakses. Selanjutnya silahkan buka main class lalu masukkan codingan seperti berikut.

{% highlight kotlin %}
package org.rizki.mufrizal.belajar.spring.vault

import org.slf4j.LoggerFactory
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.CommandLineRunner
import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.core.env.Environment
import org.springframework.vault.annotation.VaultPropertySource

@SpringBootApplication
@VaultPropertySource(value = "secret/belajar")
class BelajarSpringVaultApplication @Autowired constructor(val environment: Environment) : CommandLineRunner {
    private val logger = LoggerFactory.getLogger(BelajarSpringVaultApplication::class.java)

    override fun run(vararg args: String?) {
        logger.info("hasil secret dari url : ${environment.getRequiredProperty("spring.datasource.url")}")
        logger.info("hasil secret dari username : ${environment.getRequiredProperty("spring.datasource.username")}")
        logger.info("hasil secret dari password : ${environment.getRequiredProperty("spring.datasource.password")}")
    }
}

fun main(args: Array<String>) {
    SpringApplication.run(BelajarSpringVaultApplication::class.java, *args)
}
{% endhighlight %}

Berikut adalah penjelasan mengenai codingan diatas :

* `@VaultPropertySource` berfungsi untuk mengambil secret berdasarkan path / key, tadi kita telah menyimpan sebuah nilai secret pada `secret/belajar`.
* `environment.getRequiredProperty("spring.datasource.url")` berfungsi untuk mengambil nilai yang berasal dari key `spring.datasource.url`, jadi key yang diambil harus sesuai dengan key yang dibuat.

Oke, selanjutnya kita akan membuat sebuah test untuk memastikan bahwa environment yang dibaca adalah benar, silahkan buka file `BelajarSpringVaultApplicationTests` yang ada didalam package test, lalu ubah menjadi seperti berikut.

{% highlight kotlin %}
package org.rizki.mufrizal.belajar.spring.vault

import com.winterbe.expekt.should
import org.junit.Test
import org.junit.runner.RunWith
import org.slf4j.LoggerFactory
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.context.SpringBootTest
import org.springframework.core.env.Environment
import org.springframework.test.context.junit4.SpringRunner
import org.springframework.vault.annotation.VaultPropertySource

@RunWith(SpringRunner::class)
@SpringBootTest
@VaultPropertySource(value = "secret/belajar")
class BelajarSpringVaultApplicationTests {

    @Autowired
    lateinit var environment: Environment

    private val logger = LoggerFactory.getLogger(BelajarSpringVaultApplicationTests::class.java)

    @Test
    fun testUrl() {
        logger.info("begin test url")
        environment.getRequiredProperty("spring.datasource.url").should.equal("jdbc:mariadb://127.0.0.1:3306/microservice_book?autoReconnect=true")
        environment.getRequiredProperty("spring.datasource.url").should.not.`null`
        environment.getRequiredProperty("spring.datasource.url").should.not.`empty`
        logger.info("end test url")
    }

    @Test
    fun testUsername() {
        logger.info("begin test username")
        environment.getRequiredProperty("spring.datasource.username").should.equal("root")
        environment.getRequiredProperty("spring.datasource.username").should.not.`null`
        environment.getRequiredProperty("spring.datasource.username").should.not.`empty`
        logger.info("end test username")
    }

    @Test
    fun testPassword() {
        logger.info("begin test password")
        environment.getRequiredProperty("spring.datasource.password").should.equal("admin")
        environment.getRequiredProperty("spring.datasource.password").should.not.`null`
        environment.getRequiredProperty("spring.datasource.password").should.not.`empty`
        logger.info("end test password")
    }
}
{% endhighlight %}

Jika menggunakan terminal, silahkan jalankan dengan perintah berikut.

{% highlight bash %}
gradle test
{% endhighlight %}

Jika menggunakan Intellij IDEA, klik kanan pada class test, lalu pilih run seperti berikut.

![Screen Shot 2017-06-19 at 1.15.09 PM.png](../images/Screen Shot 2017-06-19 at 1.15.09 PM.png)

Dan berikut adalah hasil test nya.

![Screen Shot 2017-06-19 at 1.15.47 PM.png](../images/Screen Shot 2017-06-19 at 1.15.47 PM.png)

Sekian artikel mengenai Belajar Secret Storage Dengan Vault dan Terima kasih :). Untuk source code lengkap, penulis publish di [Belajar Secret Storage Dengan Vault](https://github.com/RizkiMufrizal/Belajar-SpringVault).
