---
layout: post
title: Blocking IO Dan Non Blocking IO
modified:
categories:
description: Blocking IO Dan Non Blocking IO
tags: [Blocking IO, Non Blocking IO, Thread, Thread Pool, Task Queue]
image:
  background: abstract-2.png
comments: true
share: true
date: 2021-02-01T20:15:28+07:00
---

Aplikasi - aplikasi yang telah berjalan pada zaman sekarang biasa nya dibuat dalam bentuk blocking. Contoh yang sangat sederhana adalah aplikasi yang dibangun dengan menggunakan spring webmvc, dimana aplikasi ini masih menggunakan konsep blocking. Sebelum penulis membahas mengenai blocking dan non blocking, maka penulis akan membahas terlebih dahulu mengenai thread dikarenakan blocking dan non blocking sangat berkaitan dengan thread.

## Apa Itu Thread ?

>>Thread adalah rangkain eksekusi dari sebuah aplikasi, biasa nya setiap aplikasi akan memiliki thread, minimal 1 thread. 

Contoh yang paling sederhana dapat dilihat dari sebuah aplikasi web. Setiap request yang masuk ke dalam sebuah aplikasi web, maka akan membentuk sebuah koneksi. Koneksi yang dibentuk karena sebuah request akan dijadikan 1 thread pada aplikasi web tersebut. Berikut merupakan sample thread jika diilustrasikan pada sebuah aplikasi web.

![sample-thread-web.png](../images/sample-thread-web.png)

Dari gambar diatas, dapat dilihat terdapat request yang masuk secara bersamaan dari pengguna 1 dan pengguna 2. Masing - masing pengguna akan dibuatkan sebuah thread atau istilahnya multithread, dimana masing - masing proses akan dijalankan secara independent sehingga tidak ada proses tunggu menunggu antara kedua request tersebut.

Mungkin ada yang bertanya - tanya, apa perbedaan nya antara multithread dan multicore ? dan apakah dengan komputer single core dapat menggunakan multithread ?. Secara arsitektur, 1 core mempunya beberapa thread, biasanya 1 core memiliki 2 thread. Berikut jika digambarkan arsitektur dari core dan thread.

![core-thread.png](../images/core-thread.png)

Jika anda menggunakan linux, kita dapat melakukan pengecekan core dan thread dengan perintah

{% highlight bash %}
lscpu
{% endhighlight %}

Kemudian lihat pada bagian `Thread(s) per core` yang menandakan jumlah thread per core dan `Core(s) per socket` yang menandakan jumlah core dari processor. berikut contohnya

{% highlight bash %}
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   43 bits physical, 48 bits virtual
CPU(s):                          16
On-line CPU(s) list:             0-15
Thread(s) per core:              2
Core(s) per socket:              8
Socket(s):                       1
NUMA node(s):                    1
Vendor ID:                       AuthenticAMD
CPU family:                      23
Model:                           8
Model name:                      AMD Ryzen 7 2700X Eight-Core Processor
{% endhighlight %}

Dari informasi diatas dapat dilihat bahwa cpu nya memiliki 8 core dan setiap core memiliki 2 thread sehingga total nya ada 16 thread.

### Apa Itu Thread pool Dan Task Queue ?

Setiap menjalankan 1 thread, biasa nya pada komputer dengan arsitektur x64 membutuhkan memory sebanyak 1MB. Bayangkan jika thread tidak di manage dengan baik maka dapat dipastikan resource sebuah komputer akan habis dan aplikasi tersebut tidak dapat berjalan.

>>Thread Pool merupakan sebuah management thread yang dapat digunakan untuk memanage thread dari sebuah aplikasi.

Dengan ada nya thread pool ini, kita dapat mendefinisikan berapa maksimal thread yang akan dibuat, minimal thread, idle dari setiap thread dan lain sebagainya. Jika thread pool nya penuh, bagaimana dengan thread baru selanjutnya ?. Thread baru selanjutnya akan dibuat di dalam sebuah antrian dengan nama task queue.

>>Task Queue berisi thread antrian jika jumlah thread pada thread pool telah penuh.

Task queue atau istilah lain nya yaitu backlog jika disisi http port. Task queue atau backlog juga dapat dibatasi, jika melebihi dari yang ditentukan maka task tersebut tidak akan masuk ke dalam sebuah task queue. Berikut merupakan gambar dari arsitektur thread pool dan task queue.

![thread-pool-task-queue.png](../images/thread-pool-task-queue.png)

## Apa Itu Blocking IO ?

>>Blocking IO adalah sebuah proses akan dijalankan jika proses sebelum nya telah selesai dijalankan atau istilah lain nya yaitu synchronous

Blocking IO diadopsi oleh hampir semua web server dan web framework seperti apache tomcat, jetty, dan lain sebagai nya. Blocking IO sendiri mengadopsi teknologi thread pool dan task queue sehingga proses dijalankan jika terdapat thread yang sedang idle atau tidak digunakan. Berikut contoh arsitektur blocking IO jika diimplementasikan ke dalam sebuah web server.

