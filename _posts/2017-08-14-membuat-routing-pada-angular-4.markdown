---
layout: post
title: Membuat Routing Pada Angular 4
modified:
categories:
description: Membuat Routing Pada Angular 4
tags: [angular 4, typescript, routing]
image:
  background: abstract-2.png
comments: true
share: true
date: 2017-08-14T20:15:28+07:00
---

Pada artikel [sebelumnya](https://rizkimufrizal.github.io/setup-project-angular-4/), penulis telah membahas mengenai bagaimana cara setup project angular 4. Pada artikel ini, penulis akan membahas mengenai bagaimana cara membuat routing pada angular 4.

## Apa Itu Routing ?

>>Routing adalah salah satu fitur angular dimana routing ini biasanya digunakan untuk navigasi sebuah web.

Dalam melakukan development sebuah web, biasanya arsitektur yang digunakan adalah membuat sebuah web dengan full fitur angular, sehingga untuk urusan routing dan navigasi juga akan dihandle oleh angular. Untuk mengakses data ke bagian server, angular dapat menggunakan fungsi http bawaan. Pada artikel ini, kita akan mencoba membuat 2 routing, akan tetapi sebelum membuat routing, kita akan melakukan konfigurasi bootstrap terlebih dahulu.

## Setup Bootstrap

Untuk menambah dependency library bootstrap silahkan jalankan perintah berikut.

{% highlight bash %}
npm install --save @ng-bootstrap/ng-bootstrap bootstrap@4.0.0-beta
{% endhighlight %}

pada artikel ini,kita akan menggunakan library [ng-bootstrap](https://ng-bootstrap.github.io). Setelah selesai melakukan instalasi dependency, selanjutnya kita akan mendaftarkan library bootstrap pada file `.angular-cli.json` yang berada pada root project. Silahkan buka file `.angular-cli.json` lalu ubah seperti berikut.

{% highlight json %}
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "project": {
    "name": "belajar-angular4"
  },
  "apps": [
    {
      "root": "src",
      "outDir": "dist",
      "assets": [
        "assets",
        "favicon.ico"
      ],
      "index": "index.html",
      "main": "main.ts",
      "polyfills": "polyfills.ts",
      "test": "test.ts",
      "tsconfig": "tsconfig.app.json",
      "testTsconfig": "tsconfig.spec.json",
      "prefix": "app",
      "styles": [
        "styles.css",
        "../node_modules/bootstrap/dist/css/bootstrap.min.css"
      ],
      "scripts": [],
      "environmentSource": "environments/environment.ts",
      "environments": {
        "dev": "environments/environment.ts",
        "prod": "environments/environment.prod.ts"
      }
    }
  ],
  "e2e": {
    "protractor": {
      "config": "./protractor.conf.js"
    }
  },
  "lint": [
    {
      "project": "src/tsconfig.app.json"
    },
    {
      "project": "src/tsconfig.spec.json"
    },
    {
      "project": "e2e/tsconfig.e2e.json"
    }
  ],
  "test": {
    "karma": {
      "config": "./karma.conf.js"
    }
  },
  "defaults": {
    "styleExt": "css",
    "component": {}
  }
}
{% endhighlight %}

Setelah selesai, silahkan buka `app.module.ts` di dalam folder `src/app`, lalu tambahkan main module dari angular bootstrap seperti berikut.

{% highlight typescript %}
import { NgbModule } from '@ng-bootstrap/ng-bootstrap';
{% endhighlight %}

Lalu kita memerlukan import pada bagian annotation `@NgModule`, maka hasilnya akan seperti berikut.

{% highlight typescript %}
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { NgbModule } from '@ng-bootstrap/ng-bootstrap';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    NgbModule.forRoot()
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
{% endhighlight %}

Langkah selanjutnya kita akan coba membuat 2 component, dimana nanti 2 component ini yang akan kita gunakan untuk kebutuhan routing. Disini penulis akan mencoba membuat 2 component yaitu `not-found` dan `dashboard`. Untuk membuat component silahkan jalankan perintah berikut.

{% highlight typescript %}
ng g component dashboard
ng g component not-found
{% endhighlight %}

Untuk melakukan setting routing, silahkan buat sebuah file di dalam folder `src/app` dengan nama `app.routing.module.ts`. Lalu silahkan masukkan codingan seperti berikut.

{% highlight typescript %}
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { DashboardComponent } from './dashboard/dashboard.component';
import { NotFoundComponent } from './not-found/not-found.component';

const routes: Routes = [
    { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
    { path: 'dashboard', component: DashboardComponent },
    { path: '**', component: NotFoundComponent }
];

@NgModule({
    imports: [RouterModule.forRoot(routes)],
    exports: [RouterModule]
})
export class AppRoutingModule { }

export const RoutedComponents = [
    DashboardComponent,
    NotFoundComponent
];
{% endhighlight %}

Nah kemudian silahkan buka file `app.module.ts`, silahkan ubah codingan menjadi seperti berikut.

{% highlight typescript %}
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { NgbModule } from '@ng-bootstrap/ng-bootstrap';

import { AppComponent } from './app.component';
import { AppRoutingModule, RoutedComponents } from './app.routing.module';

@NgModule({
  declarations: [
    AppComponent,
    RoutedComponents
  ],
  imports: [
    BrowserModule,
    FormsModule,
    NgbModule.forRoot(),
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
{% endhighlight %}

Nah dapat dilihat pada source code diatas, bahwa component yang tadi nya telah diregister telah dipindahkan ke file `app.routing.module.ts`. Langkah selanjutnya kita harus mendeklarasikan routing nya pada file `app.component.html`, silahkan ubah codingan nya menjadi seperti berikut.

{% highlight html %}
<nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
  <a class="navbar-brand" routerLink="/dashboard">Belajar Angular 4</a>
  <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse" aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
    <span class="navbar-toggler-icon"></span>
  </button>
  <div class="collapse navbar-collapse" id="navbarCollapse">
    <ul class="navbar-nav mr-auto">
      <li class="nav-item active">
        <a class="nav-link" routerLink="/dashboard">Home <span class="sr-only">(current)</span></a>
      </li>
    </ul>
  </div>
</nav>

<div class="container">
  <router-outlet></router-outlet>
</div>
{% endhighlight %}

Agar halaman nya tampil maka dibutuhkan css tambahan,silahkan buka file `index.html` di dalam folder `src` lalu silahkan ubah seperti berikut.

{% highlight html %}
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>BelajarAngular4</title>
  <style>
    body {
      min-height: 75rem;
      padding-top: 4.5rem;
    }
  </style>
  <base href="/">

  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <app-root></app-root>
</body>
</html>
{% endhighlight %}

Jika berhasil maka outputnya akan seperti berikut.

![Screen Shot 2017-08-13 at 11.24.36 AM.png](../images/Screen Shot 2017-08-13 at 11.24.36 AM.png)

Silahkan akses dengan url [http://localhost:4200/dashboard1](http://localhost:4200/dashboard1) maka akan muncul halaman `not-found` seperti berikut.

![Screen Shot 2017-08-13 at 11.24.42 AM.png](../images/Screen Shot 2017-08-13 at 11.24.42 AM.png)

Sekian artikel mengenai Membuat Routing Pada Angular 4, jika ada pertanyaan atau saran silahkan isi di kolom komentar dan Terima kasih :).
