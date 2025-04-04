## 🌞 Allumez les VMs et effectuez la conf élémentaire :

```
[it5@efrei-xmg4agau1 ~]# ping kvm1.one
PING kvm1.one (10.3.1.11) 56(84) bytes of data.
64 bytes from kvm1.one (10.3.1.11): icmp_seq=1 ttl=64 time=0.577 ms
64 bytes from kvm1.one (10.3.1.11): icmp_seq=2 ttl=64 time=0.396 ms

[it5@efrei-xmg4agau1 ~]# ping kvm2.one
PING kvm2.one (10.3.1.12) 56(84) bytes of data.
64 bytes from kvm2.one (10.3.1.12): icmp_seq=1 ttl=64 time=0.644 ms
64 bytes from kvm2.one (10.3.1.12): icmp_seq=2 ttl=64 time=0.376 ms
64 bytes from kvm2.one (10.3.1.12): icmp_seq=3 ttl=64 time=0.308 ms

```
# I - Frontend
## 🌞 Installer un serveur MySQL
```
[it5@frontend ~]# rpm -i mysql80-community-release-el9-5.noarch.rpm
warning: mysql80-community-release-el9-5.noarch.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY

[it5@frontend ~]# dnf install mysql-community-server
Last metadata expiration check: 0:00:24 ago on Fri Mar 21 10:18:47 2025.
Dependencies resolved.
```

## 🌞 Démarrer le serveur MySQL
```
[it5@frontend ~]# sudo systemctl start mysqld
[it5@frontend ~]# sudo systemctl enable mysqld
```

## 🌞 Setup MySQL
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.41

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER 'it5'@'localhost' IDENTIFIED BY 'Password123!';
Query OK, 0 rows affected (0.01 sec)

mysql> CREATE USER 'oneadmin' IDENTIFIED BY 'Password123!';
Query OK, 0 rows affected (0.01 sec)

mysql> CREATE DATABASE opennebula;
Query OK, 1 row affected (0.01 sec)

mysql> GRANT ALL PRIVILEGES ON opennebula.* TO 'oneadmin';
Query OK, 0 rows affected (0.00 sec)

mysql> SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)
```

# B. OpenNebula
## 🌞 Ajouter les dépôts Open Nebula
```
[it5@frontend yum.repos.d]# dnf makecache -y
MySQL 8.0 Community Server                                                               17 kB/s | 2.6 kB     00:00
MySQL Connectors Community                                                               24 kB/s | 2.6 kB     00:00
MySQL Tools Community                                                                    28 kB/s | 2.6 kB     00:00
OpenNebula Community Edition                                                            2.3 kB/s | 833  B     00:00
OpenNebu##la Community Edition                                                             18 kB/s | 3.1 kB     00:00
Importing GPG key 0x906DC27C:
...
```
## 🌞 Configuration OpenNebula
```
DB = [ BACKEND = "mysql",
       SERVER  = "localhost",
       PORT    = 0,
       USER    = "oneadmin",
       PASSWD  = "O_O",
       DB_NAME = "opennebula",
       CONNECTIONS = 25,
       COMPARE_BINARY = "no" ]

