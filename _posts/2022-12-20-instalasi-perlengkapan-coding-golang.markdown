---
layout: post
title: Instalasi Perlengkapan Coding Go-Lang
modified: 2022-12-20T21:08:21+07:00
categories:
description: instalasi Go-Lang pada linux
tags: [instalasi Go-Lang, instalasi Go-Lang di ubuntu]
image:
    background: abstract-2.png
comments: true
share: true
date: 2022-12-20T21:08:21+07:00
---

Setalah sekian lama, akhirnya penulis mencoba menyempatkan kembali menulis sedikit artikel untuk bisa berbagi ke teman - teman. Pada artikel ini, penulis akan membahas mengenai salah satu bahasa pemrograman yang cukup populer yaitu Go-Lang.

## Apa itu Go-Lang ?

> > Go-Lang adalah kepanjangan dari go language, dimana go-lang ini merupakan bahasa pemrograman yang diciptakan oleh google bersama Ken Thompson, Robert Griesemer, dan Rob Pike pada tahun 2009. Go-Lang termasuk statically typed dan compiled programming dimana hasil akhir nya berupa binary.

## Instalasi Go-Lang

Melakukan instalasi Go-Lang sangat lah mudah, pada artikel ini penulis hanya akan membahas instalasi go-lang pada sistem operasi linux. Yang perlu teman - teman lakukan adalah

1. Download runtime golang di [golang](https://go.dev/dl/)
2. Lalu silahkan buka file environment dengan perintah

{% highlight bash %}
sudo gedit /etc/environment
{% endhighlight %}

Lalu tambahkan parameter berikut, silahkan sesuaikan dengan path folder Go-Lang runtime teman - teman

{% highlight bash %}
GO_HOME=/home/rizki/Tool/Runtime/go
{% endhighlight %}

Selanjutnya silahkan tambahkan pada bagian `PATH` menjadi seperti berikut

{% highlight bash %}
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/home/rizki/Tool/Runtime/go/bin"
{% endhighlight %}

Setelah selesai, silakan restart komputer teman - teman lalu jalankan perintah berikut untuk mengecek versi Go-Lang nya

{% highlight bash %}
go version
{% endhighlight %}

Jika berhasil maka akan muncul output seperti berikut

{% highlight bash %}
go version go1.19.4 linux/amd64
{% endhighlight %}

## Instalasi Editor Go-Lang

Untuk memulai melakukan coding pasti nya teman - teman memerlukan editor. Editor yang digunakan di Go-Lang ada beberapa yaitu

1. [GoLand](https://www.jetbrains.com/go/)
2. [Visual Studio Code](https://code.visualstudio.com/)

Penulis menggunakan 2 editor tersebut dan dua - dua nya sangat mudah digunakan. Jika teman - teman menggunakan visual studio code, penulis sarankan untuk melakukan instalasi plugin berikut untuk mempermudah dalam proses coding.

1. [go](https://marketplace.visualstudio.com/items?itemName=golang.Go)
2. [go test](https://marketplace.visualstudio.com/items?itemName=premparihar.gotestexplorer)
3. [prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
4. [go template](https://marketplace.visualstudio.com/items?itemName=casualjim.gotemplate)

Sekian tutorial kali ini dan selamat coding Go-Lang. Terima kasih :).
