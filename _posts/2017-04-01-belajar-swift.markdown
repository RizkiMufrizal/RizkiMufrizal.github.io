---
layout: post
title: Belajar Swift
modified: 2017-06-3T20:15:28+07:00
categories:
description: Belajar Swift
tags: [Swift, iOS, xcode]
image:
  background: abstract-2.png
comments: true
share: true
date: 2017-04-01T20:15:28+07:00
---

Bermula dari project yang sedang saya kerjakan di kantor yaitu aplikasi iOS, pada kesempatan ini saya akan share sedikit mengenai dasar - dasar pada swift :D.

## Apa Itu Swift ?

>>Swift adalah bahasa pemrograman yang digunakan untuk membangun aplikasi untuk produk apple seperti iOS, OSX dan lain sebagainya.

Terdapat 2 bahasa pemrograman yang dapat kita gunakan untuk mengembangkan aplikasi untuk produk apple yaitu Objective-C dan Swift. Pada artikel ini, penulis hanya membahas mengenai bahasa pemrograman swift untuk versi 3, untuk migrasi versi swift 3 silahkan lihat di [migration swift 3](https://swift.org/migration-guide/){:target="_blank"}.

### Mengapa Menggunakan Swift ?

Mungkin beberapa developer swift pemula pasti akan bertanya, mengapa kita menggunakan swift ? mengapa tidak menggunakan Objective-C ?, Baiklah kita akan membahas satu - persatu.

Pada zaman sekarang, para developer core dari berbagai bahasa pemrograman akan membuatkan sebuah bahasa dimana bahasa ini lebih mudah dimengerti manusia, berbeda dengan developer apple, mereka membuat bahasa pemrograman swift ini sendiri lebih dekat dengan mesin layaknya bahasa C, C++ dan Objective-C sehingga akan meningkatkan performance dari aplikasi.

Swift sendiri dibangun dengan paradigma yang berbeda dengan Objective-C, dimana bahasa pemrograman swift lebih mudah dibaca jika anda beralirah ke bahasa pemrograman berorientasi object seperti java, javascript, ruby dan lain sebagainya. Jika anda dulu nya adalah developer bahasa C atau C++ maka anda akan lebih mudah membaca source code Objective-C. Jika anda adalah developer pemula, penulis sarankan untuk mempelajari bahasa pemrograman swift 3 karena Objective-C dan swift adalah bahasa pemrograman yang berbeda dan tidak ada kaitan dari kedua nya.

Jika anda adalah seorang developer android, terlebih jika anda telah terbiasa menggunakan bahasa pemrograman [kotlin](https://kotlinlang.org/){:target="_blank"} maka anda tidak akan terlalu susah jika migrasi dari android ke iOS. Bahkan anda akan memiliki learning curve yang lebih cepat.

### Latihan Swift

Pada artikel ini, penulis menggunakan [IBM Swift Sandbox](https://swift.sandbox.bluemix.net){:target="_blank"} karena kita hanya akan latihan cli dengan menggunakan bahasa pemrograman swift 3. [IBM Swift Sandbox](https://swift.sandbox.bluemix.net){:target="_blank"} adalah salah satu editor cli online untuk latihan bahasa pemrograman swift, disana terdapat 2 versi yaitu swift versi 2 dan versi 3. Pada artikel ini, penulis akan menggunakan swift 3. 

#### Latihan Variabel Pada Swift

Secara default, pada [IBM Swift Sandbox](https://swift.sandbox.bluemix.net){:target="_blank"} akan menampilkan source code seperti berikut.

{% highlight swift %}
print("Hello world!")
{% endhighlight %}

Source code diatas berfungsi untuk menampilkan sebuah tulisan ```Hello Word! :-)```. Tahap selanjutnya tulisankan code seperti berikut.

{% highlight swift %}
let nama: String = "rizki mufrizal"
print(nama)
{% endhighlight %}

Codingan diatas berfungsi untuk membuat sebuah variabel dengan tipe data string, lalu kita inisialisasi dengan sebuah string lalu kita tampilkan. Di dalam swift terdapat 2 cara untuk mendeklarasikan variabel yaitu dengan menggunakan let dan var, dimana let berfungsi sebagai constanta atau immutable sehingga hanya bisa diinisialisasi sekali saja, berbeda dengan var yang bersifat mutable sehingga dapat diubah - ubah. Silahkan ubah source code diatas menjadi seperti berikut.

{% highlight swift %}
let nama: String = "rizki mufrizal"
nama = "ganti aja"
print(nama)
{% endhighlight %}

maka akan muncul error seperti berikut.

{% highlight bash %}
ERROR at line 5, col 6: cannot assign to value: 'nama' is a 'let' constant
nama = "ganti aja"
~~~~ ^
NOTE at line 4, col 1: change 'let' to 'var' to make it mutable
let nama: String = "rizki mufrizal"
^~~
var
{% endhighlight %}

Lalu silahkan ubah seperti berikut.

{% highlight swift %}
var nama: String = "rizki mufrizal"
nama = "ganti aja"
print(nama)
{% endhighlight %}

maka hasil nya akan sukses dikarenakan kita ingin mengubah value yang ada di dalam variabel nama.

#### Latihan Function Pada Swift

Sama seperti bahasa lain, swift juga dapat menggunakan function / method. Function / method pada swift juga dapat mengembalikan nilai berdasarkan tipe data nya dan juga dapat berupa void atau tidak mengembalikan nilai.

Berikut adalah contoh jika mengambalikan nilai.

{% highlight swift %}
func perjumlahan(angka1: Int, angka2: Int) -> Int {
    return angka1 + angka2
}
print("hasilnya adalah : \(perjumlahan(angka1: 2, angka2: 3))")
{% endhighlight %}

Diatas adalah contoh jika kita mengembalikan nilai, dan pada source code diatas kita menggunakan fungsi ```\()``` dimana fungsi tersebut adalah fungsi untuk mengeksekusi sebuah function atau variabel yang berada di dalam sebuah string atau "" (kutip dua).

Berikut adalah contoh jika function / method yang tidak mengembalikan nilai.

{% highlight swift %}
func perjumlahan(angka1: Int, angka2: Int) {
    print("hasilnya adalah : \(angka1 + angka2)")
}
perjumlahan(angka1: 2, angka2: 3)
{% endhighlight %}

#### Latihan Array Pada Swift

Pada latihan ini, kita akan membuat sebuah struct / record. Apa itu struct / record ?

>>Struct / record adalah kumpulan data dengan type yang berbeda

Sekilas memang mirip dengan array, akan tetepi terdapat perbedaan dimana array mempunyai banyak data dengan type data yang sama sedangkan struct / record memiliki data yang banyak dan memiliki type data yang berbeda. Jika kita menggunakan java, maka struct / record sama dengan class domain / entity / pojo. Berikut adalah contoh penggunaan array struct pada swift 3.

{% highlight swift %}
struct Barang {
    var idBarang: String?
    var namaBarang: String?
    var jumlahBarang: Int?
}

var barangs = [Barang]()

barangs.append(Barang(idBarang: "B001", namaBarang: "Rinso", jumlahBarang: 10))

for b in barangs {
    print("id Barang     : \(b.idBarang)")
    print("Nama Barang   : \(b.namaBarang)")
    print("Jumlah Barang : \(b.jumlahBarang)")
}
{% endhighlight %}

Maka hasilnya akan seperti berikut.

{% highlight bash %}
id Barang     : Optional("B001")
Nama Barang   : Optional("Rinso")
Jumlah Barang : Optional(10)
{% endhighlight %}

Agar tulisan Optional hilang maka tambahkan ```!``` seperti berikut.

{% highlight swift %}
struct Barang {
    var idBarang: String?
    var namaBarang: String?
    var jumlahBarang: Int?
}

var barangs = [Barang]()

barangs.append(Barang(idBarang: "B001", namaBarang: "Rinso", jumlahBarang: 10))

for b in barangs {
    print("id Barang     : \(b.idBarang!)")
    print("Nama Barang   : \(b.namaBarang!)")
    print("Jumlah Barang : \(b.jumlahBarang!)")
}
{% endhighlight %}

Maka hasilnya akan seperti berikut.

{% highlight bash %}
id Barang     : B001
Nama Barang   : Rinso
Jumlah Barang : 10
{% endhighlight %}

Sekian artikel mengenai Belajar Swift, jika ada saran dan komentar silahkan isi dibawah dan terima kasih :)