---
layout: post
title: Belajar CQRS
modified: 2023-02-01T21:08:21+07:00
categories:
description: belajar cqrs
tags: [CQRS, command, query, responsibility, segregation]
image:
    background: abstract-2.png
comments: true
share: true
date: 2023-02-01T21:08:21+07:00
---

## Apa Itu CQRS ?

>>CQRS adalah kepanjangan dari ommand Query Responsibility Segregation, dimana pattern ini dicetuskan oleh [Greg Young](https://cqrs.files.wordpress.com/2010/11/cqrs_documents.pdf). CQRS merupakah sebuah pattern yang menjelaskan bagaimana cara memisahkan antara proses menulis (command) dan membaca (query) pada sebuah aplikasi.

Jika kita melihat ke berbagai arsitektur microservice, masing - masing service dapat memiliki database nya tersendiri atau lebih sering disebut dengan [database per service](https://microservices.io/patterns/data/database-per-service.html). Pembagian database per service juga dibagi ke dalam beberapa jenis yaitu

1. Private-tables-per-service, yaitu semua service menggunakan database yang sama, akan tetapi setiap service hanya dapat mengakses tabel yang telah ditentukan.

2. Schema-per-service, yaitu semua service memiliki database masing-masing akan tetapi database tersebut berada pada satu node database server.

3. Database-server-per-service, yaitu semua service memiliki database masing-masing dan berada pada masing-masing node database server.

Berikut adalah contoh penggunaan database per service dengan menggunakan jenis Schema-per-service.

![database-per-service.png](../images/database-per-service.png)

Dari contoh microservice diatas, jika pengguna ingin mengambil data order history, dimana kita harus menampilkan data order, data customer dan juga data product maka kita perlu melakukan call api terlebih dahulu ke service yang dibutuhkan lalu mengabungkannya, hal ini membutuhkan waktu yang lumayan panjang untuk menunggu response dari masing - masing service terlebih jika service yang dipanggil lebih dari 1.

Untuk menghadapi permasalah tersebut, [Greg Young](https://cqrs.files.wordpress.com/2010/11/cqrs_documents.pdf) memberikan solusi berupa CQRS dimana proses untuk menulis (command) dan membaca (query) data dipisah sehingga service yang digunakan juga akan berbeda antara untuk pemrosesan data terkait create, update dan delete data. Misal seperti gambar arsitektur sebelum nya, maka untuk memunculkan data order history maka kita perlu membuat 1 service lagi khusus untuk menangani data order history tersebut.

Pada service yang bertugas sebagai query pada bagian CQRS juga mempunya database, dimana database pada service ini digunakan sebagai agregator untuk menyimpan data - data dari service yang diperlukan. Misalnya pada gambar arsitektur sebelum nya, maka kita membutuhkan table dan data yang berasal dari database customer dan product yang akan disimpan di database order juga. Perlu diingat bahwa database ini bisa berbeda antara service command dan service query.