
## Installation

Отключаем swap и firewall. На паблик-серверах мы этого, конечно, делать не будем.

```bash
sudo swapoff -a
sudo vim /etc/fstab
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```

Отключаем SELinux:
```bash
sudo vim /etc/sysconfig/selinux
```

Прописываем имена и адреса всех нод кластера в хосты:
```bash
sudo vim /etc/hosts
```

Пример моего файла хостов с мастер-ноды, обязательно прописываем полное имя с локальным доменом, а так же 127.0.0.1 для каждой ноды:
```bash
127.0.0.1   master.zamunda.local        master
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.27.64.210 kuber-node1.zamunda.local kuber-node1
172.27.64.212 kuber-node2.zamunda.local kuber-node2
172.27.64.213 kuber-node3.zamunda.local kuber-node3
172.27.64.215 master.zamunda.local      master
```

Настраиваем resolv.conf:
```bash
sudo vim /etc/resolv/conf
````
Пример моего resolv.conf. Указан ip-адрес внутреннего DNS, обязательно прописываем на нем полные хостнеймы ВМ с указанием локального домена
```bash
search zamunda.local
nameserver 172.27.64.20
nameserver 172.27.64.21
```
Настраиваем модули для containerd

```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

Настраиваем параметры ядра и применяем настройки:
```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF 

sudo sysctl --system
```

Добавляем репозитории и устанавливаем containerd:
```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y containerd.io
````

Настраиваем containerd:
```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/            SystemdCgroup = false/            SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl restart containerd
```
Добавляем репозитории кубернетеса:
```bash
sudo cat > tee /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
````

Устанавливаем компоненты кубернетеса:
```bash
sudo yum update
sudo yum -y install -y kubelet kubeadm kubectl
````

Инициализируем мастер, указываем полный хостнейм с доменом:
```bash
sudo kubeadm init --control-plane-endpoint=master.zamunda.local
````


