# Part I : Programmatic approach

- [Part I : Programmatic approach](#part-i--programmatic-approach)
- [I. Premiers pas](#i-premiers-pas)
- [II. Un ptit LAN](#ii-un-ptit-lan)

# I. Premiers pas

🌞 **Créez une VM depuis le Azure CLI**

~~~
PS C:\Users\benoi> az group create --name meo --location eastus
{
  "id": "/subscriptions/22df3f3e-3b12-4e67-a5fa-b5a47a09b315/resourceGroups/meo",
  "location": "eastus",
  "managedBy": null,
  "name": "meo",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
PS C:\Users\benoi> az vm create -g meo -n super_vm --image Ubuntu2204 --admin-username it4 --ssh-key-values "C:\Users\benoi\.ssh\id_rsa.pub"
{
  "fqdns": "",
  "id": "/subscriptions/22df3f3e-3b12-4e67-a5fa-b5a47a09b315/resourceGroups/meo/providers/Microsoft.Compute/virtualMachines/super_vm",
  "location": "eastus",
  "macAddress": "7C-1E-52-7E-3B-29",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "172.174.66.56",
  "resourceGroup": "meo",
  "zones": ""
}
PS C:\Users\benoi>
~~~

🌞 **Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique.**

~~~
PS C:\Users\benoi> ssh -i "C:\Users\benoi\.ssh\id_rsa" it4@172.174.66.56
~~~


- une fois connecté, observez :
  - **la présence du service `walinuxagent`**

~~~
it4@supervm:~$ systemctl status walinuxagent
● walinuxagent.service - Azure Linux Agent
     Loaded: loaded (/lib/systemd/system/walinuxagent.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2025-03-18 08:37:34 UTC; 5min ago
   Main PID: 761 (python3)
      Tasks: 6 (limit: 4022)
     Memory: 40.3M
        CPU: 1.906s
     CGroup: /system.slice/walinuxagent.service
             ├─ 761 /usr/bin/python3 -u /usr/sbin/waagent -daemon
             └─1311 python3 -u bin/WALinuxAgent-2.12.0.2-py3.9.egg -run-exthandlers

Mar 18 08:37:42 supervm python3[1311]:     pkts      bytes target     prot opt in     out     source               dest>
Mar 18 08:37:42 supervm python3[1311]:        0        0 ACCEPT     tcp  --  *      *       0.0.0.0/0            168.63>
Mar 18 08:37:42 supervm python3[1311]:       80    11165 ACCEPT     tcp  --  *      *       0.0.0.0/0            168.63>
Mar 18 08:37:42 supervm python3[1311]:        0        0 DROP       tcp  --  *      *       0.0.0.0/0            168.63>
Mar 18 08:37:42 supervm python3[1311]: 2025-03-18T08:37:42.526982Z INFO ExtHandler ExtHandler
Mar 18 08:37:42 supervm python3[1311]: 2025-03-18T08:37:42.527327Z INFO ExtHandler ExtHandler ProcessExtensionsGoalStat>
Mar 18 08:37:42 supervm python3[1311]: 2025-03-18T08:37:42.528288Z INFO ExtHandler ExtHandler No extension handlers fou>
Mar 18 08:37:42 supervm python3[1311]: 2025-03-18T08:37:42.528972Z INFO ExtHandler ExtHandler ProcessExtensionsGoalStat>
Mar 18 08:37:42 supervm python3[1311]: 2025-03-18T08:37:42.566613Z INFO ExtHandler ExtHandler Looking for existing remo>
Mar 18 08:37:42 supervm python3[1311]: 2025-03-18T08:37:42.576012Z INFO ExtHandler ExtHandler [HEARTBEAT] Agent WALinux>
lines 1-21/21 (END)
~~~


  - **la présence du service `cloud-init`**

~~~
it4@supervm:~$ cloud-init status
status: done
~~~

# II. Un ptit LAN

🌞 **Créez deux VMs depuis le Azure CLI**

~~~
it5@supervm2:~$ ping 10.0.0.4
PING 10.0.0.4 (10.0.0.4) 56(84) bytes of data.
64 bytes from 10.0.0.4: icmp_seq=1 ttl=64 time=10.6 ms
64 bytes from 10.0.0.4: icmp_seq=2 ttl=64 time=10.4 ms
64 bytes from 10.0.0.4: icmp_seq=3 ttl=64 time=8.23 ms
^C
--- 10.0.0.4 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 8.225/9.734/10.626/1.072 ms
it5@supervm2:~$
~~~


# Part II : cloud-init

- [Part II : cloud-init](#part-ii--cloud-init)
  - [1. Intro](#1-intro)
  - [2. Gooooo](#2-gooooo)
  - [3. Write your own](#3-write-your-own)

## 1. Intro

## 2. Gooooo


🌞 **Tester `cloud-init`**

~~~
PS C:\Users\benoi> az vm create -g meo -n the_vm --image Ubuntu2204 --admin-username it5 --custom-data C:\Users\benoi\Documents\cloud\cloud-init.txt --vnet-name super_vmVNET --subnet super_vmSubnet --private-ip-address 10.0.0.6
{
  "fqdns": "",
  "id": "/subscriptions/22df3f3e-3b12-4e67-a5fa-b5a47a09b315/resourceGroups/meo/providers/Microsoft.Compute/virtualMachines/the_vm",
  "location": "eastus",
  "macAddress": "00-0D-3A-9A-0C-51",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.6",
  "publicIpAddress": "172.190.243.237",
  "resourceGroup": "meo",
  "zones": ""
}
PS C:\Users\benoi>
~~~



🌞 **Vérifier que `cloud-init` a bien fonctionné**

connection direct tout est bon : 

~~~
PS C:\Users\benoi> ssh it5@172.190.243.237
The authenticity of host '172.190.243.237 (172.190.243.237)' can't be established.
ED25519 key fingerprint is SHA256:JvlGljdcWoqCOta4FSs9JkixOdxgPgM5gLteNEBZjSI.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.190.243.237' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Mar 18 09:58:31 UTC 2025

  System load:  0.17              Processes:             106
  Usage of /:   5.2% of 28.89GB   Users logged in:       0
  Memory usage: 7%                IPv4 address for eth0: 10.0.0.6
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

it5@thevm:~$
~~~


## 3. Write your own

🌞 **Utilisez `cloud-init` pour préconfigurer la VM :**

fichier cloud init : 

~~~
#cloud-config
users:
  - default
  - name: it5
    password: $6$tHrUSv1wTaDEcU1q$1PBsTKihivlzM1EXW8.8.tlwnrGnpfaTVDt8nTvUW/bQAJFTu371dNPLGfb1Xc9fSzBNvioUFP/oZ1RkyUxyY.
    sudo: true
    shell: /bin/bash
    groups: sudo, docker
    ssh_authorized_keys:
      - ssh-rsa 
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQD4nKTe5oQFsqr6BY0wALZTy2m3Rq2HV6cMyyrkq5x8C/jglxLSrLAcp9eMDpVMdptud7DBKCuiuUPTfB5bq7smR2V+FOXQlZfrBIbTW9gCXFVZaqQRQoshTa9pPFxELNODxt0HpwC5qZ9PsdbXe+b0kg/RyrSV0MxQEi2TcSlrLXbKU0ej85YvjIIyqsMkGedhg5E9u+wLdkZ/wMszwCOvBFKmkHcxkOvX+8Zm9lR24HDugt9vURXIglPYA1T7J6/vJvLIZVgPcbdWwHL/VxSqVs/eFPk13GCNgaiifmXa8OlXmnGKV7CHzyRPUugpUTMZtGXhy05skQXV0BByQ+D3uEj9kb8DrzNX5vk401SXJpsP1kriWnXXofBHwLwXeAnRZu7+AhiGK5peP83iXxcO0f89/JoYu74jctbCiLkMC1+7401Byi7YCYE9xxR/kO+9nB3mCXDOhavgw7QA/X9fXXdQ7Xpm+ETsVfPsOZNTBDtjoF/eaROpmqsXzgCg/4kAA2yW2pJBNlBhEySMSHW6zQAWT9fCjFCmR6Ussd9UIq/r+fnNT1JF0g7YRPjXafhDnVDv5lHJYowRS0cmmafU46xkiZHF8lbf5p0CL24TaJcQ+vhAv1bZWcTeg2HnwVJJfuqEAL2VUUzA0DtVaSmAmCZQJb6tSgBDeicDB4TMVw==

apt:
  sources:
    docker.list:
      source: deb [arch=amd64] https://download.docker.com/linux/ubuntu $RELEASE stable
      keyid: 9DC858229FC7DD38854AE2D88D81803C0EBFCD88

packages:
  - docker-ce
  - docker-ce-cli

runcmd :
  - docker pull alpine:latest

~~~

A peine connecter en ssh on test :

~~~
PS C:\Users\benoi> ssh it5@74.235.20.245
The authenticity of host '74.235.20.245 (74.235.20.245)' can't be established.
ED25519 key fingerprint is SHA256:XpQ2pUl08l53A5lAMUaXlWG/Pj1BrYw2lNzuW5pWV6k.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '74.235.20.245' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Mar 18 10:37:13 UTC 2025

  System load:  0.67              Processes:             114
  Usage of /:   7.8% of 28.89GB   Users logged in:       0
  Memory usage: 12%               IPv4 address for eth0: 10.0.0.5
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

14 updates can be applied immediately.
8 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

it5@supervmwithdocker:~$ sudo docker --version
Docker version 28.0.1, build 068a01e
it5@supervmwithdocker:~$
~~~


# Part III : Terraform


🌞 **Constater le déploiement**

~~~

PS C:\Users\benoi> az vm list -o table
Name            ResourceGroup          Location    Zones
--------------  ---------------------  ----------  -------
tp2magueule-vm  TP2MAGUEULE-RESOURCES  westeurope
~~~


## 3. Do it yourself

🌞 **Créer un *plan Terraform* avec les contraintes suivantes**

- `node1`
  - Ubuntu 22.04
  - 1 IP Publique
  - 1 IP Privée
- `node2`
  - Ubuntu 22.04
  - 1 IP Privée
- les IPs privées doivent permettre aux deux machines de se `ping`

ping entre node 1 et node 2 : 

~~~
it5@onavance-vm:~$ ping 10.0.2.5
PING 10.0.2.5 (10.0.2.5) 56(84) bytes of data.
64 bytes from 10.0.2.5: icmp_seq=1 ttl=64 time=0.028 ms
64 bytes from 10.0.2.5: icmp_seq=2 ttl=64 time=0.046 ms
64 bytes from 10.0.2.5: icmp_seq=3 ttl=64 time=0.045 ms
64 bytes from 10.0.2.5: icmp_seq=4 ttl=64 time=0.046 ms
^C
--- 10.0.2.5 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3078ms
rtt min/avg/max/mdev = 0.028/0.041/0.046/0.007 ms
it5@onavance-vm:~$
~~~

et connection to node2 via ssh -J

~~~
PS C:\Users\benoi\Documents\terra> ssh -J it5@13.81.115.58 it5@10.0.2.5
The authenticity of host '10.0.2.5 (<no hostip for proxy command>)' can't be established.
ED25519 key fingerprint is SHA256:JtnzwXWNC8xNi+nru/e+Upph9mHLFffp32dgRxYRqpo.
This host key is known by the following other names/addresses:
    C:\Users\benoi/.ssh/known_hosts:16: 13.81.115.58
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.2.5' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Mar 19 14:57:35 UTC 2025

  System load:  0.05              Processes:             121
  Usage of /:   5.4% of 28.89GB   Users logged in:       0
  Memory usage: 8%                IPv4 address for eth0: 10.0.2.4
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
New release '24.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Wed Mar 19 14:47:25 2025 from 90.47.3.222
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

it5@onavance-vm:~$
~~~


## 4. cloud-iniiiiiiiiiiiiit

### A. Un premier tf + cloud-init

🌞 **Intégrer la gestion de `cloud-init`**


main.tf

~~~
provider "azurerm" {
  features {}
  subscription_id = "22df3f3e-3b12-4e67-a5fa-b5a47a09b315"
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space        = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_network_interface" "main" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_linux_virtual_machine" "main" {
  name                            = "${var.prefix}-vm"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_F2"
  admin_username                  = "it5"
  network_interface_ids = [
    azurerm_network_interface.main.id,
  ]
  admin_ssh_key {
    username   = "it5"
    public_key = file("C:\\Users\\benoi\\.ssh\\id_rsa.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }

  custom_data = file("C:\\Users\\benoi\\Documents\\terra\\cloud-init.txt")
}
~~~

cloud-init.txt

~~~

users:
  - default
  - name: it10
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQD4nKTe5oQFsqr6BY0wALZTy2m3Rq2HV6cMyyrkq5x8C/jglxLSrLAcp9eMDpVMdptud7DBKCuiuUPTfB5bq7smR2V+FOXQlZfrBIbTW9gCXFVZaqQRQoshTa9pPFxELNODxt0HpwC5qZ9PsdbXe+b0kg/RyrSV0MxQEi2TcSlrLXbKU0ej85YvjIIyqsMkGedhg5E9u+wLdkZ/wMszwCOvBFKmkHcxkOvX+8Zm9lR24HDugt9vURXIglPYA1T7J6/vJvLIZVgPcbdWwHL/VxSqVs/eFPk13GCNgaiifmXa8OlXmnGKV7CHzyRPUugpUTMZtGXhy05skQXV0BByQ+D3uEj9kb8DrzNX5vk401SXJpsP1kriWnXXofBHwLwXeAnRZu7+AhiGK5peP83iXxcO0f89/JoYu74jctbCiLkMC1+7401Byi7YCYE9xxR/kO+9nB3mCXDOhavgw7QA/X9fXXdQ7Xpm+ETsVfPsOZNTBDtjoF/eaROpmqsXzgCg/4kAA2yW2pJBNlBhEySMSHW6zQAWT9fCjFCmR6Ussd9UIq/r+fnNT1JF0g7YRPjXafhDnVDv5lHJYowRS0cmmafU46xkiZHF8lbf5p0CL24TaJcQ+vhAv1bZWcTeg2HnwVJJfuqEAL2VUUzA0DtVaSmAmCZQJb6tSgBDeicDB4TMVw== 
    groups: [docker]
    passwd: "$6$rounds=4096$randomsalt$hashedpassword"

packages:
  - docker.io
  - docker-ce-cli

runcmd:
  - sudo docker pull alpine:latest

~~~



### B. Go further

🌞 **Moar `cloud-init` and Terraform configuration**

cloud-init 

~~~
#cloud-config
users:
  - default
  - name: it10
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQD4nKTe5oQFsqr6BY0wALZTy2m3Rq2HV6cMyyrkq5x8C/jglxLSrLAcp9eMDpVMdptud7DBKCuiuUPTfB5bq7smR2V+FOXQlZfrBIbTW9gCXFVZaqQRQoshTa9pPFxELNODxt0HpwC5qZ9PsdbXe+b0kg/RyrSV0MxQEi2TcSlrLXbKU0ej85YvjIIyqsMkGedhg5E9u+wLdkZ/wMszwCOvBFKmkHcxkOvX+8Zm9lR24HDugt9vURXIglPYA1T7J6/vJvLIZVgPcbdWwHL/VxSqVs/eFPk13GCNgaiifmXa8OlXmnGKV7CHzyRPUugpUTMZtGXhy05skQXV0BByQ+D3uEj9kb8DrzNX5vk401SXJpsP1kriWnXXofBHwLwXeAnRZu7+AhiGK5peP83iXxcO0f89/JoYu74jctbCiLkMC1+7401Byi7YCYE9xxR/kO+9nB3mCXDOhavgw7QA/X9fXXdQ7Xpm+ETsVfPsOZNTBDtjoF/eaROpmqsXzgCg/4kAA2yW2pJBNlBhEySMSHW6zQAWT9fCjFCmR6Ussd9UIq/r+fnNT1JF0g7YRPjXafhDnVDv5lHJYowRS0cmmafU46xkiZHF8lbf5p0CL24TaJcQ+vhAv1bZWcTeg2HnwVJJfuqEAL2VUUzA0DtVaSmAmCZQJb6tSgBDeicDB4TMVw== benoi@athena
    groups: [docker]
    passwd: "$6$rounds=4096$randomsalt$hashedpassword"

packages:
  - docker.io
  - docker-compose

write_files:
  - path: /opt/wikijs/docker-compose.yml
    owner: root:root
    permissions: '0644'
    content: |
      version: '3'
      services:
        db:
          image: postgres:15-alpine
          environment:
            POSTGRES_DB: wiki
            POSTGRES_PASSWORD: wikijsrocks
            POSTGRES_USER: wikijs
          logging:
            driver: "none"
          restart: unless-stopped
          volumes:
            - db-data:/var/lib/postgresql/data
        wikijs:
          image: ghcr.io/requarks/wiki:2
          restart: always
          ports:
            - "10101:3000"  # Port redirigé de 3000 à 10101
          volumes:
            - /opt/wikijs/data:/wiki/data

runcmd:
  - sudo systemctl restart docker
  - cd /opt/wikijs && sudo docker-compose up -d

~~~

main.tf : 

~~~
provider "azurerm" {
  features {}
  subscription_id = "22df3f3e-3b12-4e67-a5fa-b5a47a09b315"
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space        = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "main" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}

resource "azurerm_network_interface" "internal" {
  name                = "${var.prefix}-nic2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = azurerm_network_interface.main.private_ip_address
  }
}

resource "azurerm_network_security_group" "wikijs" {
  name                = "WikiJS"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "wikijs"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "10101"
    destination_address_prefix = azurerm_network_interface.main.private_ip_address
  }
}

resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.wikijs.id
}

resource "azurerm_linux_virtual_machine" "main" {
  name                            = "${var.prefix}-vm"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_F2"
  admin_username                  = "it10"  
  network_interface_ids = [
    azurerm_network_interface.main.id,
    azurerm_network_interface.internal.id,
  ]
  admin_ssh_key {
    username   = "it10"  # Utilisateur pour SSH
    public_key = file("C:\\Users\\benoi\\.ssh\\id_rsa.pub") 
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version  

~~~


docker-compose.yml : exactement le même que dans le tp 1 je l'ai juste recup 