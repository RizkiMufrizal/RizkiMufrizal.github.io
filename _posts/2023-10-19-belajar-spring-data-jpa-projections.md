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

## Apa Itu Projections ?

>>Projections Pada Spring Data JPA adalah salah satu fitur yang ditawarkan oleh spring yang memudahkan developer untuk membuat permodelan dengan tipe tertentu atau biasa nya lebih disebut dengan permodelan dengan custom DTO.

Biasa nya projections akan digunakan ketika developer membutuhkan field - field yang tidak tersedia pada class entity, misal nya seperti hasil dari sum(), avg() dan lain sebagai nya. Projections juga bisa digunakan untuk custom query seperti sub query, dimana hasil dari query tersebut tidak dapat di mapping ke class entity. Projections pada spring data jpa dapat di implementasikan dengan 3 metode yaitu interface projections, class projections (DTO) dan dynamic projections.

### Interface Projections