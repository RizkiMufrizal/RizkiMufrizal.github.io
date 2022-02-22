---
layout: post
title: Belajar Melakukan Integrasi Jenkins Dan Gitlab Pada Docker
modified:
categories:
description: Belajar Melakukan Integrasi Jenkins Dan Gitlab Pada Docker
tags: [docker, gitlab, jenkins, heroku, Continuous Integration, Continuous Delivery, docker compose, WebHook]
image:
  background: abstract-2.png
comments: true
share: true
date: 2017-08-01T20:15:28+07:00
---

Setelah melewati proses development aplikasi, maka tahap berikutnya adalah melakukan deployment aplikasi ke server yang dituju. Sebelum melakukan deployment, maka aplikasi wajib dilakukan testing terlebih dahulu. Proses seperti testing, building dan deployment sebenarnya dapat dijalankan secara otomatis dengan menggunakan tool automated deployment misalnya seperti :

* [Jenkins](https://jenkins.io/)
* [Travis CI](https://travis-ci.org/)
* [CodeShip](https://codeship.com/)
* [TeamCity](https://www.jetbrains.com/teamcity/)

Untuk melakukan integrasi dari satu proses dengan proses lain misalnya seperti testing kemudian menjalankan building lalu mendeploy, dalam hal ini sering disebut dengan Continuous Integration. Di dalam proses Continuous Integration, terdapat 2 jenis yaitu :

1. Continuous Deployment adalah proses deployment yang dijalankan secara otomatis mulai dari testing, building hingga deployment ke server production
2. Continuous Delivery adalah proses deployment yang dijalankan secara otomatis mulai dari testing, building, akan tetapi untuk proses deployment akan dilakukan secara manual. Biasanya yang melakukan deployment ke server production adalah manager IT yang tertinggi untuk memastikan aplikasi berjalan lancar.

Pada artikel ini, penulis akan membahas bagaimana cara melakukan integrasi gitlab (git server) dengan jenkins. Source code yang telah di push ke gitlab nantinya akan diclone di jenkins server lalu dilakukan testing, jika sukses maka source code akan di deploy ke server production, dalam artikel ini penulis akan mencoba melakukan deployment otomatis ke heroku. Berikut adalah arsitektur yang akan penulis gunakan untuk membuat Continuous Deployment dengan menggunakan docker, gitlab dan jenkins.

![Flow Gitlab Jenkins.svg](../images/Flow Gitlab Jenkins.svg)

## Setup Docker Compose

Bagi anda yang belum mengerti bagaimana cara menggunakan docker, silahkan baca di artikel [Belajar Docker](https://rizkimufrizal.github.io/belajar-docker/) dan [Belajar Load Balancing Dengan HAProxy, Docker Dan Spring Boot](https://rizkimufrizal.github.io/belajar-load-balancing-dengan-haproxy-docker-dan-spring-boot/). Silahkan buat sebuah file `docker-compose.yml` lalu isikan source code seperti berikut.

{% highlight yml %}
version: '2'

services:
  gitlab:
    container_name: docker-gitlab
    image: gitlab/gitlab-ce
    ports:
      - 80:80
    networks:
      main:
        aliases:
          - gitlab

  jenkins:
    container_name: docker-jenkins
    image: jenkins/jenkins
    ports:
      - 8080:8080
    networks:
      main:
        aliases:
          - jenkins

networks:
  main:
{% endhighlight %}

Konfigurasi docker compose yang kita gunakan adalah versi 2. Disini penulis membuat 2 container yaitu `docker-gitlab` dengan base image `gitlab/gitlab-ce` dan `docker-jenkins` dengan base image `jenkins/jenkins`. Masing - masing container akan melakukan expose port sehingga kita dapat melakukan akses dari luar container. Pada konfigurasi diatas kita menggunakan fungsi networks, fungsi networks ini memungkinkan kita untuk mengakses 2 way dependency antar container, secara default sebenarnya kita dapat menggunakan fungsi `links`, akan tetapi jika menggunakan 2 way dependency link maka akan muncul error, untuk mengatasi hal diatas maka kita gunakan networks untuk menghubungkan 2 container dengan kebutuhan 2 way dependency. Dengan menggunakan network maka container jenkins dapat mengakses [http://gitlab](http://gitlab) dan container gitlab dapat mengakses [http://jenkins](http://jenkins). Jika telah selesai, kita akan melakukan menjalankan 2 container diatas dengan perintah.

{% highlight bash %}
docker-compose up
{% endhighlight %}

Jika berhasil maka akan muncul output seperti berikut.

![Screen Shot 2017-07-30 at 4.56.25 PM.png](../images/Screen Shot 2017-07-30 at 4.56.25 PM.png)

## Setup Jenkins

Ketika jenkins startup, jangan lupa untuk mencatat key security, biasanya terdapat pada baris berikut.

{% highlight bash %}
docker-jenkins | Jul 30, 2017 10:03:54 AM jenkins.install.SetupWizard init
docker-jenkins | INFO:
docker-jenkins |
docker-jenkins | *************************************************************
docker-jenkins | *************************************************************
docker-jenkins | *************************************************************
docker-jenkins |
docker-jenkins | Jenkins initial setup is required. An admin user has been created and a password generated.
docker-jenkins | Please use the following password to proceed to installation:
docker-jenkins |
docker-jenkins | a541f4e0f9f94dff807afb3273c260a1
docker-jenkins |
docker-jenkins | This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
docker-jenkins |
docker-jenkins | *************************************************************
docker-jenkins | *************************************************************
docker-jenkins | *************************************************************
{% endhighlight %}

Dapat dilihat, bahwa key security yang akan kita gunakan nantinya adalah `a541f4e0f9f94dff807afb3273c260a1`. Jika anda melihat output seperti berikut.

{% highlight bash %}
docker-jenkins | --> setting agent port for jnlp
docker-jenkins | --> setting agent port for jnlp... done
docker-jenkins | Jul 30, 2017 10:04:02 AM hudson.model.UpdateSite updateData
docker-jenkins | INFO: Obtained the latest update center data file for UpdateSource default
docker-jenkins | Jul 30, 2017 10:04:04 AM hudson.model.DownloadService$Downloadable load
docker-jenkins | INFO: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
docker-jenkins | Jul 30, 2017 10:04:08 AM hudson.model.DownloadService$Downloadable load
docker-jenkins | INFO: Obtained the updated data file for hudson.tools.JDKInstaller
docker-jenkins | Jul 30, 2017 10:04:08 AM hudson.model.AsyncPeriodicWork$1 run
docker-jenkins | INFO: Finished Download metadata. 19,060 ms
docker-jenkins | Jul 30, 2017 10:04:11 AM hudson.model.UpdateSite updateData
docker-jenkins | INFO: Obtained the latest update center data file for UpdateSource default
docker-jenkins | Jul 30, 2017 10:04:11 AM hudson.WebAppMain$3 run
docker-jenkins | INFO: Jenkins is fully up and running
{% endhighlight %}

Ini menandakan bahwa jenkins adalah telah berjalan dan siap digunakan. Untuk gitlab, nantinya kita akan lakukan konfigurasi jika gitlab telah berjalan dengan sempurna, gitlab membutuhkan waktu yang lebih lama dikarenakan melakukan beberapa setup database dan lain sebagainya. Oke silahkan buka http://127.0.0.1:8080 di browser anda. Jika berhasil maka akan muncul output seperti berikut.

![Screen Shot 2017-07-30 at 5.07.19 PM.png](../images/Screen Shot 2017-07-30 at 5.07.19 PM.png)

Lalu silahkan isikan security key yang telah anda simpan. Jika berhasil maka akan muncul halaman `Customize Jenkins` untuk melakukan instalasi plugin, silahkan pilih `install suggested plugins`, nantinya anda akan dibawa ke halaman proses instalasi plugin seperti berikut.

![Screen Shot 2017-07-30 at 5.12.20 PM.png](../images/Screen Shot 2017-07-30 at 5.12.20 PM.png)

Jika instalasi plugin telah selesai, nantinya anda akan diarahkan ke halaman pengisian data administrator, silahkan isi sesuai dengan kebutuhan anda. Berikut adalah halaman awal ketika anda memasuki dashboard jenkins.

![Screen Shot 2017-07-30 at 5.19.29 PM.png](../images/Screen Shot 2017-07-30 at 5.19.29 PM.png)

## Instalasi Plugin Gitlab

Langkah selanjutnya, karena kita menggunakan gitlab, maka kita harus melakukan instalasi plugin gitlab terlebih dahulu. Berikut adalah langkah - langkahnya.

1. Silahkan pilih menu `manage jenkins` yang ada disebelah kanan anda.
2. Lalu pilih menu `manage plugins`.
3. Lalu pilih tab available
4. Kemudian silahkan cari plugin `GitLab Plugin`, cek list lalu pilih `install without restart`
5. Kemudian akan diarahkan ke halaman proses instalasi, silahkan cek list `Restart Jenkins when installation is complete and no jobs are running` agar jenkins melakukan restart setelah melakukan instalasi plugin tersebut.

## Setup Gradle Dan JDK

Project yang akan penulis deploy adalah project berbasis gradle, maka kita perlu melakukan setup gradle dan juga jdk nya. Berikut adalah langkah - langkahnya.

1. Silahkan pilih menu `manage jenkins` yang ada disebelah kanan anda.
2. Lalu pilih menu `Global Tool Configuration`
3. Pada menu JDK, silahkan pilih `Add JDK`
4. Bagian name isikan dengan `JDK 8`, cek list pada `agreement` dan jangan lupa login dengan menggunakan oracle account.
5. Pada menu gradle, silahkan pilih `Add Gradle`
6. Bagian name isikan dengan `Gradle 4.0.2`.
7. Jika telah selesai silahkan klik save.

## Setup Gitlab

Langkah selanjutnya kita akan melakukan setup gitlab, silahkan akses `http://127.0.0.1`, secara default gitlab menggunakan port 80. Ketika anda mengakses pertama kali, anda akan diminta untuk mengisi password baru, silahkan isi dengan kebutuhan anda. Jika telah selesai, anda akan dipindahkan ke halaman login seperti berikut.

![Screen Shot 2017-07-30 at 5.34.08 PM.png](../images/Screen Shot 2017-07-30 at 5.34.08 PM.png)

Silahkan login dengan menggunakan user `root` dan password yang telah anda register kan pada halaman sebelumnya. Jika berhasil maka akan muncul halaman seperti berikut.

![Screen Shot 2017-07-30 at 5.36.17 PM.png](../images/Screen Shot 2017-07-30 at 5.36.17 PM.png)

## Setup User Jenkins Pada Gitlab

Repository yang ada di dalam gitlab biasanya akan disetting menjadi private repository, sehingga hanya user tertentu yang dapat melakukan akses terhadap repository tersebut. Agar jenkins dapat melakukan akses repository tersebut maka kita harus melakukan register untuk user jenkins. Silahkan logout terlebih dahulu, lalu lakukan registrasi melalui halaman register gitlab. Setelah selesai, silahkan logout kembali lalu login kembali menggunakan user `root`. Silahkan buat sebuah group dengan nama `jenkins-group` lalu buat sebuah project di dalam `jenkins-group` dengan nama `Belajar-Jenkins` seperti gambar berikut.

![Screen Shot 2017-07-30 at 5.47.02 PM.png](../images/Screen Shot 2017-07-30 at 5.47.02 PM.png)

Setelah selesai, silahkan pilih menu admin area (gambar kunci) seperti berikut

![Screen Shot 2017-07-30 at 5.48.40 PM](../images/Screen Shot 2017-07-30 at 5.48.40 PM.png)

Maka akan muncul halaman baru, silahkan pilih group `jenkins-group` dibagian kanan bawah. Nantinya anda akan masuk ke halaman management user di dalam group tersebut, silahkan pilih user `jenkins` dan berikan hak akses sebagai `owner` seperti gambar berikut.

![Screen Shot 2017-07-30 at 5.50.41 PM.png](../images/Screen Shot 2017-07-30 at 5.50.41 PM.png)

## Setup Credentials Pada Jenkins

Setelah membuat user pada gitlab, sekarang kita akan mencoba membuat credentials pada jenkins, sehingga nantinya jenkins dapat melakukan clone dan deployment otomatis ke heroku. Silahkan pilih menu credentials pada menu jenkins yang berada di bawah enu `My views`, maka akan muncul menu system yang telah di expand, silahkan pilih menu system tersebut. Lalu pilih domain `Global credentials (unrestricted)`, dan kemudian pilih menu `Add Credentials`. Untuk user jenkins silahkan isikan seperti berikut.

![Screen Shot 2017-07-30 at 6.18.11 PM.png](../images/Screen Shot 2017-07-30 at 6.18.11 PM.png)

Untuk user heroku juga silahkan isikan seperti berikut.

![Screen Shot 2017-07-30 at 6.20.17 PM.png](../images/Screen Shot 2017-07-30 at 6.20.17 PM.png)

Yang terakhir, kita harus mendaftarkan api token gitlab pada jenkins, untuk membuat api token gitlab, silahkan login kembali ke gitlab dengan menggunakan user `root`, lalu pilih menu `setting` dan pilih tab `access token`. Lalu isikan seperti berikut.

![Screen Shot 2017-07-30 at 6.22.56 PM.png](../images/Screen Shot 2017-07-30 at 6.22.56 PM.png)

Jika sudah, nantinya akan muncul api token gitlab, kemudian silahkan isikan seperti berikut di menu jenkins.

![Screen Shot 2017-07-30 at 6.24.02 PM.png](../images/Screen Shot 2017-07-30 at 6.24.02 PM.png)

Berikut adalah list credentials yang didaftarkan.

![Screen Shot 2017-07-30 at 6.25.56 PM.png](../images/Screen Shot 2017-07-30 at 6.25.56 PM.png)

## Setup Connection Jenkins Ke Gitlab

Agar jenkins dapat mengakses source code pada gitlab maka kita harus melakukan konfigurasi connection terlebih dahulu pada jenkins. Berikut adalah tahapan nya.

1. Pilih menu manage jenkins.
2. Lalu pilih configure system.
3. Lalu isikan konfigurasi berikut pada menu gitlab

![Screen Shot 2017-07-30 at 6.38.07 PM.png](../images/Screen Shot 2017-07-30 at 6.38.07 PM.png)

Pada gitlab host URL, penulis menggunakan http://gitlab, ini disebabkan kita menggunakan network dari docker, sehingga untuk memanggil connection gitlab, kita dapat menggunakan url http://gitlab. Kemudian silahkan lakukan test connection, jika error silahkan periksa konfigurasinya kembali.

## Setup Project Pada Heroku

Silahkan login pada dashboard heroku, lalu silahkan buat sebuah app, bebas dengan nama apa saja, misalnya disini saya membuat projectnya dengan nama `belajar-jenkins`. Maka nantinya url git yang digunakan akan menjadi `git@heroku.com:belajar-jenkins.git`

## Membuat Project Pada Jenkins

Silahkan akses kembali dashboard jenkins, silahkan pilih menu `create new jobs` atau `new item`, lalu pilih menu `Freestyle Project`, nama project diisikan dengan `Belajar-Jenkins` lalu pilih oke. Untuk bagian general silahkan pilih gitlab connection seperti berikut.

![Screen Shot 2017-07-30 at 6.41.02 PM.png](../images/Screen Shot 2017-07-30 at 6.41.02 PM.png)

Pada source code management silahkan isikan seperti berikut.

![Screen Shot 2017-07-30 at 9.19.29 PM.png](../images/Screen Shot 2017-07-30 at 9.19.29 PM.png)

![Screen Shot 2017-07-30 at 9.19.34 PM.png](../images/Screen Shot 2017-07-30 at 9.19.34 PM.png)

Pada bagian rspec silahkan isikan sintak berikut untuk origin.

{% highlight bash %}
+refs/heads/*:refs/remotes/origin/* +refs/merge-requests/*/head:refs/remotes/origin/merge-requests/*
{% endhighlight %}

dan sintak berikut untuk heroku.

{% highlight bash %}
+refs/heads/*:refs/remotes/heroku/* +refs/merge-requests/*/head:refs/remotes/heroku/merge-requests/*
{% endhighlight %}

Pada bagian build triggers silahkan isikan seperti berikut.

![Screen Shot 2017-07-30 at 6.45.18 PM.png](../images/Screen Shot 2017-07-30 at 6.45.18 PM.png)

Pada bagian build environment silahkan pilih `Delete workspace before build starts`. Pada bagian build silahkan pilih `add build step` lalu pilih `invoke gradle script`. Pada bagian tasks silahkan isikan dengan perintah.

{% highlight bash %}
clean test
{% endhighlight %}

berikut adalah konfigurasi untuk invoke gradle

![Screen Shot 2017-07-30 at 9.24.06 PM.png](../images/Screen Shot 2017-07-30 at 9.24.06 PM.png)

Pada bagian Post-build Actions, silahkan pilih menu `add post-build action`, lalu pilih publish build status to gitlab commit, kemudian pilih menu git publisher, lalu silahkan ubah konfigurasinya menjadi seperti berikut.

![Screen Shot 2017-07-30 at 6.51.31 PM.png](../images/Screen Shot 2017-07-30 at 6.51.31 PM.png)

![Screen Shot 2017-07-30 at 6.51.39 PM.png](../images/Screen Shot 2017-07-30 at 6.51.39 PM.png)

## Setup WebHook Pada Gitlab

Setiap 1 project yang ada di jenkins akan mewakili 1 project atau bahkan akan mewakili 1 branch yang ada di gitlab. Silahkan login kembali pada gitlab dengan user `root`, lalu pilih project yang akan didaftarkan webhook nya. Pilih menu setting pada projectnya, lalu pilih menu integrations kemudian isikan konfigurasinya seperti berikut.

![Screen Shot 2017-07-30 at 7.07.08 PM.png](../images/Screen Shot 2017-07-30 at 7.07.08 PM.png)

Kemudian lakukan test webhook nya, jika berhasil maka akan muncul output seperti berikut.

{% highlight bash %}
Hook execution failed. Ensure the project has commits.
{% endhighlight %}

Pesan diatas dikarenakan repository masih kosong dan belum ada source code yang dicommit.

## Setup Project Yang Akan Di Deploy

Project yang akan kita deploy ke heroku adalah project spring boot dengan menggunakan kotlin, silahkan download terlebih dahulu projectnya di [Belajar-Jenkins](https://github.com/RizkiMufrizal/Belajar-Jenkins). Lalu silahkan extract project nya. Silahkan inisialisasi git pada project tersebut dengan perintah.

{% highlight bash %}
git init
{% endhighlight %}

Lalu inisialisasi remote git nya dengan perintah

{% highlight bash %}
git remote add origin http://127.0.0.1/jenkins-group/Belajar-Jenkins.git
{% endhighlight %}

lalu lakukan add semua project dengan perintah berikut.

{% highlight bash %}
git add .
{% endhighlight %}

lalu commit seperti berikut.

{% highlight bash %}
git commit -m "deploy jenkins"
{% endhighlight %}

## Melakukan Deploy

Untuk melakukan deploy, silahkan lakukan push dengan perintah.

{% highlight bash %}
git push origin master
{% endhighlight %}

Ketika dijalankan teryata menyebabkan error, ini disebabkan karena ssh key yang ada di docker belum kita daftarkan pada heroku. Sebelumnya, kita akan melakukan instalasi heroku cli terlebih dahulu pada docker, untuk dapat mengakses ssh docker, kita membutuhkan id container yang sedang aktif. Silahkan jalan perintah berikut untuk mengetahui id nya.

{% highlight bash %}
docker ps
{% endhighlight %}

maka hasilnya akan seperti berikut.

{% highlight bash %}
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                 PORTS                                 NAMES
a794b66cca69        jenkins/jenkins     "/bin/tini -- /usr..."   3 hours ago         Up 3 hours             0.0.0.0:8080->8080/tcp, 50000/tcp     docker-jenkins
2cb159c58b5e        gitlab/gitlab-ce    "/assets/wrapper"        3 hours ago         Up 3 hours (healthy)   22/tcp, 443/tcp, 0.0.0.0:80->80/tcp   docker-gitlab
{% endhighlight %}

Untuk mengakses root container silahkan jalankan perintah berikut.

{% highlight bash %}
docker exec -u 0 -it a794b66cca69 bash
{% endhighlight %}

Kemudian silahkan jalankan perintah dibawah ini berdasarkan baris nya.

{% highlight bash %}
wget https://cli-assets.heroku.com/heroku-cli/channels/stable/heroku-cli-linux-x64.tar.gz -O heroku.tar.gz
$ tar -xvzf heroku.tar.gz
$ mkdir -p /usr/local/lib /usr/local/bin
$ mv heroku-cli-v6.x.x-darwin-64 /usr/local/lib/heroku
$ ln -s /usr/local/lib/heroku/bin/heroku /usr/local/bin/heroku
{% endhighlight %}

Setelah selesai, silahkan keluar dari ssh, kemudian login lagi ke ssh dengan perintah berikut.

{% highlight bash %}
docker exec -i -t a794b66cca69 /bin/bas
{% endhighlight %}

Kemudian lakukan login heroku dengan perintah.

{% highlight bash %}
heroku login
{% endhighlight %}

silahkan lakukan login dengan username dan password anda. Tahap selanjutnya, silahkan generate ssh anda dengan perintah.

{% highlight bash %}
ssh-keygen -t rsa -C "mufrizalrizki@gmail.com"
{% endhighlight %}

Lalu setting git nya dengan perintah

{% highlight bash %}
git config --global user.name "RizkiMufrizal"
git config --global user.email "mufrizalrizki@gmail.com"
git config --global color.ui true
{% endhighlight %}

Setelah selesai, silahkan jalankan perintah berikut untuk memunculkan public key.

{% highlight bash %}
cat ~/.ssh/id_rsa.pub
{% endhighlight %}

Kemudian hasilnya public key nya silahkan didaftarkan di heroku. Setelah selesai, coba lakukan clone repository dari heroku agar mendapatkan hak akses ssh nya dengan perintah.

{% highlight bash %}
git clone git@heroku.com:belajar-jenkins.git
{% endhighlight %}

Maka akan muncul info seperti berikut.

{% highlight bash %}
Cloning into 'belajar-jenkins'...
The authenticity of host 'heroku.com (50.19.85.132)' can't be established.
RSA key fingerprint is SHA256:8tF0wX2WquK45aGKs/Bh1dKmBXH08vxUe0VCJJWOA/o.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'heroku.com,50.19.85.132' (RSA) to the list of known hosts.
{% endhighlight %}

Silahkan ketik yes, maka project akan di clone dari heroku. Nah jika telah selesai, silahkan balik lagi ke dashboard project jenkins, silahkan pilih menu build now, sehingga jenkins akan menjalankan ulang proses test build dan deploy ke heroku.

Jika berhasil maka akan muncul output seperti berikut.

![Screen Shot 2017-07-30 at 9.25.52 PM.png](../images/Screen Shot 2017-07-30 at 9.25.52 PM.png)

Jika ingin melihat status pipeline yang ada di gitlab, silahkan login kembali dengan user `root` lalu akses project, pilih menu `pipelines` dan berikut adalah hasilnya.

![Screen Shot 2017-07-30 at 9.29.58 PM.png](../images/Screen Shot 2017-07-30 at 9.29.58 PM.png)

Untuk mengecek apakah sudah berhasil di deploy atau belum, silahkan cek di https://{nama-aplikasi-heroku}.herokuapp.com/api/barangs, karena saya menggunakan belajar-jenkins, maka saya mengakses nya di [https://belajar-jenkins.herokuapp.com/api/barangs](https://belajar-jenkins.herokuapp.com/api/barangs) dan berikut adalah hasilnya.

![Screen Shot 2017-07-30 at 9.28.27 PM.png](../images/Screen Shot 2017-07-30 at 9.28.27 PM.png)

Jika sewaktu - waktu terjadi error pada saat testing, maka jenkins akan mengeluarkan pesan error dan deployment ke production tidak akan dilakukan, berikut output jika testing error.

![Screen Shot 2017-07-30 at 9.42.22 PM.png](../images/Screen Shot 2017-07-30 at 9.42.22 PM.png)

Dan berikut jika output yang akan ditampilkan pada gitlab.

![Screen Shot 2017-07-30 at 9.42.25 PM.png](../images/Screen Shot 2017-07-30 at 9.42.25 PM.png)

Sekian artikel mengenai Belajar Melakukan Integrasi Jenkins Dan Gitlab Pada Docker, jika ada pertanyaan atau saran silahkan isi di kolom komentar dan Terima kasih :).
