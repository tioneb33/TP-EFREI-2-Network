# Part I : Docker basics

**Partie introduction**, avec install de Docker et quelques `docker run`.

## Sommaire

- [Part I : Docker basics](#part-i--docker-basics)
  - [Sommaire](#sommaire)
  - [1. Install](#1-install)
  - [2. Vérifier l'install](#2-vérifier-linstall)
  - [3. Lancement de conteneurs](#3-lancement-de-conteneurs)

## 1. Install

🌞 **Installer Docker votre machine Azure**

~~~
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

~~~

test : 

~~~
azureuser@toto:~$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
e6590344b1a5: Pull complete
Digest: sha256:7e1a4e2d11e2ac7a8c3f768d4166c2defeb09d2a750b010412b6ea13de1efb19
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

~~~

Démarrer le service Docker avec systemctl : 

~~~
sudo systemctl start docker
~~~

Ajouter ton utilisateur au groupe docker:

~~~
sudo usermod -aG docker $(whoami)
~~~

puis reboot

## 2. Vérifier l'install



## 3. Lancement de conteneurs


🌞 **Utiliser la commande `docker run`**
commande: 

~~~
docker run -d -p 9999:80 --name mon-nginx nginx
~~~

result:
~~~

azureuser@toto:~$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
61efbc9b8924   nginx     "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   0.0.0.0:9999->80/tcp, [::]:9999->80/tcp   mon-nginx
azureuser@toto:~$
~~~

🌞 **Rendre le service dispo sur internet**

~~~
azureuser@toto:~$ curl http://172.17.0.1:9999
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
azureuser@toto:~$
~~~

🌞 **Custom un peu le lancement du conteneur**

Fichier de configuration NGINX (nginx.conf) :

~~~

server {
  listen 7777;  
  root /var/www/tp_docker; 
~~~

~~~
azureuser@toto:~/nginx-custom/html$ curl localhost:7777
<h1>Bienvenue sur mon serveur </h1>
~~~

🌞 **Call me**

OK

# Part II : Images

~~~
azureuser@toto:~/dockerfile1$ docker build -t mon-image-apache .
docker rm tada
docker run -d --name tada -p 8080:80 mon-image-apache
[+] Building 2.3s (10/10) FINISHED                                                                                                           docker:default
 => [internal] load build definition from Dockerfile                                                                                                   0.0s
 => => transferring dockerfile: 307B                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/debian:latest                                                                                       0.0s
 => [internal] load .dockerignore                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                        0.0s
 => [1/5] FROM docker.io/library/debian:latest                                                                                                         0.0s
 => [internal] load build context                                                                                                                      0.0s
 => => transferring context: 398B                                                                                                                      0.0s
 => CACHED [2/5] RUN apt-get update && apt-get install -y apache2 && rm -rf /var/lib/apt/lists/*                                                       0.0s
 => [3/5] COPY apache2.conf /etc/apache2/apache2.conf                                                                                                  0.1s
 => [4/5] COPY index.html /var/www/html/index.html                                                                                                     0.1s
 => [5/5] RUN mkdir -p /var/log/apache2                                                                                                                0.3s
 => exporting to image                                                                                                                                 1.5s
 => => exporting layers                                                                                                                                1.4s
 => => writing image sha256:9013e26540399dea503a3171748954a0b10a7d6a20bee7981e71e9710d035a9d                                                           0.0s
 => => naming to docker.io/library/mon-image-apache                                                                                                    0.0s
tada
2e08a9abb997178718732899dee387bc1dab87bf4427d10ffb2509804bd579c6
azureuser@toto:~/dockerfile1$ docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS         PORTS                                     NAMES
2e08a9abb997   mon-image-apache   "apachectl -D FOREGR…"   9 seconds ago   Up 8 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   tada
azureuser@toto:~/dockerfile1$ cat Dockerfile
FROM debian:latest

RUN apt-get update && apt-get install -y apache2 

COPY apache2.conf /etc/apache2/apache2.conf

COPY index.html /var/www/html/index.html

RUN mkdir -p /var/log/apache2

EXPOSE 80

CMD ["apachectl", "-D", "FOREGROUND"]
azureuser@toto:~/dockerfile1$
~~~

# Part III : `docker-compose`

## 2. WikiJS

utilisation de cette page pour Wiki.js : 

- https://docs.requarks.io/install/docker

Et les commandes : 

~~~
docker compose up -d

azureuser@toto:~/wikijs$ docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                                                   NAMES
9498544ff600   requarks/wiki:2   "docker-entrypoint.s…"   14 seconds ago   Up 12 seconds   3443/tcp, 0.0.0.0:8080->3000/tcp, [::]:8080->3000/tcp   wikijs-wikijs-1
3dc2ecfead2a   postgres:13       "docker-entrypoint.s…"   14 seconds ago   Up 13 seconds   5432/tcp                                                wikijs-db-1
azureuser@toto:~/wikijs$ cat docker-compose.yaml
version: "3.8"

services:
  db:
    image: postgres:13
    restart: always
    environment:
      POSTGRES_DB: wiki
      POSTGRES_USER: wikijs
      POSTGRES_PASSWORD: wikijs-pass
    volumes:
      - db_data:/var/lib/postgresql/data

  wikijs:
    image: requarks/wiki:2
    restart: always
    ports:
      - "8080:3000"
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijs-pass
      DB_NAME: wiki
    depends_on:
      - db

volumes:
  db_data:
azureuser@toto:~/wikijs$
~~~

## 3. Make your own meow

les cat de fichiers
~~~
azureuser@toto:~/el-app$ cat docker-compose.yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "8888:8888"
    depends_on:
      - db
  db:
    image: redis:latest
    ports:
      - "6379:6379"


azureuser@toto:~/el-app$ cat Dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

COPY . /app/

EXPOSE 8888

CMD ["python", "app.py"]

~~~

Suite a la création des 5 fichier dont 3 pas de moi on passe a l'execution : 

~~~
azureuser@toto:~/el-app$ docker-compose up --build
Creating network "el-app_default" with the default driver
Building web
[+] Building 2.0s (10/10) FINISHED                                                                                                           docker:default
 => [internal] load build definition from Dockerfile                                                                                                   0.1s
 => => transferring dockerfile: 206B                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/python:3.9-slim                                                                                     0.6s
 => [internal] load .dockerignore                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                        0.0s
 => [1/5] FROM docker.io/library/python:3.9-slim@sha256:d1fd807555208707ec95b284afd10048d0737e84b5f2d6fdcbed2922b9284b56                               0.0s
 => [internal] load build context                                                                                                                      0.0s
 => => transferring context: 374B                                                                                                                      0.0s
 => CACHED [2/5] WORKDIR /app                                                                                                                          0.0s
 => CACHED [3/5] COPY requirements.txt /app/                                                                                                           0.0s
 => CACHED [4/5] RUN pip install --no-cache-dir -r requirements.txt                                                                                    0.0s
 => [5/5] COPY . /app/                                                                                                                                 0.1s
 => exporting to image                                                                                                                                 1.0s
 => => exporting layers                                                                                                                                0.9s
 => => writing image sha256:4fcad906ec6f4e68323fa258bb4fa8e1d64c5409115f1eafadb5e509be035894                                                           0.0s
 => => naming to docker.io/library/el-app_web                                                                                                          0.0s
Creating el-app_db_1 ... done
Creating el-app_web_1 ... done
Attaching to el-app_db_1, el-app_web_1
db_1   | 1:C 14 Mar 2025 16:05:44.428 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
db_1   | 1:C 14 Mar 2025 16:05:44.429 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
db_1   | 1:C 14 Mar 2025 16:05:44.430 * Redis version=7.4.2, bits=64, commit=00000000, modified=0, pid=1, just started
db_1   | 1:C 14 Mar 2025 16:05:44.430 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
db_1   | 1:M 14 Mar 2025 16:05:44.431 * monotonic clock: POSIX clock_gettime
db_1   | 1:M 14 Mar 2025 16:05:44.434 * Running mode=standalone, port=6379.
db_1   | 1:M 14 Mar 2025 16:05:44.434 * Server initialized
db_1   | 1:M 14 Mar 2025 16:05:44.436 * Ready to accept connections tcp
web_1  |  * Serving Flask app 'app'
web_1  |  * Debug mode: off
web_1  | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
web_1  |  * Running on all addresses (0.0.0.0)
web_1  |  * Running on http://127.0.0.1:8888
web_1  |  * Running on http://172.19.0.3:8888
web_1  | Press CTRL+C to quit
web_1  | 90.47.3.222 - - [14/Mar/2025 16:06:01] "GET / HTTP/1.1" 200 -
~~~

et Boom la page web : 


Add key
Key: 
 Value: 
 
Check key
Key: 
 
Host : 2977c2a9c825

~~~

azureuser@toto:~/el-app$ docker run -it --rm --privileged --pid=host ubuntu bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
5a7813e071bf: Pull complete
Digest: sha256:72297848456d5d37d1262630108ab308d3e9ec7ed1c3286a32fe09856619a782
Status: Downloaded newer image for ubuntu:latest
root@39d624567660:/# cat /etc/shadow
root:*:20115:0:99999:7:::
daemon:*:20115:0:99999:7:::
bin:*:20115:0:99999:7:::
sys:*:20115:0:99999:7:::
sync:*:20115:0:99999:7:::
etc ...... 
~~~
## 1. Le groupe docker


🌞 **Prouvez que vous pouvez devenir `root`**

Commande trouvé sur internet pour le coup :
~~~

azureuser@toto:~/el-app$ docker run -it --rm --privileged --pid=host ubuntu bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
5a7813e071bf: Pull complete
Digest: sha256:72297848456d5d37d1262630108ab308d3e9ec7ed1c3286a32fe09856619a782
Status: Downloaded newer image for ubuntu:latest
root@39d624567660:/# cat /etc/shadow
root:*:20115:0:99999:7:::
daemon:*:20115:0:99999:7:::
bin:*:20115:0:99999:7:::
sys:*:20115:0:99999:7:::
sync:*:20115:0:99999:7:::
etc ...... 
~~~

## 2. Scan de vuln

Alors la commande qui va suivre est une adaptation qui cherche que les vulnérabilté et 
le -q sup tout le coté inutile car sinon a chaque fois la recherche était avorté avec un KILLED : 

Celle de WikiJS que vous avez build:
~~~
azureuser@toto:~/wikijs$ trivy image -q --scanners vuln --slow requarks/wiki:2

requarks/wiki:2 (alpine 3.21.2)

Total: 27 (UNKNOWN: 4, LOW: 2, MEDIUM: 19, HIGH: 2, CRITICAL: 0)

il y a le detail apres ...
~~~

celle de sa base de données, encore une fois très longue 

~~~
azureuser@toto:~/wikijs$ trivy image -q --scanners vuln postgres:13

postgres:13 (debian 12.9)

Total: 152 (UNKNOWN: 2, LOW: 107, MEDIUM: 33, HIGH: 9, CRITICAL: 1)

il y a le detail apres ...
~~~

l'image de Apache que vous avez build

~~~
azureuser@toto:~/dockerfile1$ trivy image -q --scanners vuln apache_image_name

apache_image_name (debian 12.9)

Total: 177 (UNKNOWN: 0, LOW: 128, MEDIUM: 36, HIGH: 12, CRITICAL: 1)
~~~

et enfin l'image de NGINX officielle utilisée dans la première partie: 

~~~
azureuser@toto:~/dockerfile1$ trivy image -q --scanners vuln nginx:latest

nginx:latest (debian 12.9)

Total: 157 (UNKNOWN: 2, LOW: 99, MEDIUM: 43, HIGH: 11, CRITICAL: 2)
~~~

- En résumé WikiJS a 27 vulnérabilités, principalement de sévérité moyenne (19) et basse (2), ce qui est relativement léger, mais quelques vulnérabilités moyennes restent préoccupantes.
- La base de données a 152 vulnérabilités, dont 1 critique et plusieurs hautes (9), ce qui la rend assez risquée.
- Apache et NGINX sont dans la même gamme avec respectivement 177 et 157 vulnérabilités, mais Apache semble légèrement plus vulnérable avec plus de vulnérabilités basses (128 contre 99 pour NGINX) et une critique aussi.
- En gros, la base de données a la vulnérabilité la plus dangereuse.