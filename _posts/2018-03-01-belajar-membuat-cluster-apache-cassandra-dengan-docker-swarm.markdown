---
layout: post
title: Belajar Membuat Cluster Apache Cassandra Dengan Docker Swarm
modified:
categories:
description: Belajar Membuat Cluster Apache Cassandra Dengan Docker Swarm
tags: [docker, swarm, apache, cassandra, cluster, data center, data recovery, rack]
image:
  background: abstract-2.png
comments: true
share: true
date: 2018-03-01T20:15:28+07:00
---

Apache Cassandra adalah salah satu database yang banyak digunakan, terutama jika anda adalah developer API Gateway. Beberapa product API Gateway seperti axway dan kong menggunakan database Apache Cassandra untuk menyimpan seluruh konfigurasi API Gateway. Agar data lebih aman ketika terjadi sesuatu yang tidak diinginkan pada database Apache Cassandra maka dibutuhkan suatu proses replication pada database tersebut. Secara default, Apache Cassandra mendukung replication dimana replication yang ditawarkan dari Apache Cassandra adalah master - master, sehingga kita tidak perlu membuat 2 konfigurasi seperti konfigurasi master - slave pada MySQL atau mariadb. Pada artikel ini, penulis akan membahas mengenai bagaimana cara membuat replication di dalam 1 cluster apache cassandra dengan menggunakan docker swarm.

## Arsitektur Cluster Apache Cassandra

Arsitektur cluster Apache Cassandra yang akan kita gunakan adalah sebagai berikut

![Cassandra.png](../images/Cassandra.png)

Pada gambar diatas terdapat :

* 6 Node Apache Cassandra, node disini adalah satu instalasi Apache Cassandra, dimana 1 node hanya dipasang pada 1 server, maka seharusnya kita menggunakan 6 server.
* 3 Rack, rack disini untuk menentukan replication akan dilakukan secarah horizontal
* 2 Data Center (data center dan data recovery), di dalam cassandra biasanya penulis menentukan 2 data center sehingga data center pertama adalah sebagai data center pada umumnya dan data ceter kedua sebagai data recovery.
* 1 Cluster

## Apa Itu Docker Swarm

Sebelum membahas tentang arsitektur nya, kita akan membahas terlebih dahulu mengenai docker swarm.

>>Docker Swarm adalah Salah satu product nya docker untuk dapat mendeploy container pada multihost.

Multihost disini adalah di banyak server. Misalnya kita mempunyai 2 server, misalnya server A dan server B. Masing - masing server akan dilakukan instalasi docker, dan masing - masing docker mempunyai 1 container yang sedang berjalan. Bagaimana caranya agar container yang terdapat pada server A dan melakukan komunikasi dengan container yang ada pada server B, sedangkan jaringan yang terdapat pada server berbeda dengan jaringan yang terdapat di dalam masing - masing container. Jawaban nya adalah dengan menggunakan docker swarm, dengan menggunakan docker swarm maka docker yang yang terdapat di dalam beberapa host dapat kita lakukan cluster sehingga seakan - akan docker tersebut terdapat pada 1 host. Dengan menggunakan docker swarm, kita bebas menetukan container tersebut ingin di deploy ke host yang diinginkan.

## Arsitektur Cluster Apache Cassandra Pada Docker Swarm

Berikut adalah arsitektur yang akan digunakan.

![Docker Swarm.png](../images/Docker Swarm.png)

Pada gambar diatas terdapat

* 2 server yaitu dengan IP `192.168.50.2` dan `192.168.50.3`
* 3 container di masing - masing server, di dalam 1 container terdapat 1 node cassandra

Dari gambar diatas dapat kita lihat bahwa antar server, terdapat jaringan fisik misalnya LAN dan sebagainya, sedangkan di dalam docker container terdapat jaringan tersendiri, sehingga untuk menyambungkan container antar server/host maka digunakan jaringan overlay.

>>Jaringan Overlay adalah jaringan komputer virtual yang dibangun di atas jaringan lain.

Jaringan overlay ini nantinya akan dibuatkan oleh docker swarm sehingga kita tidak perlu melakukan konfigurasi secara manual, sehingga setiap container yang berbeda host dapat berkomunikasi pada 1 jaringan overlay.

## Setup Centos 7 Dengan Vagrant

