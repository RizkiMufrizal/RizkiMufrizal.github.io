---
layout: post
title: Setup Project Kotlin
modified:
categories:
description: Setup Project Kotlin
tags: [Kotlin, Jetbrains, IntelliJ IDEA, gradle]
image:
  background: abstract-2.png
comments: true
share: true
date: 2017-06-5T20:15:28+07:00
---

Pada artikel [instalasi perlengkapan coding kotlin](https://rizkimufrizal.github.io/instalasi-perlengkapan-coding-kotlin/), penulis telah membahas sedikit mengenai bagaimana cara instalasi kotlin dengan sdkman. Oke, pada artikel ini, penulis akan membahas mengenai dasar - dasar dari bahasa pemrograman kotlin. Project yang akan kita buat adalah berbasis gradle, jika anda pengguna linux dan windows silahkan ikutin tutorial langsung dari websitenya [disini](https://gradle.org/install). Untuk penggunakan OSX, anda dapat melakukan instalasi dengan bantuan [Homebrew](https://brew.sh/) seperti berikut.

{% highlight bash %}
brew install gradle
{% endhighlight %}

Seperti biasa, penulis menggunakan IDE [IntelliJ IDEA](https://www.jetbrains.com/idea/), anda dapat menggunakan versi community maupun versi ultimate karena dua - dua nya telah mendukung bahasa pemrograman kotlin.

Silahkan buka IDE anda, lalu untuk membuat sebuah project silahkan pilih menu `create new project`, lalu pilih `gradle` dan cek list di bagian kotlin seperti gambar berikut.

![Screen Shot 2017-06-03 at 9.51.09 PM.png](../images/Screen Shot 2017-06-03 at 9.51.09 PM.png)

Lalu pilih next, di bagian groupid silahkan isikan `org.rizki.mufrizal.belajar.kotlin` dan pada bagian artifact nya isikan dengan `Belajar-Kotlin`, lalu klik next hingga akan muncul project location, silahkan sesuaikan dengan folder project anda dan klik finish. Berikut adalah project yang berhasil digenerate oleh [IntelliJ IDEA](https://www.jetbrains.com/idea/).

![Screen Shot 2017-06-03 at 9.58.37 PM.png](../images/Screen Shot 2017-06-03 at 9.58.37 PM.png)

Secara default, project belum memiliki package untuk class - class kotlin, silahkan akses project anda dengan menggunakan terminal, lalu jalankan perintah berikut.

{% highlight bash %}
mkdir -p src/main/kotlin/org/rizki/mufrizal/belajarKotlin
mkdir -p src/test/kotlin/org/rizki/mufrizal/belajarKotlin
{% endhighlight %}

Silahkan lihat kembali project anda yang ada di [IntelliJ IDEA](https://www.jetbrains.com/idea/), maka akan terbentuk 2 sub directory dari src yaitu `main` untuk source code nya, sedangkan `test` untuk keperluan testing. Langkah selanjutnya silahkan buka file `build.gradle` lalu ubah menjadi seperti berikut.

{% highlight gradle %}
group "org.rizki.mufrizal.belajar.kotlin"
version "1.0-SNAPSHOT"

buildscript {
    ext.kotlin_version = "1.1.2-4"

    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "org.junit.platform:junit-platform-gradle-plugin:1.0.0-M4"
    }
}

apply plugin: "kotlin"
apply plugin: "org.junit.platform.gradle.plugin"

junitPlatform {
    filters {
        engines {
            include "spek"
        }
    }
}

repositories {
    mavenCentral()
    maven { url "http://dl.bintray.com/jetbrains/spek" }
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-jre8:$kotlin_version"

    testCompile "com.winterbe:expekt:0.5.0"
    testCompile "org.jetbrains.kotlin:kotlin-reflect:$kotlin_version"
    testCompile("org.jetbrains.spek:spek-api:1.1.2") {
        exclude group: "org.jetbrains.kotlin"
    }
    testRuntime("org.jetbrains.spek:spek-junit-platform-engine:1.1.2") {
        exclude group: "org.junit.platform"
        exclude group: "org.jetbrains.kotlin"
    }
}
{% endhighlight %}

Pada konfigurasi diatas, terdapat beberapa tambahan yaitu penulis menambahkan library test untuk kotlin yang disupport langsung dari jetbrains yaitu [spekframework](http://spekframework.org/). Untuk fungsi assert, penulis menggunakan framework [expekt](https://github.com/winterbe/expekt).

Silahkan buat sebuah class `App` di dalam package `org.rizki.mufrizal.belajarKotlin`, silahkan sesuaikan dengan package anda, lalu masukkan codingan seperti berikut.

{% highlight kotlin %}
package org.rizki.mufrizal.belajarKotlin

import java.math.BigDecimal

/**
 * Created by rizkimufrizal on 6/3/17.
 */
class App {

    fun extensionMoney(extension: String, money: BigDecimal): String {
        return "$extension $money"
    }

    companion object {
        @JvmStatic
        fun main(args: Array<String>) {
            val money = App().extensionMoney(extension = "Rp", money = BigDecimal(5000))
            println("saya mempunyai uang sebesar $money")
        }
    }
}
{% endhighlight %}

Berikut adalah beberapa penjelasan dari codingan diatas :

* `fun` berfungsi untuk mendeklarasikan sebuah function / method pada kotlin. Jika anda tidak mendeklarasikan sebuah type data maka secara default method tersebut adalah berupa `Unit` atau `void`.
* `"$extension $money"` adalah contoh dari string interpolation, sehingga yang memiliki tanda `$` akan dieksekusi.
* `companion object` sebagai static jika anda menggunakan bahasa pemrograman java.
* `@JvmStatic` juga sebagai static, akan tetapi berbeda dengan companion object, dimana companion object hanya dapat diakses melalui instance, sedangkan jika menggunakan `@JvmStatic` maka anda dapat langsung memanggil suatu fungsi dari class nya langsung. Contoh penggunaan ini akan dibahas pada artikel selanjutnya.
* `val` berfungsi untuk mendeklarasikan variabel yang bersifat immutable / jika object nya telah dibuat maka isi dari vaeriabel tersebut tidak dapat dirubah, sedangkan `var` kebalikan dari `val` yaitu bersifat mutable.
* `money` adalah variabel dengan type data string. Mengapa type string ? karena return dari function `extensionMoney` adalah string. Tanpa mendeklarasikan type data, kotlin dapat dengan mudah menebak type data yang ada di dalam variabel `money`.

Untuk menjalankan codingan diatas, silahkan klik pada icon kotlin dibagian kiri, lalu pilih menu run seperti berikut.

![Screen Shot 2017-06-03 at 11.30.48 PM.png](../images/Screen Shot 2017-06-03 at 11.30.48 PM.png)

Oke, langkah selanjutnya kita akan melakukan test, silahkan buat sebuah file `AppTest` di dalam package test anda. Lalu masukkan codingan seperti berikut.

{% highlight kotlin %}
package org.rizki.mufrizal.belajarKotlin

import com.winterbe.expekt.should
import org.jetbrains.spek.api.Spek
import org.jetbrains.spek.api.dsl.given
import org.jetbrains.spek.api.dsl.it
import org.jetbrains.spek.api.dsl.on
import java.math.BigDecimal

/**
 * Created by rizkimufrizal on 6/3/17.
 */
class AppTest : Spek({
    given("a extension money") {
        val app = App()
        on("set extension and money") {
            val money = BigDecimal(7000)
            val extensionMoney = app.extensionMoney("Rp", money)
            it("should return extension and money") {
                extensionMoney.should.equal("Rp $money")
            }
        }
    }
})
{% endhighlight %}

Untuk menjalankan test diatas, silahkan akses project anda dengan terminal lalu jalankan perintah berikut.

{% highlight bash %}
gradle test
{% endhighlight %}

Jika berhasil maka akan muncul output seperti berikut.

{% highlight bash %}
:compileKotlin UP-TO-DATE
:compileJava NO-SOURCE
:copyMainKotlinClasses UP-TO-DATE
:processResources NO-SOURCE
:classes UP-TO-DATE
:compileTestKotlin
Using kotlin incremental compilation
:compileTestJava NO-SOURCE
:copyTestKotlinClasses UP-TO-DATE
:processTestResources NO-SOURCE
:testClasses UP-TO-DATE
:junitPlatformTest

Test run finished after 5066 ms
[         4 containers found      ]
[         0 containers skipped    ]
[         4 containers started    ]
[         0 containers aborted    ]
[         4 containers successful ]
[         0 containers failed     ]
[         1 tests found           ]
[         0 tests skipped         ]
[         1 tests started         ]
[         0 tests aborted         ]
[         1 tests successful      ]
[         0 tests failed          ]

:test SKIPPED

BUILD SUCCESSFUL

Total time: 9.272 secs
{% endhighlight %}

Sekian artikel mengenai setup project kotlin, jika ada saran dan komentar silahkan isi dibawah dan terima kasih :)