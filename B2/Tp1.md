# TP1 : Premiers pas Docker

## I. Init

### 3. sudo c pa bo

##### 🌞 Ajouter votre utilisateur au groupe docker

```
[hicham@localhost ~]$ sudo groupadd docker
groupadd: group 'docker' already exists
[hicham@localhost ~]$ sudo usermod -aG docker hicham
[hicham@localhost ~]$ exit
logout
Connection to 192.168.56.102 closed.
PS C:\Users\belma> ssh hicham@192.168.56.102
hicham@192.168.56.102's password:
Last login: Thu Dec 21 15:20:08 2023 from 192.168.56.1
[hicham@localhost ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[hicham@localhost ~]$
```

### 4. Un premier conteneur en vif

##### 🌞 Lancer un conteneur NGINX

je l'avais deja fait
```
[hicham@localhost ~]$ dk run -d -p 9999:80 nginx
90bad38f172f2aa8a2637f83a3e028768ec30693318d5ce766bca270d1f68056
docker: Error response from daemon: driver failed programming external connectivity on endpoint compassionate_blackburn (e37e58f2e252788a6a21b23a3497447ab2311ca7af83accace485cb858ba844a): Bind for 0.0.0.0:9999 failed: port is already allocated.
```


##### 🌞 Visitons