Pada artikel ini, penulis akan menggunakan vagrant untuk virtualisasi 2 server, bagi yang belum paham vagrant, silahkan simak artikel [Belajar Vagrant](https://rizkimufrizal.github.io/belajar-vagrant/). Silahkan buat sebuah file `Vagrantfile` lalu masukkan code berikut.

{% highlight bash %}
Vagrant.configure("2") do |config|

  config.vm.provider "virtualbox" do |v|
    v.memory = 3072
  end

  config.vm.define "master" do |master|
    master.vm.box = "centos/7"
    master.vm.hostname = 'master'
    master.vm.network "private_network", ip: "192.168.50.2"
  end

  config.vm.define "worker" do |worker|
    worker.vm.box = "centos/7"
    worker.vm.hostname = 'worker'
    worker.vm.network "private_network", ip: "192.168.50.3"
  end
end
{% endhighlight %}

Dari konfigurasi diatas, kita membuat 2 server dengan masing - masing ip yaitu `192.168.50.2` dan `192.168.50.3`. Sistem operasi yang kita gunakan adalah centos 7. Setelah selesai, silahkan jalankan perintah berikut untuk menjalankan kedua server tersebut.

{% highlight bash %}
vagrant up
{% endhighlight %}

Setelah selesai, silahkan akses kedua vagrant tersebut dengan perintah

{% highlight bash %}
vagrant ssh master
{% endhighlight %}

{% highlight bash %}
vagrant ssh worker
{% endhighlight %}

Lakukan perintah berikut pada kedua server diatas. Silahkan login dengan user dengan perintah.

{% highlight bash %}
sudo -s
{% endhighlight %}

Kemudian jalankan perintah berikut untuk menghapus docker versi lama

{% highlight bash %}
yum remove docker docker-common docker-selinux docker-engine
{% endhighlight %}

Jalankan perintah berikut untuk instalasi config manager yum

{% highlight bash %}
yum install -y yum-utils device-mapper-persistent-data lvm2
{% endhighlight %}

Lalu tambahkan repo docker pada centos dengan perintah berikut.

{% highlight bash %}
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
{% endhighlight %}

Lalu lakukan update dengan perintah

{% highlight bash %}
yum update -y
{% endhighlight %}

Setelah selesai, lakukan instalasi docker dengan perintah

{% highlight bash %}
yum install -y docker-ce
{% endhighlight %}

Setelah selesai, silahkan jalankan docker dengan perintah

{% highlight bash %}
systemctl start docker
{% endhighlight %}

## Setup Docker Swarm

Silahkan akses server `master`, lalu jalankan perintah berikut untuk inisialisasi docker swarm

{% highlight bash %}
docker swarm init --advertise-addr 192.168.50.2
{% endhighlight %}

IP diatas adalah IP dari server master, jika berhasil maka akan muncul output seperti berikut.

{% highlight bash %}
[root@master vagrant]# docker swarm init --advertise-addr 192.168.50.2
Swarm initialized: current node (hw3kzewnluvm9gmf6cz8sfunb) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2il6rouemqho08tl2dmipd77prj293gqqn8elzgzeiu5qhmwrb-efy6r4nj9wnrvci6x7aen3ikz 192.168.50.2:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
{% endhighlight %}

Lalu silahkan akses server `worker` lalu jalankan perintah join seperti berikut

{% highlight bash %}
docker swarm join --token SWMTKN-1-2il6rouemqho08tl2dmipd77prj293gqqn8elzgzeiu5qhmwrb-efy6r4nj9wnrvci6x7aen3ikz 192.168.50.2:2377
{% endhighlight %}

Perintah join tersebut berfungsi untuk mendaftarkan server/host pada docker swarm master. Setelah selesai, silahkan akses server `master` kembali lalu jalankan perintah berikut untuk melihat node / host / server apa saja yang telah terdaftar.

{% highlight bash %}
docker node ls
{% endhighlight %}

Dan berikut adalah outputnya.

{% highlight bash %}
[root@master vagrant]# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
hw3kzewnluvm9gmf6cz8sfunb *   master              Ready               Active              Leader
wknzip11beucg2u4e6u46noy4     worker              Ready               Active
{% endhighlight %}

Setelah selesai, langkah berikut nya yaitu kita akan memberikan penamaan kepada setiap node / host / server agar nantinya kita dapat menentukan sebuah container dapat di deploy pada node / host / server yang mana. Silahkan jalankan perintah berikut untuk melakukan penamaan label pada node / host / server master dan worker.

{% highlight bash %}
docker node update --label-add server=master hw3kzewnluvm9gmf6cz8sfunb
docker node update --label-add server=worker wknzip11beucg2u4e6u46noy4
{% endhighlight %}

ID diatas dapat dilihat dengan perintah menampilkan semua node yang telah kita lakukan sebelumnya.

## Membuat Compose Cassandra Untuk Docker Swarm

Langkah selanjutnya, kita akan membuat sebuah file yaitu `docker-compose.yml`, isinya sama seperti konfigurasi docker compose hanya saja terdapat penambahan untuk kebutuhan docker swarm, silahkan buat file `docker-compose.yml` di dalam server master, lalu tambahkan code berikut.

{% highlight yaml %}
version: '3'
services:
  cassandra_dc_1:
    image: cassandra:latest
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 0; fi && /docker-entrypoint.sh cassandra -f'
    deploy:
      placement:
        constraints:
          - node.labels.server == master
      restart_policy:
        condition: on-failure
        max_attempts: 3
        window: 120s
    environment:
      CASSANDRA_CLUSTER_NAME: "CassandraCluster"
      CASSANDRA_BROADCAST_ADDRESS: cassandra_dc_1
      CASSANDRA_SEEDS: cassandra_dc_1,cassandra_dr_1
      CASSANDRA_DC: DC
      CASSANDRA_RACK: RACK1
      CASSANDRA_ENDPOINT_SNITCH: GossipingPropertyFileSnitch
      MAX_HEAP_SIZE: 300m
      HEAP_NEWSIZE: 200m
    ports:
    - "7000"
    networks:
      default:

  cassandra_dc_2:
    image: cassandra:latest
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 120; fi && /docker-entrypoint.sh cassandra -f'
    deploy:
      placement:
        constraints:
          - node.labels.server == master
      restart_policy:
        condition: on-failure
        max_attempts: 3
        window: 120s
    environment:
      CASSANDRA_CLUSTER_NAME: "CassandraCluster"
      CASSANDRA_BROADCAST_ADDRESS: cassandra_dc_2
      CASSANDRA_SEEDS: cassandra_dc_1,cassandra_dr_1
      CASSANDRA_DC: DC
      CASSANDRA_RACK: RACK2
      CASSANDRA_ENDPOINT_SNITCH: GossipingPropertyFileSnitch
      MAX_HEAP_SIZE: 300m
      HEAP_NEWSIZE: 200m
    depends_on:
      - cassandra_dc_1
    ports:
    - "7000"
    networks:
      default:

  cassandra_dc_3:
    image: cassandra:latest
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 240; fi && /docker-entrypoint.sh cassandra -f'
    deploy:
      placement:
        constraints:
          - node.labels.server == master
      restart_policy:
        condition: on-failure
        max_attempts: 3
        window: 120s
    environment:
      CASSANDRA_CLUSTER_NAME: "CassandraCluster"
      CASSANDRA_BROADCAST_ADDRESS: cassandra_dc_3
      CASSANDRA_SEEDS: cassandra_dc_1,cassandra_dr_1
      CASSANDRA_DC: DC
      CASSANDRA_RACK: RACK3
      CASSANDRA_ENDPOINT_SNITCH: GossipingPropertyFileSnitch
      MAX_HEAP_SIZE: 300m
      HEAP_NEWSIZE: 200m
    depends_on:
      - cassandra_dc_2
    ports:
    - "7000"
    networks:
      default:

  cassandra_dr_1:
    image: cassandra:latest
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 60; fi && /docker-entrypoint.sh cassandra -f'
    deploy:
      placement:
        constraints:
          - node.labels.server == worker
      restart_policy:
        condition: on-failure
        max_attempts: 3
        window: 120s
    environment:
      CASSANDRA_CLUSTER_NAME: "CassandraCluster"
      CASSANDRA_BROADCAST_ADDRESS: cassandra_dr_1
      CASSANDRA_SEEDS: cassandra_dr_1,cassandra_dc_1
      CASSANDRA_DC: DR
      CASSANDRA_RACK: RACK1
      CASSANDRA_ENDPOINT_SNITCH: GossipingPropertyFileSnitch
      MAX_HEAP_SIZE: 300m
      HEAP_NEWSIZE: 200m
    depends_on:
      - cassandra_dc_1
    ports:
    - "7000"
    networks:
      default:

  cassandra_dr_2:
    image: cassandra:latest
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 180; fi && /docker-entrypoint.sh cassandra -f'
    deploy:
      placement:
        constraints:
          - node.labels.server == worker
      restart_policy:
        condition: on-failure
        max_attempts: 3
        window: 120s
    environment:
      CASSANDRA_CLUSTER_NAME: "CassandraCluster"
      CASSANDRA_BROADCAST_ADDRESS: cassandra_dr_2
      CASSANDRA_SEEDS: cassandra_dr_1,cassandra_dc_1
      CASSANDRA_DC: DR
      CASSANDRA_RACK: RACK2
      CASSANDRA_ENDPOINT_SNITCH: GossipingPropertyFileSnitch
      MAX_HEAP_SIZE: 300m
      HEAP_NEWSIZE: 200m
    depends_on:
      - cassandra_dr_1
    ports:
    - "7000"
    networks:
      default:

  cassandra_dr_3:
    image: cassandra:latest
    command: bash -c 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 300; fi && /docker-entrypoint.sh cassandra -f'
    deploy:
      placement:
        constraints:
          - node.labels.server == worker
      restart_policy:
        condition: on-failure
        max_attempts: 3
        window: 120s
    environment:
      CASSANDRA_CLUSTER_NAME: "CassandraCluster"
      CASSANDRA_BROADCAST_ADDRESS: cassandra_dr_3
      CASSANDRA_SEEDS: cassandra_dr_1,cassandra_dc_1
      CASSANDRA_DC: DR
      CASSANDRA_RACK: RACK3
      CASSANDRA_ENDPOINT_SNITCH: GossipingPropertyFileSnitch
      MAX_HEAP_SIZE: 300m
      HEAP_NEWSIZE: 200m
    depends_on:
      - cassandra_dr_2
    ports:
    - "7000"
    networks:
      default:

networks:
  default:
{% endhighlight %}

Setelah selesai, silahkan jalankan perintah berikut untuk melakukan pull image cassandra terlebih dahulu.

{% highlight bash %}
docker pull cassandra
{% endhighlight %}

Setelah selesai, silahkan jalankan semua cassandra dengan perintah berikut

{% highlight bash %}
docker stack deploy --compose-file docker-compose.yml cassandra
{% endhighlight %}

Jika berhasil, silahkan cek service nya dengan perintah

{% highlight bash %}
docker service ls
{% endhighlight %}

Maka akan muncul output seperti berikut

{% highlight bash %}
ID                  NAME                       MODE                REPLICAS            IMAGE               PORTS
hx95sgod38ip        cassandra_cassandra_dc_1   replicated          1/1                 cassandra:latest    *:30020->7000/tcp
n3dbljgnptsj        cassandra_cassandra_dc_2   replicated          1/1                 cassandra:latest    *:30021->7000/tcp
tdqtwh7eaib3        cassandra_cassandra_dc_3   replicated          1/1                 cassandra:latest    *:30016->7000/tcp
l69lcdacvnnw        cassandra_cassandra_dr_1   replicated          1/1                 cassandra:latest    *:30017->7000/tcp
tgyypjfaeenf        cassandra_cassandra_dr_2   replicated          1/1                 cassandra:latest    *:30018->7000/tcp
7ao6ubi2j7e4        cassandra_cassandra_dr_3   replicated          1/1                 cassandra:latest    *:30019->7000/tcp
{% endhighlight %}

Silahkan cek container yang sudah jalan dengan perintah

{% highlight bash %}
docker ps
{% endhighlight %}

Maka akan muncul output seperti berikut

{% highlight bash %}
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                         NAMES
305e963a4989        cassandra:latest    "docker-entrypoint.s…"   8 minutes ago       Up 8 minutes        7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp   cassandra_cassandra_dc_2.1.53t1s2c5obqwwfzgenbnj3a3f
03501ebc8870        cassandra:latest    "docker-entrypoint.s…"   8 minutes ago       Up 8 minutes        7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp   cassandra_cassandra_dc_1.1.qj4nikx2x9qweqrjp5h2i644a
16b1d316c721        cassandra:latest    "docker-entrypoint.s…"   8 minutes ago       Up 8 minutes        7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp   cassandra_cassandra_dc_3.1.f3fesq271jnpky0zw1la4vdhy
{% endhighlight %}

Cari `NAMES` yang memiliki nama `cassandra_cassandra_dc_1`, misalnya jika dilihat dari atas, kita menemukan nya di `cassandra_cassandra_dc_1.1.qj4nikx2x9qweqrjp5h2i644a`. Lalu silahkan akses container tersebut dengan perintah.

{% highlight bash %}
docker exec -it 03501ebc8870 /bin/bash
{% endhighlight %}

Lalu jalankan perintah berikut untuk mengecek cluster cassandra

{% highlight bash %}
nodetool status
{% endhighlight %}

Jika berhasil maka akan muncul output seperti berikut.

{% highlight bash %}
Datacenter: DC
==============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.0.0.67  92.35 KiB  256          31.5%             a5c66734-eb77-4c2c-9555-7b6de6d61529  RACK1
UN  10.0.0.69  75.05 KiB  256          34.2%             4a1db9a3-8a29-4da7-9fce-809ef056a35c  RACK2
UN  10.0.0.59  74.99 KiB  256          35.2%             0e12bb91-91db-4344-b532-1ca01a0e36df  RACK3
Datacenter: DR
==============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.0.0.61  69.93 KiB  256          33.1%             d07abb28-c4ce-4b49-b322-0aed46324adc  RACK1
UN  10.0.0.63  69.93 KiB  256          34.9%             697e25ba-0e1f-4e8b-ba65-d7fa0c8c98b1  RACK2
UN  10.0.0.65  74.98 KiB  256          31.2%             3c28cce9-7514-4a54-97ca-97a063ecdd06  RACK3
{% endhighlight %}

Sekian artikel mengenai Belajar Membuat Cluster Apache Cassandra Dengan Docker Swarm dan terima kasih :).