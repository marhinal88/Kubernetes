
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

Пример моего файла хостов с мастер-ноды:
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


