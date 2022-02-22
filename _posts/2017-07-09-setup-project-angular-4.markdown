---
layout: post
title: Setup Project Angular 4
modified:
categories:
description: Setup Project Angular 4
tags: [angular 4, typescript]
image:
  background: abstract-2.png
comments: true
share: true
date: 2017-07-09T20:15:28+07:00
---

Pada artikel [sebelumnya](https://rizkimufrizal.github.io/belajar-angular-js/) telah membahas mengenai angular js. Yups angular js adalah salah satu framework yang dikembangkan oleh google. Akan tetapi sejak angular versi 2, google hanya memberikan nama dengan angular, sedangkan untuk versi 1 mereka menamakan nya dengan angular js. Jika Anda menggunakan angular js maka kita menggunakan bahasa pemrograman javascript, sedangkan jika kita menggunakan angular 2 atau 4 maka nantinya kita akan menggunakan typescript.

## Apa Itu TypeScrypt ?

>>TypeScrypt adalah bahasa pemrograman yang dikembangkan oleh microsoft, dimana TypeScrypt memiliki fitur strong-typing sehingga membuat developer lebih rapi dalam melakukan development sebuah aplikasi.

Jika Anda adalah seorang developer java, maka tidak perlu memiliki waktu lama untuk menguasai TypeScrypt. Sintak yang digunakan juga tidak jauh berbeda dengan javascript. Salah satu yang penulis sukai adalah terdapat autocomplete yang bisa kita gunakan jika kita menggunakan editor [visual studio code](https://code.visualstudio.com/) untuk melakukan development dengan TypeScrypt.

## Setup Project Angular 4

Pada artikel ini, penulis akan membuatkan sebuah artikel mengenai bagaimana setup project angular 4. Bagi Anda yang belum melakukan instalasi node js, silahkan lihat artikel nya di [Instalasi Perlengkapan Coding Node JS](https://rizkimufrizal.github.io/instalasi-perlengkapan-coding-node-js/). Untuk membuat project angular 4, kita dapat menggunakan bantuan [angular cli](https://cli.angular.io/). Untuk melakukan instalasi angular 4, silahkan jalankan perintah berikut.

{% highlight bash %}
npm install -g @angular/cli
{% endhighlight %}

Jika telah selesai, tahap selanjutnya kita akan membuat sebuah project angular 4 dengan perintah berikut.

{% highlight bash %}
ng new Belajar-Angular4
{% endhighlight %}

Jika berhasil maka project nya akan memiliki struktur folder seperti berikut.

![Screen Shot 2017-07-08 at 10.59.12 PM.png](../images/Screen Shot 2017-07-08 at 10.59.12 PM.png)

Untuk menjalankan projectnya, silahkan jalankan perintah berikut.

{% highlight bash %}
ng serve
{% endhighlight %}

Nantinya akan dilakukan compile dan dilakukan transpiler dari TypeScrypt ke javascript versi ES5. Jika telah selesai, silahkan akses url [http://localhost:4200](http://localhost:4200) di browser Anda.

## Membuat Hello Word Dengan Angular 4

Silahkan buka folder `app` yang terdapat di dalam folder `src`. Silahkan buka file `app.component.html` lalu ubah menjadi seperti berikut.

{% highlight html %}
{% raw %}
<!--The content below is only a placeholder and can be replaced.-->
<div style="text-align:center">
  <h1>
    Welcome to {{title}}!!
  </h1>
  <img width="300" src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4NCjwhLS0gR2VuZXJhdG9yOiBBZG9iZSBJbGx1c3RyYXRvciAxOS4xLjAsIFNWRyBFeHBvcnQgUGx1Zy1JbiAuIFNWRyBWZXJzaW9uOiA2LjAwIEJ1aWxkIDApICAtLT4NCjxzdmcgdmVyc2lvbj0iMS4xIiBpZD0iTGF5ZXJfMSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgeD0iMHB4IiB5PSIwcHgiDQoJIHZpZXdCb3g9IjAgMCAyNTAgMjUwIiBzdHlsZT0iZW5hYmxlLWJhY2tncm91bmQ6bmV3IDAgMCAyNTAgMjUwOyIgeG1sOnNwYWNlPSJwcmVzZXJ2ZSI+DQo8c3R5bGUgdHlwZT0idGV4dC9jc3MiPg0KCS5zdDB7ZmlsbDojREQwMDMxO30NCgkuc3Qxe2ZpbGw6I0MzMDAyRjt9DQoJLnN0MntmaWxsOiNGRkZGRkY7fQ0KPC9zdHlsZT4NCjxnPg0KCTxwb2x5Z29uIGNsYXNzPSJzdDAiIHBvaW50cz0iMTI1LDMwIDEyNSwzMCAxMjUsMzAgMzEuOSw2My4yIDQ2LjEsMTg2LjMgMTI1LDIzMCAxMjUsMjMwIDEyNSwyMzAgMjAzLjksMTg2LjMgMjE4LjEsNjMuMiAJIi8+DQoJPHBvbHlnb24gY2xhc3M9InN0MSIgcG9pbnRzPSIxMjUsMzAgMTI1LDUyLjIgMTI1LDUyLjEgMTI1LDE1My40IDEyNSwxNTMuNCAxMjUsMjMwIDEyNSwyMzAgMjAzLjksMTg2LjMgMjE4LjEsNjMuMiAxMjUsMzAgCSIvPg0KCTxwYXRoIGNsYXNzPSJzdDIiIGQ9Ik0xMjUsNTIuMUw2Ni44LDE4Mi42aDBoMjEuN2gwbDExLjctMjkuMmg0OS40bDExLjcsMjkuMmgwaDIxLjdoMEwxMjUsNTIuMUwxMjUsNTIuMUwxMjUsNTIuMUwxMjUsNTIuMQ0KCQlMMTI1LDUyLjF6IE0xNDIsMTM1LjRIMTA4bDE3LTQwLjlMMTQyLDEzNS40eiIvPg0KPC9nPg0KPC9zdmc+DQo=">
</div>

<input [(ngModel)]="nama" />

<h1>Hello {{nama}}</h1>
{% endraw %}
{% endhighlight %}

dari codingan diatas, dapat dilihat bahwa kita menggunakan directive `ngModel` dimana nantinya isi dari property `nama` akan dibinding ke {% raw %} <h1>Hello {{nama}}</h1> {% endraw %}.

Jika langsung dijalankan maka akan muncul error, error disini disebabkan kita belum melakukan import untuk module `FormsModule`, untuk melakukan module tersebut, silahkan buka file `app.module.ts` lalu ubah codingan nya menjadi seperti berikut.

{% highlight typescript %}
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
{% endhighlight %}

Setelah selesai, silahkan akses kembali [http://localhost:4200](http://localhost:4200) di browser anda, disana terdapat 1 inputan, silahkan isikan nama anda, jika berhasil maka akan muncul output seperti berikut.

![Screen Shot 2017-07-08 at 11.25.44 PM.png](../images/Screen Shot 2017-07-08 at 11.25.44 PM.png)

Sekian artikel mengenai Setup Project Angular 4 dan Terima kasih :)
