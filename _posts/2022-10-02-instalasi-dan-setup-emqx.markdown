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
date: 2022-10-02T20:15:28+07:00
---

Setalah sekian lama, akhirnya penulis mencoba menyempatkan kembali menulis sedikit artikel untuk bisa berbagi ke teman - teman. Pada artikel ini, penulis akan membahas mengenai salah satu broker MQTT yang cukup populer yaitu EMQX.

## Apa itu MQTT ?

>>MQTT (Message Queuing Telemetry Transport) merupakah protokol yang digunakan untuk IoT (internet of things). Protokol ini biasanya digunakan untuk komunikasi antar mesin ke mesin, misal nya seperti perangkat arduino, raspi dan lain - lain. Protokol MQTT ini dirancang khusus untuk komunikasi mesin ke mesin dan protokol ini juga berjalan diatas TCP/IP. Berbeda dengan protokol HTTP, protokol MQTT menggunakan mekanisme publish subscribe, dimana penggunaan nya sama seperti message queue seperti Active MQ, Apache Kafka, Rabbit MQ dan lain - lain.

## Apa itu EMQX ?

EMQX is an Open-source MQTT broker with a high-performance real-time message processing engine, powering event streaming for IoT devices at massive scale.

>>EMQX adalah salah satu broker MQTT yang bersifat opensource dan termasuk broker yang banyak menawarkan fitur untuk kebutuhan IoT. EMQX juga menyediakan versi enterprise jika kita membutuhkan fitur - fitur tambahan, misal Enterprise Data Integration (fitur integrasi dengan external database seperti oracle, postgresql dan lain - lain).

## Instalasi EMQX

Pada artikel ini, penulis akan menggunakan 2 buat vm yaitu :

1. VM Node 1 : berfungsi sebagai node master, node master berfungsi untuk primary node nya.
2. VM Node 2 : berfungsi sebagai node worker, dimana node worker akan melakukan join cluster ke node master.

### Requirment Per Node

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
    node.vm.network "private_network", ip: "192.168.50.2"
  end

  config.vm.define "node2" do |node|
    node.vm.box = "centos/7"
    node.vm.hostname = 'node2'
    node.vm.network "private_network", ip: "192.168.50.3"
  end
end
{% endhighlight %}

Berikut adalah gambaran arsitektur yang penulis gunakan untuk membuat cluster EMQX.

![mqtt-arsitektur.png](../images/mqtt-arsitektur.png)