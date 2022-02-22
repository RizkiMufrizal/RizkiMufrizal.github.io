---
layout: post
title: Belajar iOS Bagian 1
modified: 2017-06-3T20:15:28+07:00
categories:
description: Belajar iOS Bagian 1
tags: [Swift, iOS, Xcode, alamofire, RxSwift, ObjectMapper]
image:
  background: abstract-2.png
comments: true
share: true
date: 2017-05-01T20:15:28+07:00
---

Pada artikel [sebelumnya](https://rizkimufrizal.github.io/belajar-swift/){:target="_blank"} penulis telah membahas mengenai bahasa pemrograman swift versi 3. Pada artikel ini, penulis akan membahas bagaimana cara membuat aplikasi iOS sederhana :).

## Kebutuhan Project

Pada dasarnya, swift dapat dijalankan di linux dan OSX, akan tetapi ketika kita mulai ingin membangun sebuah aplikasi iOS maka kita membutuhkan sistem operasi OSX. Mengapa demikian ? karena untuk membuat aplikasi iOS membutuhkan IDE Xcode, mungkin ada yang bertanya, apakah kita dapat membuat aplikasi iOS tanpa Xcode ? ya bisa saja seperti menggunakan [AppCode](https://www.jetbrains.com/objc/){:target="_blank"} punya jetbrains, akan tetapi kita diharuskan semuanya untuk ngoding mulai dari logic hingga UI nya :(, berbeda dengan Xcode yang akan memberikan fitur UI layaknya android studio :D. Penulis menyarankan anda untuk menggunakan Xcode untuk development iOS, terdapat banyak fitur salah satunya adalah Xcode dapat memberitahukan kepada kita jika layout yang kita buat tidak responsive bahkan dia dapat mengetahui jika tata letak suatu komponent tidak sesuai dengan letaknya :). Wow... mungkin fitur Xcode bahkan bisa dibilang lebih lengkap dari android studio, oke berikut adalah kebutuhan project untuk development iOS.

* MacBook / Laptop dengan [Hackintosh](http://www.hackintosh.com/){:target="_blank"}, disarankan menggunakan OSX sierra
* Xcode terbaru, pada artikel ini penulis menggunakan Xcode versi 8.3.2
* secangkir kopi untuk bersantai :D

## Tahap - Tahap Development

Kira - kira kita akan membuat apa ya ? ya, pada artikel ini, kita akan membuat sebuah aplikasi untuk menampilkan lagu - lagu yang ada di [spotify](https://www.spotify.com){:target="_blank"} :D. Bagaimana cara nya kita dapat mendapatkan datanya ? :o, gampang kok, kita hanya perlu mengakses API yang telah disediakan oleh spotify [disini](https://developer.spotify.com/web-api/console/){:target="_blank"}, karena kita hanya akan menampilkan item tertentu maka kita akan menggunakan API search yang ada pada spotify yaitu API yang ada [disini](https://developer.spotify.com/web-api/console/get-search-item/){:target="_blank"}. Pada API spotify, kita tidak diharuskan untuk login terkecuali jika berhubungan dengan data user :). Berikut adalah tahapan development yang akan kita lakukan.

* Membuat Project iOS dengan Xcode
* Melakukan Instalasi Dependency Dengan [Cocoapods](https://cocoapods.org/){:target="_blank"}
* Membuat Model Response
* Membuat Service / Consume API
* Membuat View Dengan Multi StoryBoard Dan Navigation
* Membuat Controller
* Uji Coba Aplikasi

## Membuat Project iOS dengan Xcode

Hal yang pertama kita lakukan adalah membuat project dengan Xcode. Silahkan buka Xcode anda maka akan muncul tampilan seperti berikut.

![Screen Shot 2017-03-31 at 10.29.11 PM.png](../images/Screen Shot 2017-03-31 at 10.29.11 PM.png)

Lalu pilih create a new Xcode project maka akan muncul menu seperti berikut.

![Screen Shot 2017-03-31 at 10.31.17 PM.png](../images/Screen Shot 2017-03-31 at 10.31.17 PM.png)

Lalu pilih single view application lalu klik next dan muncul menu untuk pengisian deskripsi aplikasi, silahkan isi seperti berikut.

![Screen Shot 2017-03-31 at 10.33.39 PM.png](../images/Screen Shot 2017-03-31 at 10.33.39 PM.png)

Jika berhasil maka akan muncul menu project seperti berikut.

![Screen Shot 2017-03-31 at 10.34.24 PM.png](../images/Screen Shot 2017-03-31 at 10.34.24 PM.png)

## Melakukan Instalasi Dependency Dengan Cocoapods

Terkadang pada sebuah project yang kita buat, kita membutuhkan dependency library dari pihak lain misalnya seperti [Alamofire](https://github.com/Alamofire/Alamofire){:target="_blank"} untuk consume API dan lain sebagainya. Setiap bahasa pemrograman mempunya tool tersendiri, begitu pula dengan swift, di swift kita dapat menggunakan [Cocoapods](https://cocoapods.org/){:target="_blank"} sebagai dependency management karena [Cocoapods](https://cocoapods.org/){:target="_blank"} salah satu tool yang sangat banyak digunakan oleh kalangan developer iOS.

Untuk melakukan instalasi [Cocoapods](https://cocoapods.org/){:target="_blank"}, kita dapat menggunakan fungsi gem yang secara default telah tersedia di OSx, silahkan jalankan perintah berikut untuk melakukan instalasi cocoapods.

{% highlight bash %}
gem install cocoapods
{% endhighlight %}

Setelah selesai, silahkan close project Xcode anda, lalu akses folder project iOS anda dan jalankan perintah berikut untuk membuat konfigurasi cocoapods.

{% highlight bash %}
pod init
{% endhighlight %}

Maka akan terbentuk sebuah file yaitu `Podfile` langka selanjutnya silahkan buka file `Podfile` tersebut lalu silahkan ubah konfigurasi seperti berikut.

{% highlight ruby %}
# Uncomment the next line to define a global platform for your project
 platform :ios, '10.0'

target 'Belajar-iOS' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for Belajar-iOS
  pod 'RxSwift',    '~> 3.0'
  pod 'RxCocoa',    '~> 3.0'
  pod 'ObjectMapper', '~> 2.2'
  pod 'Alamofire', '~> 4.4'
  pod 'AlamofireObjectMapper', '~> 4.0'
  pod 'Kingfisher', '~> 3.0'
  pod 'SwiftOverlays', '~> 3.0.0'
  pod 'AudioPlayerManager'
  pod 'MarqueeLabel/Swift'

  target 'Belajar-iOSTests' do
    inherit! :search_paths
    # Pods for testing
  end

  target 'Belajar-iOSUITests' do
    inherit! :search_paths
    # Pods for testing
  end

end
{% endhighlight %}

Berikut adalah beberapa penjelasan dari konfigurasi diatas :

* platform : mendefiniskan versi iOS yang akan kita gunakan
* RxSwift : berfungsi sebagai manipulasi UI Events dan untuk menghandle response API
* RxCocoa : berfungsi sebagai library tambahan untuk mendukung RxSwift karena kita menggunakan Cocoapods
* ObjectMapper : berfungsi untuk melakukan mapping json
* Alamofire : berfungsi untuk mengakses API
* AlamofireObjectMapper : berfungsi untuk mengubah response dari alamofire menjadi object swift
* Kingfisher : berfungsi untuk cache gambar
* SwiftOverlays : berfungsi untuk animasi loading
* AudioPlayerManager : berfungsi untuk memutarkan lagu
* MarqueeLabel : berfungsi untuk membuat text marquee

Oke, selanjutnya silahkan jalankan perintah berikut untuk melakukan instalasi dependency diatas.

{% highlight bash %}
pod install
{% endhighlight %}

Silahkan buka folder project anda, jika berhasil maka akan muncul sebuah workspace seperti berikut.

![Screen Shot 2017-03-31 at 11.03.41 PM.png](../images/Screen Shot 2017-03-31 at 11.03.41 PM.png)

Lalu silahkan klik kanan pada `Belajar-iOS.xcworkspace` tersebut lalu open with Xcode maka tampilan project anda akan seperti berikut.

![Screen Shot 2017-03-31 at 11.06.02 PM.png](../images/Screen Shot 2017-03-31 at 11.06.02 PM.png)

## Membuat Model Response

Setelah selesai dengan dependency management, langkah selanjutnya kita akan membuat model response dimana sebenarnya model response ini adalah representasi dari response json yang berasal dari API. Berikut adalah contoh json yang dihasilkan dari API spotify.

{% highlight json %}
{
  "tracks": {
    "href": "https://api.spotify.com/v1/search?query=Coldplay&type=track&offset=0&limit=20",
    "items": [
      {
        "album": {
          "album_type": "single",
          "artists": [
            {
              "external_urls": {
                "spotify": "https://open.spotify.com/artist/69GGBxA162lTqCwzJG5jLp"
              },
              "href": "https://api.spotify.com/v1/artists/69GGBxA162lTqCwzJG5jLp",
              "id": "69GGBxA162lTqCwzJG5jLp",
              "name": "The Chainsmokers",
              "type": "artist",
              "uri": "spotify:artist:69GGBxA162lTqCwzJG5jLp"
            },
            {
              "external_urls": {
                "spotify": "https://open.spotify.com/artist/4gzpq5DPGxSnKTe4SA8HAU"
              },
              "href": "https://api.spotify.com/v1/artists/4gzpq5DPGxSnKTe4SA8HAU",
              "id": "4gzpq5DPGxSnKTe4SA8HAU",
              "name": "Coldplay",
              "type": "artist",
              "uri": "spotify:artist:4gzpq5DPGxSnKTe4SA8HAU"
            }
          ],
          "available_markets": [
            "AD",
            "AR",
            "AT",
            "AU",
            "BE",
            "BG",
            "BO",
            "BR",
            "CA",
            "CH",
            "CL",
            "CO",
            "CR",
            "CY",
            "CZ",
            "DE",
            "DK",
            "DO",
            "EC",
            "EE",
            "ES",
            "FI",
            "FR",
            "GB",
            "GR",
            "GT",
            "HK",
            "HN",
            "HU",
            "ID",
            "IE",
            "IS",
            "IT",
            "JP",
            "LI",
            "LT",
            "LU",
            "LV",
            "MC",
            "MT",
            "MX",
            "MY",
            "NI",
            "NL",
            "NO",
            "NZ",
            "PA",
            "PE",
            "PH",
            "PL",
            "PT",
            "PY",
            "SE",
            "SG",
            "SK",
            "SV",
            "TR",
            "TW",
            "US",
            "UY"
          ],
          "external_urls": {
            "spotify": "https://open.spotify.com/album/7IzpJkWQqgz1BTutQvSitX"
          },
          "href": "https://api.spotify.com/v1/albums/7IzpJkWQqgz1BTutQvSitX",
          "id": "7IzpJkWQqgz1BTutQvSitX",
          "images": [
            {
              "height": 640,
              "url": "https://i.scdn.co/image/a57625bbecee7b4e7fe932a31cd92319d2a32855",
              "width": 640
            },
            {
              "height": 300,
              "url": "https://i.scdn.co/image/4c45b4ad2a79f5a345e854180cec249731db1908",
              "width": 300
            },
            {
              "height": 64,
              "url": "https://i.scdn.co/image/cd7345b44b0e950735295029bfd96b5d174bead6",
              "width": 64
            }
          ],
          "name": "Something Just Like This",
          "type": "album",
          "uri": "spotify:album:7IzpJkWQqgz1BTutQvSitX"
        },
        "artists": [
          {
            "external_urls": {
              "spotify": "https://open.spotify.com/artist/69GGBxA162lTqCwzJG5jLp"
            },
            "href": "https://api.spotify.com/v1/artists/69GGBxA162lTqCwzJG5jLp",
            "id": "69GGBxA162lTqCwzJG5jLp",
            "name": "The Chainsmokers",
            "type": "artist",
            "uri": "spotify:artist:69GGBxA162lTqCwzJG5jLp"
          },
          {
            "external_urls": {
              "spotify": "https://open.spotify.com/artist/4gzpq5DPGxSnKTe4SA8HAU"
            },
            "href": "https://api.spotify.com/v1/artists/4gzpq5DPGxSnKTe4SA8HAU",
            "id": "4gzpq5DPGxSnKTe4SA8HAU",
            "name": "Coldplay",
            "type": "artist",
            "uri": "spotify:artist:4gzpq5DPGxSnKTe4SA8HAU"
          }
        ],
        "available_markets": [
          "AD",
          "AR",
          "AT",
          "AU",
          "BE",
          "BG",
          "BO",
          "BR",
          "CA",
          "CH",
          "CL",
          "CO",
          "CR",
          "CY",
          "CZ",
          "DE",
          "DK",
          "DO",
          "EC",
          "EE",
          "ES",
          "FI",
          "FR",
          "GB",
          "GR",
          "GT",
          "HK",
          "HN",
          "HU",
          "ID",
          "IE",
          "IS",
          "IT",
          "JP",
          "LI",
          "LT",
          "LU",
          "LV",
          "MC",
          "MT",
          "MX",
          "MY",
          "NI",
          "NL",
          "NO",
          "NZ",
          "PA",
          "PE",
          "PH",
          "PL",
          "PT",
          "PY",
          "SE",
          "SG",
          "SK",
          "SV",
          "TR",
          "TW",
          "US",
          "UY"
        ],
        "disc_number": 1,
        "duration_ms": 247626,
        "explicit": false,
        "external_ids": {
          "isrc": "USQX91700278"
        },
        "external_urls": {
          "spotify": "https://open.spotify.com/track/1dNIEtp7AY3oDAKCGg2XkH"
        },
        "href": "https://api.spotify.com/v1/tracks/1dNIEtp7AY3oDAKCGg2XkH",
        "id": "1dNIEtp7AY3oDAKCGg2XkH",
        "name": "Something Just Like This",
        "popularity": 97,
        "preview_url": "https://p.scdn.co/mp3-preview/499eefd42a24ec562c464bd7acfad7ed41eb9179?cid=null",
        "track_number": 1,
        "type": "track",
        "uri": "spotify:track:1dNIEtp7AY3oDAKCGg2XkH"
      }
    ],
    "limit": 20,
    "next": "https://api.spotify.com/v1/search?query=Coldplay&type=track&offset=20&limit=20",
    "offset": 0,
    "previous": null,
    "total": 4618
  }
}
{% endhighlight %}

Kita tidak akan menggunakan semua value diatas, kita hanya akan menampilkan track, nama artist, nama album, judul album, gambar album dan akan memutarkan contoh lagu nya. Silahkan klik kanan pada Folder `Belajar-iOS` yang berwarna kuning lalu pilih `New Group` dan rename menjadi `models`. Setelah selesai, silahkan klik kanan pada group model lalu pilih menu `New File`, silahkan pilih menu `Swift File`, kemudian untuk menetukan direktory, anda harus membuat folder terlebih dahulu sehingga group models akan direpresentasikan ke folder models, berikan nama `TrackResponse` pada file swift yang anda buat, kemudian masukkan codingan seperti berikut.

{% highlight swift %}
//
//  TrackResponse.swift
//  Belajar-iOS
//
//  Created by rizki mufrizal on 4/30/17.
//  Copyright © 2017 rizki mufrizal. All rights reserved.
//

import Foundation
import ObjectMapper

struct TrackResponse: Mappable {
    var tracks: Track?

    init?(map: Map) {

    }

    mutating func mapping(map: Map) {
        tracks <- map["tracks"]
    }
}

struct Track: Mappable {
    var limit: Int?
    var next: String?
    var offset: Int?
    var previous: String?
    var total: Int?
    var items: [Item]?

    init?(map: Map) {

    }

    mutating func mapping(map: Map) {
        limit <- map["limit"]
        next <- map["next"]
        offset <- map["offset"]
        previous <- map["previous"]
        total <- map["total"]
        items <- map["items"]
    }
}

struct Item: Mappable {
    var name: String?
    var previewUrl: String?
    var album: Album?
    var artists: [Artist]?

    init?(map: Map) {

    }

    mutating func mapping(map: Map) {
        name <- map["name"]
        previewUrl <- map["preview_url"]
        album <- map["album"]
        artists <- map["artists"]
    }
}

struct Artist: Mappable {
    var name: String?

    init?(map: Map) {

    }

    mutating func mapping(map: Map) {
        name <- map["name"]
    }
}

struct Album: Mappable {
    var name: String?
    var images: [Image]?

    init?(map: Map) {

    }

    mutating func mapping(map: Map) {
        name <- map["name"]
        images <- map["images"]
    }
}

struct Image: Mappable {
    var height: Int?
    var width: Int?
    var url: String?

    init?(map: Map) {

    }

    mutating func mapping(map: Map) {
        height <- map["height"]
        width <- map["width"]
        url <- map["url"]
    }
}
{% endhighlight %}

Akhirnya selesai juga untuk class response nya :D. Artikel selanjutnya akan membahas mengenai bagaimana cara membuat service, view dan controller nya :). Untuk source code diatas dapat anda akses di [Belajar-iOS](https://github.com/RizkiMufrizal/Belajar-iOS/tree/bagian-1){:target="_blank"}. Sekian artikel mengenai Belajar iOS bagian 1, jika ada saran dan komentar silahkan isi dibawah dan terima kasih :)
