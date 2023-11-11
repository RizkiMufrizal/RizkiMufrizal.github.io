---
layout: post
title: Instalasi Dan Setup EMQX
modified:
categories:
description: Instalasi Dan Setup EMQX
tags: [EMQX, MQTT]
image:
  background: abstract-2.png
comments: true
share: true
date: 2023-12-01T20:15:28+07:00
---

# Apa itu MQTT ?

>>MQTT (Message Queuing Telemetry Transport) merupakah protokol yang digunakan untuk IoT (internet of things). Protokol ini biasanya digunakan untuk komunikasi antar mesin ke mesin, misal nya seperti perangkat arduino, raspi dan lain - lain. Protokol MQTT ini dirancang khusus untuk komunikasi mesin ke mesin dan protokol ini juga berjalan diatas TCP/IP. Berbeda dengan protokol HTTP, protokol MQTT menggunakan mekanisme publish subscribe, dimana penggunaan nya sama seperti message queue seperti Active MQ, Apache Kafka, Rabbit MQ dan lain - lain.

# Apa itu EMQX ?

>>EMQX adalah salah satu broker MQTT yang cukup populer bersifat opensource dan termasuk broker yang banyak menawarkan fitur untuk kebutuhan IoT. EMQX juga menyediakan versi enterprise jika kita membutuhkan fitur - fitur tambahan, misal Enterprise Data Integration (fitur integrasi dengan external database seperti oracle, postgresql dan lain - lain).

## Instalasi EMQX

Pada artikel ini, penulis akan menggunakan 2 buat vm yaitu :

1. VM Node 1 : berfungsi sebagai node master, node master berfungsi untuk primary node nya.
2. VM Node 2 : berfungsi sebagai node worker, dimana node worker akan melakukan join cluster ke node master.

## Requirment Per Node

Adapun kebutuhan untuk per node adalah : 

* Ram 1 GB
* Disk 10 GB
* IP 192.168.50.2 (node 1)
* IP 192.168.50.3 (node 2)
* Centos 7

Untuk memudahkan setup VM per masing - masing node, penulis mencoba menggunakan [vagrant](https://rizkimufrizal.github.io/belajar-vagrant/). Berikut adalah konfigurasi Vagrantfile yang digunakan oleh penulis.

{% highlight bash %}
Vagrant.configure("2") do |config|

  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
  end

  config.vm.define "node1" do |node|
    node.vm.box = "centos/7"
    node.vm.hostname = 'node1'
    node.vm.network "private_network", ip: "192.168.56.2"
  end

  config.vm.define "node2" do |node|
    node.vm.box = "centos/7"
    node.vm.hostname = 'node2'
    node.vm.network "private_network", ip: "192.168.56.3"
  end
end
{% endhighlight %}

Berikut adalah gambaran arsitektur yang penulis gunakan untuk membuat cluster EMQX.

![mqtt-arsitektur.png](../images/mqtt-arsitektur.png)

Lalu jalankan vagrant dengan perintah

{% highlight bash %}
vagrant up
{% endhighlight %}

Selanjutnya silahkan akses node 1 dengan perintah

{% highlight bash %}
vagrant ssh node1
{% endhighlight %}

Setalah masuk ke node 1, silahkan jalankan perintah berikut untuk melakukan instalasi

{% highlight bash %}
sudo yum update -y && sudo yum install wget -y
sudo mkdir /apps
sudo chown -R vagrant:vagrant /apps/
cd /apps
wget https://www.emqx.com/en/downloads/broker/5.0.8/emqx-5.0.8-el7-amd64.tar.gz
mkdir -p emqx && tar -zxvf emqx-5.0.8-el7-amd64.tar.gz -C emqx
{% endhighlight %}

Dan lakukan juga perintah tersebut pada node 2.

## Setup EMQX

Setelah melakukan instalasi EMQX, tahap selanjutnya adalah melakukan setup cluster untuk EMQX. Setup yang dilakukan ada 3 bagian yaitu tunning server, setup node 1 dan setup node 2.

## Tunning Server dan TCP Network

Ada beberapa yang harus dilakukan tunning baik disisi server maupun disisi TCP Network. Berikut adalah tahapan nya

### Tunning kernel linux

tunning ini wajib dilakukan dengan menggunakan user root. Silahkan jalankan perintah berikut

{% highlight bash %}
sysctl -w fs.file-max=2097152
sysctl -w fs.nr_open=2097152
echo 2097152 > /proc/sys/fs/nr_open
ulimit -n 1048576
{% endhighlight %}

Silahkan buka file `/etc/sysctl.conf` lalu tambahkan code berikut

{% highlight bash %}
fs.file-max = 1048576
{% endhighlight %}

Kemudian buka file `/etc/systemd/system.conf` dan tambahkan juga code berikut

{% highlight bash %}
DefaultLimitNOFILE=1048576
{% endhighlight %}

dan yang terakhir silahkan buka file `/etc/security/limits.conf` dan tambahkan code berikut

{% highlight bash %}
*      soft    nofile      1048576
*      hard    nofile      1048576
{% endhighlight %}

### Tunning TCP Network

Untuk melakukan tunning disisi TCP Network, cukup jalankan perintah berikut

{% highlight bash %}
sysctl -w net.core.somaxconn=32768
sysctl -w net.ipv4.tcp_max_syn_backlog=16384
sysctl -w net.core.netdev_max_backlog=16384
sysctl -w net.ipv4.ip_local_port_range='1000 65535'
sysctl -w net.core.rmem_default=262144
sysctl -w net.core.wmem_default=262144
sysctl -w net.core.rmem_max=16777216
sysctl -w net.core.wmem_max=16777216
sysctl -w net.core.optmem_max=16777216
sysctl -w net.ipv4.tcp_rmem='1024 4096 16777216'
sysctl -w net.ipv4.tcp_wmem='1024 4096 16777216'
sysctl -w net.nf_conntrack_max=1000000
sysctl -w net.netfilter.nf_conntrack_max=1000000
sysctl -w net.netfilter.nf_conntrack_tcp_timeout_time_wait=30
sysctl -w net.ipv4.tcp_max_tw_buckets=1048576
sysctl -w net.ipv4.tcp_fin_timeout=15
{% endhighlight %}