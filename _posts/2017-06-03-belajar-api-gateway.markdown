---
layout: post
title: Belajar API Gateway
modified: 2017-06-3T20:15:28+07:00
categories:
description: Belajar API Gateway
tags: [API Gateway, Authentication, Redis, OAuth2, Spring Framework, Spring Session, Spring OAuth2]
image:
  background: abstract-2.png
comments: true
share: true
date: 2017-06-3T20:15:28+07:00
---

# Apa Itu API Gateway ?

>>API Gateway merupakan gerbang dari beberapa API, bertugas sebagai management API, merge beberapa API, authentication API dan lain - lain.

Berbicara mengenai API gateway maka tidak terlepas dengan pembahasan microservice. Microservice itu sendiri sebenarnya adalah kumpulan dari beberapa service atau kumpulan dari beberapa API. API ini biasanya berbentuk REST API menggunakan protokol http. Microservice sendiri sebenarnya berasal dari monolithic yang telah dipecah. Monolithic adalah aplikasi yang sehari - hari kita kembangkan, dimana semua module terdapat di dalam 1 aplikasi yang sangat besar.

Client biasanya hanya mengakses 1 URL REST API sedangkan microservice memiliki banyak service REST API, untuk mengatasi masalah ini maka kita gunakan API gateway. Terdapat banyak contoh API gateway yang ada di pasaran contohnya [kong](https://getkong.org/), [tyk](https://tyk.io/), [API Umbrella](https://apiumbrella.io/) dan lain - lain.

## Bagaimana arsitektur yang pernah saya gunakan ?

Biasanya saya akan membuat microservice dengan stack spring framework :). Mengapa menggunakan spring framework ? ya alasan nya karena framework ini merupakan framework yang sudah memiliki banyak support, terutama dalam hal membuat REST API, authentication dengan OAuth2, session dan lain - lain. Berikut adalah arsitektur yang pernah saya gunakan untuk membuat microservice dan API gateway sederhana.

![API Gateway.svg](../images/API Gateway.svg)

Dari gambar diatas dapat dilihat bahwa aplikasi yang dibuat sangatlah besar meskipun hanya membuat service buku dan review buku. Pada gambar diatas terdapat :