```

## 🌞 Créer un user pour se log sur la WebUI OpenNebula
```
sudo su - oneadmin
vi /var/lib/one/.one/one_auth
user:password
```

## 🌞 Démarrer les services OpenNebula
```
[it5@frontend ~]# sudo systemctl start opennebula
[it5@frontend ~]# sudo systemctl enable opennebula
Created symlink /etc/systemd/system/multi-user.target.wants/opennebula.service → /usr/lib/systemd/system/opennebula.service.
[it5@frontend ~]# sudo systemctl enable opennebula-sunstone
Created symlink /etc/systemd/system/multi-user.target.wants/opennebula-sunstone.service → /usr/lib/systemd/system/opennebula-sunstone.service.
[it5@frontend ~]# sudo systemctl start opennebula-sunstone
```

# C. Conf système
## 🌞 Ouverture firewall
```
firewall-cmd --add-port=9869/tcp --add-port=22/tcp --add-port=2633/tcp --add-port=4124/tcp --add-port=4124/udp --add-port=29876/tcp --permanent && firewall-cmd --reload
```

# II. Noeuds KVM
# A. KVM
## 🌞 Ajouter des dépôts supplémentaires
```
dnf install -y epel-release
```

## 🌞 Installer KVM
```
dnf install opennebula-node-kvm
```

## 🌞 Démarrer le service libvirtd
```
systemctl start libvirtd
systemctl enable libvirtd
```
# B. Système
## 🌞 Ouverture firewall
```
[it5@kvm1 yum.repos.d]# firewall-cmd --add-port=22/tcp --add-port=8472/udp --permanent
success
[it5@kvm1 yum.repos.d]# firewall-cmd --reload
```
## 🌞 Handle SSH
```
[oneadmin@frontend ~]$ ssh-keyscan 10.3.1.12 >> ~/.ssh/known_hosts
[oneadmin@frontend ~]$ cat .ssh/id_rsa.pub
[oneadmin@kvm1 .ssh]$ vi authorized_keys
[oneadmin@frontend ~]$ ssh oneadmin@10.3.1.11
Last login: Fri Mar 21 12:26:06 2025 from 10.3.1.10
```

## 🌞 Créer et configurer le bridge Linux
```
PS C:\Users\benoit> ssh kvm1.one
benoit@kvm1.one's password:
PS C:\Users\benoit> ssh it5@kvm1.one
it5@kvm1.one's password:
Last login: Fri Mar 21 22:09:56 2025
[it5@kvm1 ~]# ip link add name vxlan_bridge type bridge
[it5@kvm1 ~]# ip link set dev vxlan_bridge up
[it5@kvm1 ~]# ip addr add 10.220.220.201/24 dev vxlan_bridge
[it5@kvm1 ~]# firewall-cmd --add-interface=vxlan_bridge --zone=public --permanent
success
[it5@kvm1 ~]# firewall-cmd --add-masquerade --permanent
fisuccess
[it5@kvm1 ~]# firewall-cmd reload
usage: 'firewall-cmd --help' for usage information or see firewall-cmd(1) man page
firewall-cmd: error: unrecognized arguments: reload
[it5@kvm1 ~]# firewall-cmd --reload
success
[it5@kvm1 ~]#

```
## 🌞 Setup de kvm2.one, à l'identique de kvm1.one excepté : blabla

### VM sur le deuxième noeud
```
[oneadmin@frontend ~]$ ssh -J kvm2.one it5@10.220.220.2
The authenticity of host '10.220.220.2 (<no hostip for proxy command>)' can't be established.
ED25519 key fingerprint is SHA256:1h3xhqpWsgqOBiWJcuOVoMiEBRdJfF5sy7K5Vb9EzRo.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.220.220.2' (ED25519) to the list of known hosts.
[it5@localhost ~]# 
```
### 3. Connectivité entre les VMs
```
depuis la vm sur kvm1
[it5@localhost ~]# ping 10.220.220.2
PING 10.220.220.2 (10.220.220.2) 56(84) bytes of data.
64 bytes from 10.220.220.2: icmp_seq=1 ttl=64 time=6.92 ms
64 bytes from 10.220.220.2: icmp_seq=2 ttl=64 time=6.45 ms
64 bytes from 10.220.220.2: icmp_seq=3 ttl=64 time=2.70 ms

depuis la vm sur kvm2
[it5@localhost ~]# ping 10.220.220.1
PING 10.220.220.1 (10.220.220.1) 56(84) bytes of data.
64 bytes from 10.220.220.1: icmp_seq=1 ttl=64 time=2.97 ms
64 bytes from 10.220.220.1: icmp_seq=2 ttl=64 time=4.73 ms
64 bytes from 10.220.220.1: icmp_seq=3 ttl=64 time=4.17 ms
```