![blocking-io.png](../images/blocking-io.png)

Dari gambar diatas, 1 request yang datang dari client dianggap sebagai sebuah thread, dimana 1 thread ini juga mewakili dari sebuah connection. Backlog pada gambar diatas berfungsi untuk menampung request jika thread yang terdapat pada thread pool sedang penuh atau istilahnya request tersebut akan diantrikan di dalam backlog tersebut. Jika thread yang terdapat pada thread pool tersebut ada yang telah direlease atau dilepas maka request yang terdapat pada backlog akan dimasukkan ke dalam thread pool. Misal maksimal kapasitas thread pool kita setting di angka 500, dan backlog di setting di angka 200. Request yang datang dari client sebanyak 1000 request maka 500 request akan di process secara bersamaan, 200 request akan dimasukkan ke dalam antrian sedangkan 300 request lagi akan di reject karena melebihi kapasitas dari backlog yang telah ditentukan. Dari proses diatas, response server biasa nya terlihat sangat lambat dikarenakan adanya antrian dan harus menunggu proses yang sebelum nya. Contoh framework yang menggunakan blocking IO adalah spring webmvc, ruby on rails, laravel dan lain sebagain nya.

## Apa Itu Non Blocking IO ?

>>Non Blocking IO adalah sebuah proses akan dijalankan tanpa harus menunggu proses lain selesai atau istilah lain nya yaitu asynchronous.

Berbeda dengan blocking IO, proses pada non blocking IO dapat berjalan tanpa harus menunggu proses sebelum nya selesai sehingga non blocking io dapat berjalan pada single thread. Jika pada aplikasi web, semua request dapat ditampung tanpa ada proses thread pool dan task queue sehingga response server terlihat sangat cepat. Berikut contoh arsitektur Non blocking IO jika diimplementasikan ke dalam sebuah web server.

![non-blocking-io.png](../images/non-blocking-io.png)

Dari gambar diatas, semua request akan diterima oleh event loop atau istilahnya diterima oleh thread bos. Semua request tersebut langsung diteruskan ke worker, jika worker tersebut lambat maka dapat dipastikan response juga lambat. Pada proses non blocking ini, cepat atau lambat nya sebuah aplikasi bergantung dari cara melakukan pemrograman dari seorang developer.

## Contoh Implementasi Blocking IO Dan Non Blocking IO

Pada artikel ini, penulis akan memberikan contoh implementasi blocking IO dan non blocking IO dengan menggunakan node js. Bagi kamu yang belum tau apa itu node js, silahkan simak artikel [Instalasi Perlengkapan Coding Node JS](https://rizkimufrizal.github.io/instalasi-perlengkapan-coding-node-js/).

### Implementasi Blocking IO

Silkahkan buat sebuah file `data.txt` lalu isi dengan text berikut.

{% highlight text %}
Contoh Blocking IO dan Non Blocking IO dengan menggunakan node js
{% endhighlight %}

Lalu buat sebuah file `blocking-io.js` lalu isi dengan codingan berikut

{% highlight js %}
const fs = require('fs');

console.log('start');

const dataFromFile = fs.readFileSync('data.txt');
console.log(dataFromFile.toString());

console.log('End');
{% endhighlight %}

Kemudian jalankan dengan perintah `node blocking-io.js` maka akan muncul hasil

{% highlight bash %}
start
Contoh Blocking IO dan Non Blocking IO dengan menggunakan node js
End
{% endhighlight %}

Dari codingan diatas, penulis menggunakan module fs untuk membaca sebuah file. Dapat dilihat, pada codingan diatas penulis menggunakan fungsi readFileSync, dimana fungsi tersebut berfungsi untuk membaca file dengan metode sync atau blocking IO sehingga proses diatas berjalan secara sequence. Jika proses readFileSync belum selesai maka console.log('End') tidak akan dijalankan sampai proses readFileSync selesai.

### Implementasi Non Blocking IO

Silahkan buat sebuah file `non-blocking-io.js` lalu isi dengan codingan berikut

{% highlight js %}
const fs = require('fs');

console.log('start');

fs.readFile('data.txt', function (error, data) {
    if(error) console.log(error);
    console.log(data.toString());
});

console.log('End');
{% endhighlight %}

Kemudian jalankan dengan perintah `node non-blocking-io.js` maka akan muncul hasil

{% highlight bash %}
start
End
Contoh Blocking IO dan Non Blocking IO dengan menggunakan node js
{% endhighlight %}

Dari codingan diatas dapat dilihat perbedaannya yaitu pada non blocking IO, penulis menggunakan fungsi readFile dimana fungsi tersebut berfungsi untuk membaca file dengan metode async atau non blocking IO. Dikarenakan proses read file lama maka proses console.log('End') dijalankan terlebih dahulu pada implementasi non blocking IO. Pada non blocking IO, semua proses akan dijalankan tapi tidak seperti blocking IO dimana proses selanjutnya tidak berpengaruh jika proses sebelum nya masih belum selesai.