* database redis : ini kita gunakan untuk menyimpan session
* database authentication : database untuk menyimpan data user, client credenetial dan lain - lain
* database book : database untuk menyimpan data - data katalog buku
* database review : database untuk menyimpan data - data review dari sebuah buku
* authorization server : adalah authorization server yang berfungsi sebagai management otorisasi setiap service REST API. Authorization yang saya gunakan adalah berbasis OAuth2, dimana antar service wajib melakukan authentication terlebih dahulu sebelum mengakses service yang lain, contohnya adalah ketika service book ingin mengakses service review maka service book diharuskan login terlebih dahulu dengan menggunakan authentication OAuth2 grant type client credentials. Tidak hanya antar service, API gateway yang akan mengakses suatu service juga wajib melakukan authentication terlebih dahulu, akan tetapi setiap service memiliki client id, client secret dan hak akses masing - masing sehingga memiliki keterbatasan dalam mengakses masing - masing service. Bagi anda yang masih bingung dengan OAuth2, silahkan baca artikel [Belajar OAuth2](https://rizkimufrizal.github.io/belajar-oauth2/).
* service book : REST API penyedia book
* service review : REST API penyedia review book
* API Gateway : yang bertugas untuk management API. Biasanya saya akan melakukan merge beberapa request API jika client membutuhkan banyak API. Contohnya jika kita membutuhkan API yang berisi data book beserta review nya, maka kita harus melakukan request sebanyak 2x yaitu melakukan request ke service book lalu request kembali ke service review. Biasanya model seperti ini akan membuat banyak latency di bagian client, nah untuk mengurangi hal tersebut maka kita cukup melakukan merge request API di bagian API gateway sehingga client hanya perlu melakukan request sekali ke API gateway, biasanya saya akan melakukan merge request API dengan menggunakan RxJava, contohnya adalah seperti berikut.

{% highlight kotlin %}
fun getBookDetails(bookId: Long): Observable<BookDetail> {
    return Observable.zip(
        catalogService.getBook(bookId),
        reviewService.getReviews(bookId),
        this::buildBookDetails
    )
}

private fun buildBookDetails(book: Book, reviews: Iterable<Review>): BookDetail {
    return BookDetail(
            id = book.id,
            title = book.title,
            description = book.description,
            reviews = reviews
    )
}
{% endhighlight %}

source code diatas ada [disini](https://github.com/RizkiMufrizal/Simple-API-Gateway/blob/master/Book-Gateway/src/main/kotlin/org/rizki/mufrizal/api/gateway/BookGateway/service/integration/IntegrationService.kt)

Mungkin ada yang bertanya - tanya, mengapa menggunakan redis ?, yups untuk menyimpan sebuah session yang berupa token, terdapat beberapa pendekatan yaitu kita dapat menyimpan token tersebut pada database atau dapat juga disimpan ke dalam redis. Jika menggunakan database maka database nya harus disharing antar service, ini mungkin pekerjaan yang lumayan merepotkan jika kita memiliki banyak service. Alternatif lain adalah menggunakan redis, dimana redis ini akan kita buat di 1 server sendiri.

### Bagaimana Menyimpan Session Pada Redis ?

Nah lagi - lagi spring menyediakan kebutuhan session, di spring ada namanya spring-session, dimana kita dapat menyimpan session langsung ke redis. Lalu bagaimana dengan konfigurasi OAuth2 ? apakah token nya dapat disimpan ke redis ? jawaban nya adalah sangatlah mungkin, cara nya adalah dengan menggunakan token store seperti berikut.

{% highlight kotlin %}
@Bean
fun tokenStore(): TokenStore {
    return RedisTokenStore(jedisConnectionFactory)
}

@Throws(Exception::class)
override fun configure(authorizationServerEndpointsConfigurer: AuthorizationServerEndpointsConfigurer?) {
    authorizationServerEndpointsConfigurer
        ?.accessTokenConverter(jwtAccessTokenConverter())
        ?.tokenStore(tokenStore())
        ?.authenticationManager(authenticationManager)
}
{% endhighlight %}

source code diatas ada [disini](https://github.com/RizkiMufrizal/Simple-API-Gateway/blob/master/Book-Authentication/src/main/kotlin/org/rizki/mufrizal/api/gateway/BookAuthentication/configuration/OAuth2Configuration.kt)

Sehingga apabila anda berhasil melakukan authentikasi, maka token anda akan disimpan ke redis, ingat bahwa token ini memiliki masa nya tersendiri sehingga hanya dapat digunakan untuk waktu tertentu saja.

>>Bagaimana service lain seperti service book dan service review tau bahwa token yang dikirim oleh client adalah token hasil yang digenerate oleh authorization server ?

Yups lagi - lagi kita menggunakan fungsi spring-session untuk menghandle masalah seperti ini, di bagian service - service kita hanya perlu mendeklarasikan resource id dan juga konfigurasi OAuth2 seperti berikut.

{% highlight kotlin %}
@Bean
fun tokenStore(): TokenStore {
    return RedisTokenStore(jedisConnectionFactory)
}

override fun configure(resourceServerSecurityConfigurer: ResourceServerSecurityConfigurer?) {
    resourceServerSecurityConfigurer
        ?.tokenStore(tokenStore())
        ?.resourceId(RESOURCE_ID)
}
{% endhighlight %}

source code diatas ada [disini](https://github.com/RizkiMufrizal/Simple-API-Gateway/blob/master/Book-Catalog/src/main/kotlin/org/rizki/mufrizal/api/gateway/BookCatalog/configuration/OAuth2Configuration.kt)

>>CATATAN : setiap service wajib memiliki 1 resource id, dimana resource id ini akan didaftarkan ke authorization server. Pencatatan resource id ini berfungsi sebagai memberikan batasan kepada client tertentu, sehingga client tertentu hanya boleh mengakses resource tertentu, contohnya adalah jika client seperti mobile atau web langsung mengakses service book maka tidak diperbolehkan dikarenakan client seperti mobile atau web tidak memiliki resource id dari service book, yang memiliki resource id service book adalah API gateway, sehingga client seperti mobile atau web hanya bisa mengakses melalui API gateway.

Untuk source code diatas dapat anda akses di [Simple API Gateway](https://github.com/RizkiMufrizal/Simple-API-Gateway){:target="_blank"}. Sekian artikel mengenai Belajar API Gateway, jika ada saran dan komentar silahkan isi dibawah dan terima kasih :)