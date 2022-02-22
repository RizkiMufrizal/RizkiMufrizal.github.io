---
layout: post
title: Instalasi Dan Setup Kubernetes Cluster
modified:
categories:
description: Instalasi Dan Setup Kubernetes Cluster
tags: [kubernetes, cluster, docker, kubernetes dashboard]
image:
  background: abstract-2.png
comments: true
share: true
date: 2018-05-01T20:15:28+07:00
---

Pada artikel sebelumnya [Belajar Membuat Cluster Apache Cassandra Dengan Docker Swarm](https://rizkimufrizal.github.io/belajar-membuat-cluster-apache-cassandra-dengan-docker-swarm/), penulis telah membahas mengenai tentang cluster pada apache cassandra dengan menggunakan docker swarm. Untuk dapat membuat cluster pada apache cassandra, kita juga dapat menggunakan kubernetes sebagai penganti dari docker swarm. Jika docker swarm merupakan bawaan dari docker maka kubernetes perlu dilakukan instalasi dan setup. Pada artikel ini, penulis akan membahas bagaimana cara instalasi, setup kubernetes dan membuat cluster apache cassandra pada kubernetes.

## Apa Itu Kubernetes ?

>>Kubernetes adalah salah satu produk open source untuk sistem manajemen container

Sama seperti docker swarm, kubernetes dapat melakukan manajemen container, akan tetapi kubernetes merupakan produk yang akan terus berkembang dan akan lebih banyak digunakan dibandingkan dengan docker swarm. Salah satu kelebihan dari kubernetes adalah kita dapat menggunakan kubernetes dashboard untuk memonitoring container, kita dapat melakukan deployment melalui dashboard dan pastinya struktur yang diusulkan oleh kubernetes dan docker swarm sangatlah berbeda. Berikut adalah struktur yang biasanya digunakan di dalam kubernetes.

![Arsitektur Kubernetes.png](../images/Arsitektur Kubernetes.png)

Berikut adalah penjelasan nya

* Node : biasanya sebagai server, bisa sebagai server fisik atau virtual server sehingga node ini juga dapat berupa sebuah server yang berjalan diatas virtual machine seperti VMWare atau virtualbox.
* Deployment : biasanya berfungsi untuk mengontrol dari sebuah pods, misalnya adanya update dari sebuah pods, adanya penambahan replikasi pada pods dan lain sebagainya.
* Service : berfungsi untuk mengexpose pods, tujuan nya adalah agar client dapat mengakses service yang terdapat di dalam sebuah pods.
* Pods : di dalam sebuah deployment terdapat beberapa pods, biasanya pods ini terdapat 1 container atau lebih. Pods ini biasanya akan dilakukan replikasi sesuai dengan kebutuhan.
* Container : container ini mewakili dari 1 aplikasi, misalnya database postgresql atau web server apache tomcat.

## Instalasi Kubernetes

Pada artikel ini, kita akan menggunakan 2 buat vm yaitu :

1. VM Master : berfungsi sebagai node master, master disini berfungsi untuk mendeploy container ke dalam node worker kubernetes.
2. VM Worker : berfungsi sebagai node worker.

### Requirment Node Master

Adapun kebutuhan untuk node master adalah : 

* Ram 3 GB
* Disk 10 GB
* Bridge Network
* IP 192.168.88.100
* Ubuntu 16.04 Server

### Requirment Node Worker

Adapun kebutuhan untuk node worker adalah : 

* Ram 2 GB
* Disk 10 GB
* Bridge Network
* IP 192.168.88.101
* Ubuntu 16.04 Server

Dengan requirment diatas, silahkan lakukan instalasi ubuntu server pada 2 vm tersebut yaitu pada vm master dan vm worker. IP yang digunakan disini adalah IP static, sehingga anda perlu melakukan sedikit konfigurasi pada jaringan ubuntu server.

Untuk melakukan instalasi kubernetes, anda wajib melakukan instalasi docker terlebih dahulu, bagi yang belum mengerti docker, silahkan baca artikel [Belajar Docker](https://rizkimufrizal.github.io/belajar-docker/). Hal yang pertama dilakukan adalah lakukan update dan upgrade ubuntu server dengan perintah.

{% highlight bash %}
sudo -s
apt update && apt upgrade -y
{% endhighlight %}

Lalu update certificate dan kebutuhan untuk kubernetes dengan perintah

{% highlight bash %}
apt install apt-transport-https ca-certificates curl software-properties-common -y
{% endhighlight %}

Lalu tambahkan key untuk kubernetes dengan perintah

{% highlight bash %}
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
{% endhighlight %}

Lalu tambahkan repository kubernetes untuk ubuntu dengan perintah

{% highlight bash %}
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
{% endhighlight %}

Lalu lakukan update kembali dengan perintah

{% highlight bash %}
apt update
{% endhighlight %}

Dan lakukan instalasi dengan perintah

{% highlight bash %}
apt install kubelet kubeadm kubernetes-cni -y
{% endhighlight %}

## Setup Kubernetes

Setelah selesai melakukan instalasi kubernetes, tahapan selanjutnya adalah kita akan melakukan setup kubernetes. Setup kubernetes akan dilakukan secara bertahap, yaitu dilakukan pada node master terlebih dahulu kemudian akan dilakukan pada node worker.

### Setup Kubernetes Master

Silahkan login ke node master, lalu aktifkan bridged IPv4 traffic dengan perintah

{% highlight bash %}
sysctl net.bridge.bridge-nf-call-iptables=1
{% endhighlight %}

Lalu jalankan perintah berikut untuk disable swap pada ubuntu

{% highlight bash %}
swapoff -a
{% endhighlight %}

Setelah selesai, jalankan perintah berikut untuk inisialisasi kubernetes pada node master.

{% highlight bash %}
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.88.100 --kubernetes-version stable-1.9
{% endhighlight %}

Jika berhasil maka akan muncul output seperti berikut.

{% highlight bash %}
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token d36cc6.a9d32c295072f336 192.168.88.100:6443 --discovery-token-ca-cert-hash sha256:4a6626451ad1ad54e1c85e93dd26d7bfd66a2d1da4052f04b5bd1f424a4082d4

{% endhighlight %}

Perintah `kubeadm join` nantinya akan kita gunakan untuk melakukan registrasi dari node worker ke node master.

Lalu jalankan perintah berikut agar kubectl dapat diakses tanpa perlu menggunakan user root.

{% highlight bash %}
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
{% endhighlight %}

Setelah selesai, jika kita melakukan setup kubernetes cluster maka dibutuhkan network overlay seperti docker swarm yang dapat menyambung pods yang berbeda node. Jika kita menggunakan kubernetes, maka ada beberapa opsi network pod plugin yang dapat kita gunakan yaitu :

* [Flannel](https://github.com/coreos/flannel)
* [Calico](https://www.projectcalico.org)
* [Canal](https://github.com/projectcalico/canal)
* [Kube Router](https://github.com/cloudnativelabs/kube-router)
* [Romana](http://romana.io/)
* [Weave Net](https://www.weave.works/)

Pada artikel ini, penulis akan menggunakan flannel sebagai network plugin. Untuk melakukan setup flannel pada kubernetes, silahkan jalankan perintah berikut.

{% highlight bash %}
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
{% endhighlight %}

Jika telah selesai, silahkan lakukan pengecekan pods dengan perintah

{% highlight bash %}
kubectl get pods --all-namespaces
{% endhighlight %}

Jika beberapa pods masih dalam proses creating / prosess pull image maka akan muncul seperti berikut.

{% highlight bash %}
NAMESPACE     NAME                             READY     STATUS     RESTARTS   AGE
kube-system   etcd-master                      1/1       Running    0          4s
kube-system   kube-apiserver-master            1/1       Running    3          4s
kube-system   kube-controller-manager-master   1/1       Running    0          4s
kube-system   kube-dns-6f4fd4bdf-47lc6         0/3       Pending    0          7m
kube-system   kube-flannel-ds-j5d75            0/1       Init:0/1   0          19s
kube-system   kube-proxy-gmrfm                 1/1       Running    0          8m
kube-system   kube-scheduler-master            1/1       Running    0          4s
{% endhighlight %}

Jika semua pods telah berjalan maka akan muncul seperti berikut.

{% highlight bash %}
NAMESPACE     NAME                             READY     STATUS    RESTARTS   AGE
kube-system   etcd-master                      1/1       Running   0          7m
kube-system   kube-apiserver-master            1/1       Running   3          7m
kube-system   kube-controller-manager-master   1/1       Running   0          7m
kube-system   kube-dns-6f4fd4bdf-47lc6         3/3       Running   0          16m
kube-system   kube-flannel-ds-j5d75            1/1       Running   0          9m
kube-system   kube-proxy-gmrfm                 1/1       Running   0          16m
kube-system   kube-scheduler-master            1/1       Running   0          7m
{% endhighlight %}

### Setup Kubernetes Worker

Silahkan login ke node worker lalu aktifkan bridged IPv4 traffic dengan perintah

{% highlight bash %}
sysctl net.bridge.bridge-nf-call-iptables=1
{% endhighlight %}

Lalu jalankan perintah berikut untuk disable swap pada ubuntu

{% highlight bash %}
swapoff -a
{% endhighlight %}

Setelah selesai, jalankan perintah berikut untuk inisialisasi kubernetes pada node worker dengan melakukan register ke node master.

{% highlight bash %}
kubeadm join --token d36cc6.a9d32c295072f336 192.168.88.100:6443 --discovery-token-ca-cert-hash sha256:4a6626451ad1ad54e1c85e93dd26d7bfd66a2d1da4052f04b5bd1f424a4082d4
{% endhighlight %}

Setelah selesai, silahkan login kembali ke node master, lalu jalankan perintah berikut untuk melihat node yang berhasil melakukan register.

{% highlight bash %}
kubectl get nodes
{% endhighlight %}

Jika berhasil maka akan muncul seperti berikut.

{% highlight bash %}
NAME      STATUS     ROLES     AGE       VERSION
master    Ready      master    18m       v1.9.6
worker    NotReady   <none>    2s        v1.9.6
{% endhighlight %}

Dan berikut jika kedua node telah siap untuk digunakan.

{% highlight bash %}
NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    23m       v1.9.6
worker    Ready     <none>    5m        v1.9.6
{% endhighlight %}

### Setup Kubernetes Dashboard

Secara default, kubernetes tidak memiliki dashboard akan tetapi kita dapat melakukan setup untuk kubernetes dashboard. Yang pertama kali dilakukan adalah kita akan membuat sebuah file yaitu `admin.yaml` lalu isikan dengan source berikut.

{% highlight yaml %}
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
{% endhighlight %}

Konfigurasi diatas berfungsi untuk user authentication pada kubernetes dashboard. Lalu jalankan perintah berikut untuk menambahkan user tersebut.

{% highlight bash %}
kubectl create -f admin.yaml
{% endhighlight %}

Setelah selesai, silahkan buat sebuah file `dashboard.yaml` lalu isikan source berikut.

{% highlight yaml %}
apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kube-system
type: Opaque

---
# ------------------- Dashboard Service Account ------------------- #

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Role & Role Binding ------------------- #

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
rules:
  # Allow Dashboard to create 'kubernetes-dashboard-key-holder' secret.
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["create"]
  # Allow Dashboard to create 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs"]
  verbs: ["get", "update", "delete"]
  # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["kubernetes-dashboard-settings"]
  verbs: ["get", "update"]
  # Allow Dashboard to get metrics from heapster.
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["heapster"]
  verbs: ["proxy"]
- apiGroups: [""]
  resources: ["services/proxy"]
  resourceNames: ["heapster", "http:heapster:", "https:heapster:"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard-minimal
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Deployment ------------------- #

kind: Deployment
apiVersion: apps/v1beta2
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
      - name: kubernetes-dashboard
        image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.3
        ports:
        - containerPort: 8443
          protocol: TCP
        args:
          - --auto-generate-certificates
          # Uncomment the following line to manually specify Kubernetes API server Host
          # If not specified, Dashboard will attempt to auto discover the API server and connect
          # to it. Uncomment only if the default does not work.
          # - --apiserver-host=http://my-address:port
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
          # Create on-disk volume to store exec logs
        - mountPath: /tmp
          name: tmp-volume
        livenessProbe:
          httpGet:
            scheme: HTTPS
            path: /
            port: 8443
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: kubernetes-dashboard-certs
        secret:
          secretName: kubernetes-dashboard-certs
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

---
# ------------------- Dashboard Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort
{% endhighlight %}

Lalu jalankan dashboard tersebut dengan perintah

{% highlight bash %}
kubectl apply -f dashboard.yaml
{% endhighlight %}

Setelah selesai, tunggu hingga pods tersebut berjalan. Jika pods telah berjalan, silahkan lakukan pengecekan services dengan perintah.

{% highlight bash %}
kubectl get services --all-namespaces
{% endhighlight %}

Dan berikut adalah hasilnya

{% highlight bash %}
NAMESPACE     NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
default       kubernetes             ClusterIP   10.96.0.1       <none>        443/TCP         31m
kube-system   kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP   31m
kube-system   kubernetes-dashboard   NodePort    10.101.10.168   <none>        443:31601/TCP   2m
{% endhighlight %}

Dari konfigurasi diatas, dapat dilihat bahwa kubernetes dashboard jalan pada port `31601`, lalu silahkan akses kubernetes dashboard di `https://<ip>:31601` seperti berikut.

![Screenshot from 2018-03-25 18-36-20.png](../images/Screenshot from 2018-03-25 18-36-20.png)

Lalu pilih menu skip dan akan muncul halaman seperti berikut.

![Screenshot from 2018-03-25 18-37-31.png](../images/Screenshot from 2018-03-25 18-37-31.png)

## Arsitektur Cluster Apache Cassandra Pada Kubernetes

Arsitektur yang akan kita gunakan pada kubernetes tidak jauh berbeda dengan docker swarm. Pada kubernetes, kita akan melakukan deployment container pada sebuah deployment. Setiap 1 deployment akan mewakili 1 cassandra. Berikut adalah arsitektur yang akan digunakan.

![cassandra-kubernetes.png](../images/cassandra-kubernetes.png)

Silahkan buat sebuah file yaml `cassandra-service.yaml` lalu isikan dengan source berikut.

{% highlight yaml %}
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dc-1
  name: cassandra-dc-1
spec:
  ports:
  - name: "9042"
    port: 9042
    targetPort: 9042
  - name: "7000"
    port: 7000
    targetPort: 7000
  selector:
    io.kompose.service: cassandra-dc-1
status:
  loadBalancer: {}

---
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dc-2
  name: cassandra-dc-2
spec:
  ports:
  - name: "9042"
    port: 9042
    targetPort: 9042
  - name: "7000"
    port: 7000
    targetPort: 7000
  selector:
    io.kompose.service: cassandra-dc-2
status:
  loadBalancer: {}

---
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dc-3
  name: cassandra-dc-3
spec:
  ports:
  - name: "9042"
    port: 9042
    targetPort: 9042
  - name: "7000"
    port: 7000
    targetPort: 7000
  selector:
    io.kompose.service: cassandra-dc-3
status:
  loadBalancer: {}

---
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dr-1
  name: cassandra-dr-1
spec:
  ports:
  - name: "9042"
    port: 9042
    targetPort: 9042
  - name: "7000"
    port: 7000
    targetPort: 7000
  selector:
    io.kompose.service: cassandra-dr-1
status:
  loadBalancer: {}

---
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dr-2
  name: cassandra-dr-2
spec:
  ports:
  - name: "9042"
    port: 9042
    targetPort: 9042
  - name: "7000"
    port: 7000
    targetPort: 7000
  selector:
    io.kompose.service: cassandra-dr-2
status:
  loadBalancer: {}

---
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dr-3
  name: cassandra-dr-3
spec:
  ports:
  - name: "9042"
    port: 9042
    targetPort: 9042
  - name: "7000"
    port: 7000
    targetPort: 7000
  selector:
    io.kompose.service: cassandra-dr-3
status:
  loadBalancer: {}
{% endhighlight %}

Kemudian kembali ke menu dashboard, pilih menu create lalu isikan source diatas pada form seperti berikut.

![Screenshot from 2018-03-25 18-47-06.png](../images/Screenshot from 2018-03-25 18-47-06.png)

Silahkan buat sebuah file yaml `cassandra-deployment.yaml` lalu isikan dengan source berikut.

{% highlight yaml %}
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dc-1
  name: cassandra-dc-1
spec:
  replicas: 1
  minReadySeconds: 60
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: cassandra-dc-1
    spec:
      containers:
      - args:
        - bash
        - -c
        - 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 0; fi && /docker-entrypoint.sh cassandra -f'
        image: cassandra:latest
        name: cassandra-dc-1
        imagePullPolicy: IfNotPresent
        env:
        - name: CASSANDRA_CLUSTER_NAME
          value: CassandraCluster
        - name: CASSANDRA_BROADCAST_ADDRESS
          value: cassandra-dc-1
        - name: CASSANDRA_SEEDS
          value: cassandra-dc-1,cassandra-dr-1
        - name: CASSANDRA_DC
          value: DC
        - name: CASSANDRA_RACK
          value: RACK1
        - name: CASSANDRA_ENDPOINT_SNITCH
          value: GossipingPropertyFileSnitch
        - name: MAX_HEAP_SIZE
          value: 50m
        - name: HEAP_NEWSIZE
          value: 10m
        resources: {}
      restartPolicy: Always
status: {}

---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dc-2
  name: cassandra-dc-2
spec:
  replicas: 1
  minReadySeconds: 60
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: cassandra-dc-2
    spec:
      containers:
      - args:
        - bash
        - -c
        - 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 120; fi && /docker-entrypoint.sh cassandra -f'
        image: cassandra:latest
        name: cassandra-dc-2
        imagePullPolicy: IfNotPresent
        env:
        - name: CASSANDRA_CLUSTER_NAME
          value: CassandraCluster
        - name: CASSANDRA_BROADCAST_ADDRESS
          value: cassandra-dc-2
        - name: CASSANDRA_SEEDS
          value: cassandra-dc-1,cassandra-dr-1
        - name: CASSANDRA_DC
          value: DC
        - name: CASSANDRA_RACK
          value: RACK2
        - name: CASSANDRA_ENDPOINT_SNITCH
          value: GossipingPropertyFileSnitch
        - name: MAX_HEAP_SIZE
          value: 50m
        - name: HEAP_NEWSIZE
          value: 10m
        resources: {}
      restartPolicy: Always
status: {}

---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dc-3
  name: cassandra-dc-3
spec:
  replicas: 1
  minReadySeconds: 60
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: cassandra-dc-3
    spec:
      containers:
      - args:
        - bash
        - -c
        - 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 240; fi && /docker-entrypoint.sh cassandra -f'
        image: cassandra:latest
        name: cassandra-dc-3
        imagePullPolicy: IfNotPresent
        env:
        - name: CASSANDRA_CLUSTER_NAME
          value: CassandraCluster
        - name: CASSANDRA_BROADCAST_ADDRESS
          value: cassandra-dc-3
        - name: CASSANDRA_SEEDS
          value: cassandra-dc-1,cassandra-dr-1
        - name: CASSANDRA_DC
          value: DC
        - name: CASSANDRA_RACK
          value: RACK3
        - name: CASSANDRA_ENDPOINT_SNITCH
          value: GossipingPropertyFileSnitch
        - name: MAX_HEAP_SIZE
          value: 50m
        - name: HEAP_NEWSIZE
          value: 10m
        resources: {}
      restartPolicy: Always
status: {}

---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dr-1
  name: cassandra-dr-1
spec:
  replicas: 1
  minReadySeconds: 60
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: cassandra-dr-1
    spec:
      containers:
      - args:
        - bash
        - -c
        - 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 60; fi && /docker-entrypoint.sh cassandra -f'
        image: cassandra:latest
        name: cassandra-dr-1
        imagePullPolicy: IfNotPresent
        env:
        - name: CASSANDRA_CLUSTER_NAME
          value: CassandraCluster
        - name: CASSANDRA_BROADCAST_ADDRESS
          value: cassandra-dr-1
        - name: CASSANDRA_SEEDS
          value: cassandra-dr-1,cassandra-dc-1
        - name: CASSANDRA_DC
          value: DR
        - name: CASSANDRA_RACK
          value: RACK1
        - name: CASSANDRA_ENDPOINT_SNITCH
          value: GossipingPropertyFileSnitch
        - name: MAX_HEAP_SIZE
          value: 50m
        - name: HEAP_NEWSIZE
          value: 10m
        resources: {}
      restartPolicy: Always
status: {}

---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dr-2
  name: cassandra-dr-2
spec:
  replicas: 1
  minReadySeconds: 60
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: cassandra-dr-2
    spec:
      containers:
      - args:
        - bash
        - -c
        - 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 180; fi && /docker-entrypoint.sh cassandra -f'
        image: cassandra:latest
        name: cassandra-dr-2
        imagePullPolicy: IfNotPresent
        env:
        - name: CASSANDRA_CLUSTER_NAME
          value: CassandraCluster
        - name: CASSANDRA_BROADCAST_ADDRESS
          value: cassandra-dr-2
        - name: CASSANDRA_SEEDS
          value: cassandra-dr-1,cassandra-dc-1
        - name: CASSANDRA_DC
          value: DR
        - name: CASSANDRA_RACK
          value: RACK2
        - name: CASSANDRA_ENDPOINT_SNITCH
          value: GossipingPropertyFileSnitch
        - name: MAX_HEAP_SIZE
          value: 50m
        - name: HEAP_NEWSIZE
          value: 10m
        resources: {}
      restartPolicy: Always
status: {}

---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.7.0 (767ab4b)
  creationTimestamp: null
  labels:
    io.kompose.service: cassandra-dr-3
  name: cassandra-dr-3
spec:
  replicas: 1
  minReadySeconds: 60
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: cassandra-dr-3
    spec:
      containers:
      - args:
        - bash
        - -c
        - 'if [ -z "$$(ls -A /var/lib/cassandra/)" ] ; then sleep 300; fi && /docker-entrypoint.sh cassandra -f'
        image: cassandra:latest
        name: cassandra-dr-3
        imagePullPolicy: IfNotPresent
        env:
        - name: CASSANDRA_CLUSTER_NAME
          value: CassandraCluster
        - name: CASSANDRA_BROADCAST_ADDRESS
          value: cassandra-dr-3
        - name: CASSANDRA_SEEDS
          value: cassandra-dr-1,cassandra-dc-1
        - name: CASSANDRA_DC
          value: DR
        - name: CASSANDRA_RACK
          value: RACK3
        - name: CASSANDRA_ENDPOINT_SNITCH
          value: GossipingPropertyFileSnitch
        - name: MAX_HEAP_SIZE
          value: 50m
        - name: HEAP_NEWSIZE
          value: 10m
        resources: {}
      restartPolicy: Always
status: {}
{% endhighlight %}

Kemudian kembali ke menu dashboard, pilih menu create lalu isikan source diatas pada form, Berikut adalah jika pods dalam keadaan creating atau pull images.

![Screenshot from 2018-03-25 19-21-35.png](../images/Screenshot from 2018-03-25 19-21-35.png)

Dan berikut ketika seluruh pods telah berjalan.

![Screenshot from 2018-03-25 19-27-18.png](../images/Screenshot from 2018-03-25 19-27-18.png)

Lalu pada kubernetes dashboard, silahkan klik di salah satu pods, misalnya penulis ingin memilih pods `cassandra-dc-1-<code>`, contohnya `cassandra-dc-1-8f6fdf494-w247g`, maka akan muncul halaman seperti berikut.

![Screenshot from 2018-03-25 19-44-42.png](../images/Screenshot from 2018-03-25 19-44-42.png)

Lalu pilih menu `exec` dan akan muncul halaman seperti berikut.

![Screenshot from 2018-03-25 19-44-59.png](../images/Screenshot from 2018-03-25 19-44-59.png)

Pada halaman tersebut, kita dapat mengakses terminal yang terdapat pada pods tersebut. Lalu jalankan perintah berikut untuk melihat topology cassandra cluster yang telah terbentuk.

{% highlight bash %}
nodetool status
{% endhighlight %}

Jika berhasil maka akan muncul seperti ini

![Screenshot from 2018-03-25 19-46-15.png](../images/Screenshot from 2018-03-25 19-46-15.png)

Atau jika dalam bentuk bash

{% highlight bash %}
root@cassandra-dc-1-8f6fdf494-w247g:/# nodetool status
Datacenter: DC
==============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address         Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.106.21.202   75.01 KiB  256          32.4%             f15606c4-e7e7-4dd3-81fc-5a072ac6b3d8  RACK3
UN  10.108.237.0    110.2 KiB  256          31.5%             a907cb28-3e39-4b0b-bce4-052e16485e99  RACK1
UN  10.108.167.162  74.97 KiB  256          33.7%             b9b8feb1-6ebc-442a-a386-44b8c9eea3da  RACK2
Datacenter: DR
==============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address         Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.109.153.57   74.88 KiB  256          32.8%             f8d3f2b1-ebfc-4826-b1c9-39bc37e9c9bb  RACK1
UN  10.107.250.188  69.94 KiB  256          35.1%             00d659f4-d473-492f-afed-fc46ce1c0402  RACK2
UN  10.111.237.50   151.11 KiB  256         34.5%             7471c743-a2e1-4eb8-96f7-ad852d262e43  RACK3
{% endhighlight %}

Sekian artikel mengenai Instalasi Dan Setup Kubernetes Cluster dan terima kasih :).