vérifier que le conteneur est actif avec une commande qui liste les conteneurs en cours de fonctionnement
```
[hicham@localhost ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                   NAMES
8b71a59b1bca   nginx     "/docker-entrypoint.…"   6 minutes ago   Up 5 minutes   0.0.0.0:9999->80/tcp, :::9999->80/tcp   keen_mendeleev
```
afficher les logs du conteneur
```
[hicham@localhost ~]$ docker logs 8b71a59b1bca
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/12/21 15:12:41 [notice] 1#1: using the "epoll" event method
2023/12/21 15:12:41 [notice] 1#1: nginx/1.25.3
2023/12/21 15:12:41 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2023/12/21 15:12:41 [notice] 1#1: OS: Linux 5.14.0-284.11.1.el9_2.x86_64
2023/12/21 15:12:41 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1073741816:1073741816
2023/12/21 15:12:41 [notice] 1#1: start worker processes
2023/12/21 15:12:41 [notice] 1#1: start worker process 30
2023/12/21 15:12:41 [notice] 1#1: start worker process 31
```
afficher toutes les informations relatives au conteneur avec une commande docker inspect
```
[hicham@localhost ~]$ docker inspect 8b71a59b1bca
[
    {
        "Id": "8b71a59b1bcac58bb958dd91288a149d9e1e48d44964da982167a9a9187a40df",
        "Created": "2023-12-21T15:12:37.306807638Z",
        "Path": "/docker-entrypoint.sh",
        "Args": [
            "nginx",
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 4386,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2023-12-21T15:12:40.798502408Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        
        
        ...
        ...
        ...


                    "Aliases": null,
                    "NetworkID": "c5140a982e498ccb2e60466b6461b43cd11740e25c72c77e64991ee6cbc60e41",
                    "EndpointID": "e27d6e3388eea1e53cb3c155be913b68120c1c9aaf5a401337d02991ae127efb",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

afficher le port en écoute sur la VM avec un sudo ss -lnpt

```
[hicham@localhost ~]$ sudo ss -lnpt
[sudo] password for hicham:
State    Recv-Q   Send-Q       Local Address:Port       Peer Address:Port   Process
LISTEN   0        4096               0.0.0.0:9999            0.0.0.0:*       users:(("docker-proxy",pid=4342,fd=4))
LISTEN   0        128                0.0.0.0:22              0.0.0.0:*       users:(("sshd",pid=705,fd=3))
LISTEN   0        4096                  [::]:9999               [::]:*       users:(("docker-proxy",pid=4348,fd=4))
LISTEN   0        128                   [::]:22                 [::]:*       users:(("sshd",pid=705,fd=4))
```

ouvrir le port 9999/tcp (vu dans le ss au dessus normalement) dans le firewall de la VM

```
[hicham@localhost ~]$ sudo firewall-cmd --zone=public --add-port=9999/tcp --permanent
success
```

depuis le navigateur de votre PC, visiter le site web sur http://IP_VM:9999
![Alt text](<Screenshot 2023-12-22 144322.png>)





##### 🌞 On va ajouter un site Web au conteneur NGINX

```
[hicham@localhost ~]$ mkdir nginx
[hicham@localhost ~]$ ls
nginx
[hicham@localhost ~]$ cd nginx
[hicham@localhost nginx]$ ls
[hicham@localhost nginx]$ mk index.html
-bash: mk: command not found
[hicham@localhost nginx]$ nano index.thml
[hicham@localhost nginx]$ ls
index.thml
[hicham@localhost nginx]$ mv index.html index.thml
mv: cannot stat 'index.html': No such file or directory
[hicham@localhost nginx]$ mv index.thml index.html
[hicham@localhost nginx]$ ls
index.html
[hicham@localhost nginx]$ nano site_nul.conf
[hicham@localhost nginx]$ ls
index.html  site_nul.conf
```

##### 🌞 Visitons

###### vérifier que le conteneur est actif
```
[hicham@localhost ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                               NAMES
f09fc6de342c   nginx     "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   80/tcp, 0.0.0.0:9999->8080/tcp, :::9999->8080/tcp   amazing_galileo
[hicham@localhost ~]$
```
###### visiter le site web depuis votre PC
![Alt text](<Screenshot 2023-12-22 160320.png>)

### 5. Un deuxième conteneur en vif

##### 🌞 Lance un conteneur Python, avec un shell

```
[hicham@localhost ~]$ dk run -it python bash
Unable to find image 'python:latest' locally
latest: Pulling from library/python
bc0734b949dc: Pull complete
b5de22c0f5cd: Pull complete
917ee5330e73: Pull complete
b43bd898d5fb: Pull complete
7fad4bffde24: Pull complete
d685eb68699f: Pull complete
107007f161d0: Pull complete
02b85463d724: Pull complete
Digest: sha256:3733015cdd1bd7d9a0b9fe21a925b608de82131aa4f3d397e465a1fcb545d36f
Status: Downloaded newer image for python:latest
root@ccfc6bcb74d6:/#
```

##### 🌞 Installe des libs Python

```
root@ccfc6bcb74d6:/# pip install aiohttp aioconsole
Collecting aiohttp
  Obtaining dependency information for aiohttp from https://files.pythonhosted.org/packages/75/5f/90a2869ad3d1fb9bd984bfc1b02d8b19e381467b238bd3668a09faa69df5/aiohttp-3.9.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata
  Downloading aiohttp-3.9.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (7.4 kB)
Collecting aioconsole



  ...


[notice] A new release of pip is available: 23.2.1 -> 23.3.2
[notice] To update, run: pip install --upgrade pip
root@ccfc6bcb74d6:/# pytho
bash: pytho: command not found
root@ccfc6bcb74d6:/# python
Python 3.12.1 (main, Dec 19 2023, 20:14:15) [GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import aiohttp
>>> ls
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'ls' is not defined
>>> exit
Use exit() or Ctrl-D (i.e. EOF) to exit
>>> exit()
```



## II. Images




### 1. Images publiques
#### 🌞 Récupérez des images

##### avec la commande docker pull



```
[hicham@localhost ~]$ docker pull python:3.11
docker pull mysql:5.7
docker pull wordpress:latest
docker pull linuxserver/wikijs:latest
3.11: Pulling from library/python
bc0734b949dc: Already exists
b5de22c0f5cd: Already exists
917ee5330e73: Already exists 

...

20b23561c7ea: Pull complete
Digest: sha256:131d247ab257cc3de56232b75917d6f4e24e07c461c9481b0e7072ae8eba3071
Status: Downloaded newer image for linuxserver/wikijs:latest
docker.io/linuxserver/wikijs:latest
[hicham@localhost ~]$
```


##### listez les images que vous avez sur la machine avec une commande docker

```
[hicham@localhost ~]$ docker images
REPOSITORY           TAG       IMAGE ID       CREATED       SIZE
linuxserver/wikijs   latest    869729f6d3c5   6 days ago    441MB
mysql                5.7       5107333e08a8   8 days ago    501MB
python               latest    fc7a60e86bae   13 days ago   1.02GB
wordpress            latest    fd2f5a0c6fba   2 weeks ago   739MB
python               3.11      22140cbb3b0c   2 weeks ago   1.01GB
nginx                latest    d453dd892d93   8 weeks ago   187MB
[hicham@localhost ~]$
```

#### 🌞 Lancez un conteneur à partir de l'image Python

```
[hicham@localhost ~]$ docker run -it python:3.11 bash
root@e0c590dedcdd:/# version
bash: version: command not found
root@e0c590dedcdd:/# python--vers
bash: python--vers: command not found
root@e0c590dedcdd:/# python --vers
unknown option --vers
usage: python [option] ... [-c cmd | -m mod | file | -] [arg] ...
Try `python -h' for more information.
root@e0c590dedcdd:/# python --version
Python 3.11.7
root@e0c590dedcdd:/# exit
```


### 2. Construire une image



#### 🌞 Ecrire un Dockerfile pour une image qui héberge une application Python

```
[hicham@localhost python-app-build]$ ls
app.py  Dockerfile
[hicham@localhost python-app-build]$ pwd
/home/hicham/python-app-build
```


```
  GNU nano 5.6.1                    Dockerfile                              FROM debian:latest

RUN apt update || true

RUN apt install -y python3

RUN apt-get install -y python3-emoji

COPY app.py /app.py

ENTRYPOINT ["python3", "/app.py"]


```


#### 🌞 Build l'image

```
[hicham@localhost python-app-build]$ docker build . -t python-app-build:version_de_ouf
[+] Building 11.7s (10/10) FINISHED                          docker:default
 => [internal] load .dockerignore                                      0.0s
 => => transferring context: 2B                                        0.0s
 => [internal] load build definition from Dockerfile                   0.1s
 => => transferring dockerfile: 263B                                   0.0s
 => [internal] load metadata for docker.io/library/debian:latest       0.8s
 => [1/5] FROM docker.io/library/debian:latest@sha256:bac353db4cc04bc  0.0s
 => [internal] load build context                                      0.0s
 => => transferring context: 86B                                       0.0s
 => CACHED [2/5] RUN apt update || true                                0.0s
 => CACHED [3/5] RUN apt install -y python3                            0.0s
 => [4/5] RUN apt-get install -y python3-emoji                         4.8s
 => [5/5] COPY app.py /app.py                                          0.4s
 => exporting to image                                                 5.3s
 => => exporting layers                                                5.2s
 => => writing image sha256:f3d6bfb30d3922d37a7282f53bdc4833665598bd9  0.0s
 => => naming to docker.io/library/python-app-build:version_de_ouf     0.0s
```


#### 🌞 Lancer l'image

```
[hicham@localhost python-app-build]$ docker run python-app-build:version_de_
ouf
Cet exemple d'application est vraiment naze 👎
[hicham@localhost python-app-build]$
```

## III. Docker compose


#### 🌞 Créez un fichier docker-compose.yml


```
[hicham@localhost ~]$ mkdir compose-test
[hicham@localhost ~]$ ls
compose-test  nginx  python-app-build
[hicham@localhost ~]$ cd compose-test/
[hicham@localhost compose-test]$ nano docker-compose.yml
```


#### 🌞 Lancez les deux conteneurs avec docker compose

```
[hicham@localhost compose-test]$ docker compose up -d
[+] Running 3/3
 ✔ conteneur_nul Pulled                                                5.8s
 ✔ conteneur_flopesque 1 layers [⣿]      0B/0B      Pulled             5.6s
   ✔ bc0734b949dc Already exists                                       0.0s
[+] Running 3/3
 ✔ Network compose-test_default                  Created               1.6s
 ✔ Container compose-test-conteneur_flopesque-1  Started               0.2s
 ✔ Container compose-test-conteneur_nul-1        Started               0.2s
[hicham@localhost compose-test]$
```

#### 🌞 Vérifier que les deux conteneurs tournent

```
[hicham@localhost compose-test]$ dk compose ls
NAME                STATUS              CONFIG FILES
compose-test        running(2)          /home/hicham/compose-test/docker-compose.yml
[hicham@localhost compose-test]$ dk compose top
compose-test-conteneur_flopesque-1
UID    PID     PPID    C    STIME   TTY   TIME       CMD
root   13459   13414   0    23:01   ?     00:00:00   sleep 9999

compose-test-conteneur_nul-1
UID    PID     PPID    C    STIME   TTY   TIME       CMD
root   13470   13431   0    23:01   ?     00:00:00   sleep 9999

[hicham@localhost compose-test]$ docker compose ps
NAME                                 IMAGE     COMMAND        SERVICE               CREATED         STATUS         PORTS
compose-test-conteneur_flopesque-1   debian    "sleep 9999"   conteneur_flopesque   9 minutes ago   Up 9 minutes
compose-test-conteneur_nul-1         debian    "sleep 9999"   conteneur_nul         9 minutes ago   Up 9 minutes
[hicham@localhost compose-test]$
```


#### 🌞 Pop un shell dans le conteneur conteneur_nul

```
[hicham@localhost compose-test]$ docker exec -it  compose-test-conteneur_flo
pesque-1 bash
root@44232bc335f6:/# apt get update
E: Invalid operation get
root@44232bc335f6:/# apt-get update
Hit:1 http://deb.debian.org/debian bookworm InRelease
Get:2 http://deb.debian.org/debian bookworm-updates InRelease [52.1 kB]
Get:3 http://deb.debian.org/debian-security bookworm-security InRelease [48.0 kB]
Reading package lists... Done
E: Release file for http://deb.debian.org/debian/dists/bookworm-updates/InRelease is not valid yet (invalid for another 3d 15h 31min 56s). Updates for this repository will not be applied.
E: Release file for http://deb.debian.org/debian-security/dists/bookworm-security/InRelease is not valid yet (invalid for another 2d 13h 29min 55s). Updates for this repository will not be applied.
root@44232bc335f6:/# apt-get install iputils-ping
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libcap2-bin libpam-cap
The following NEW packages will be installed:
  iputils-ping libcap2-bin libpam-cap
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 96.2 kB of archives.
After this operation, 311 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://deb.debian.org/debian bookworm/main amd64 libcap2-bin amd64 1:2.66-4 [34.7 kB]
Get:2 http://deb.debian.org/debian bookworm/main amd64 iputils-ping amd64 3:20221126-1 [47.1 kB]
Get:3 http://deb.debian.org/debian bookworm/main amd64 libpam-cap amd64 1:2.66-4 [14.5 kB]
Fetched 96.2 kB in 7s (13.8 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libcap2-bin.
(Reading database ... 6098 files and directories currently installed.)
Preparing to unpack .../libcap2-bin_1%3a2.66-4_amd64.deb ...
Unpacking libcap2-bin (1:2.66-4) ...
Selecting previously unselected package iputils-ping.
Preparing to unpack .../iputils-ping_3%3a20221126-1_amd64.deb ...
Unpacking iputils-ping (3:20221126-1) ...
Selecting previously unselected package libpam-cap:amd64.
Preparing to unpack .../libpam-cap_1%3a2.66-4_amd64.deb ...
Unpacking libpam-cap:amd64 (1:2.66-4) ...
Setting up libcap2-bin (1:2.66-4) ...
Setting up libpam-cap:amd64 (1:2.66-4) ...
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 78.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.36.0 /usr/local/share/perl/5.36.0 /usr/lib/x86_64-linux-gnu/perl5/5.36 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl-base /usr/lib/x86_64-linux-gnu/perl/5.36 /usr/share/perl/5.36 /usr/local/lib/site_perl) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Setting up iputils-ping (3:20221126-1) ...
root@44232bc335f6:/# ping conteneur_flocompose-test-conteneur_nul-1
ping: conteneur_flocompose-test-conteneur_nul-1: Name or service not known
root@44232bc335f6:/# ping compose-test-conteneur_nul-1
PING compose-test-conteneur_nul-1 (172.18.0.3) 56(84) bytes of data.
64 bytes from compose-test-conteneur_nul-1.compose-test_default (172.18.0.3): icmp_seq=1 ttl=64 time=13.8 ms
64 bytes from compose-test-conteneur_nul-1.compose-test_default (172.18.0.3): icmp_seq=2 ttl=64 time=0.149 ms

... 

64 bytes from compose-test-conteneur_nul-1.compose-test_default (172.18.0.3): icmp_seq=53 ttl=64 time=0.200 ms
64 bytes from compose-test-conteneur_nul-1.compose-test_default (172.18.0.3): icmp_seq=54 ttl=64 time=0.206 ms
^C
--- compose-test-conteneur_nul-1 ping statistics ---
54 packets transmitted, 54 received, 0% packet loss, time 52978ms
rtt min/avg/max/mdev = 0.108/0.446/13.757/1.831 ms
root@44232bc335f6:/#
```