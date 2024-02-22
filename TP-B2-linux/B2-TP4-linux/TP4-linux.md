# TP4 : Vers une maîtrise des OS Linux

## 1. LVM dès l'installation
 ```[hp@localhost ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   40G  0 disk 
├─sda1        8:1    0   21G  0 part 
│ ├─rl-root 253:0    0   10G  0 lvm  /
│ ├─rl-swap 253:1    0    1G  0 lvm  [SWAP]
│ ├─rl-home 253:2    0   23G  0 lvm  /home
│ └─rl-var  253:3    0    5G  0 lvm  /var
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   18G  0 part 
  └─rl-home 253:2    0   23G  0 lvm  /home
sr0          11:0    1 1024M  0 rom  
sr1          11:1    1 1024M  0 rom  
```

## 2. Scénario remplissage de partition

### 🌞 Remplissez votre partition /home
### 🌞 Constater que la partition est pleine
### 🌞 Agrandir la partition
### 🌞 Remplissez votre partition /home
```[hp@localhost ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                882M     0  882M   0% /dev/shm
tmpfs                353M  5.0M  348M   2% /run
/dev/mapper/rl-root  9.8G  967M  8.3G  11% /
/dev/sda2            974M  182M  725M  21% /boot
/dev/mapper/rl-home   23G   22G     0 100% /home
/dev/mapper/rl-var   4.9G  104M  4.5G   3% /var
tmpfs                177M     0  177M   0% /run/user/1000
```
Leo on avait fait ca ensemble, mais j'ai oublié de copier les commandes avant d'eteindre la VM

### 🌞 Utiliser ce nouveau disque pour étendre la partition /home de 40G

```
hb313@chaussette:~$ ssh hp@10.3.1.2
hp@10.3.1.2's password: 
Last login: Thu Feb  1 15:25:21 2024
[hp@localhost ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   40G  0 disk 
├─sda1        8:1    0   21G  0 part 
│ ├─rl-root 253:0    0   10G  0 lvm  /
│ ├─rl-swap 253:1    0    1G  0 lvm  [SWAP]
│ ├─rl-home 253:2    0   23G  0 lvm  /home
│ └─rl-var  253:3    0    5G  0 lvm  /var
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   18G  0 part 
  └─rl-home 253:2    0   23G  0 lvm  /home
sdb           8:16   0   40G  0 disk 
sr0          11:0    1 1024M  0 rom  
sr1          11:1    1 1024M  0 rom  
[hp@localhost ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                882M     0  882M   0% /dev/shm
tmpfs                353M  5.0M  348M   2% /run
/dev/mapper/rl-root  9.8G  967M  8.3G  11% /
/dev/sda2            974M  182M  725M  21% /boot
/dev/mapper/rl-home   23G   22G     0 100% /home
/dev/mapper/rl-var   4.9G  104M  4.5G   3% /var
tmpfs                177M     0  177M   0% /run/user/1000


[hp@localhost ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.

[hp@localhost ~]$ sudo vgs
  VG #PV #LV #SN Attr   VSize  VFree
  rl   2   4   0 wz--n- 38.99g    0 
[hp@localhost ~]$ vgextend rl /dev/sdb
  WARNING: Running as a non-root user. Functionality may be unavailable.
  /run/lock/lvm/P_global:aux: open failed: Permission denied
[hp@localhost ~]$ sudo !!
sudo vgextend rl /dev/sdb
  Volume group "rl" successfully extended

[hp@localhost ~]$ sudo vgs
  VG #PV #LV #SN Attr   VSize   VFree  
  rl   3   4   0 wz--n- <78.99g <40.00g
[hp@localhost ~]$ sudo lvs
  LV   VG Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home rl -wi-ao---- 22.99g                                                    
  root rl -wi-ao---- 10.00g                                                    
  swap rl -wi-ao----  1.00g                                                    
  var  rl -wi-ao----  5.00g                                                    
[hp@localhost ~]$ lvextend -l +100%FREE /dev/mapper/rl-home
  WARNING: Running as a non-root user. Functionality may be unavailable.
  Failed to lock the devices file.
  Failed to set up devices.
[hp@localhost ~]$ sudo !!
sudo lvextend -l +100%FREE /dev/mapper/rl-home
  Size of logical volume rl/home changed from 22.99 GiB (5886 extents) to <62.99 GiB (16125 extents).
  Logical volume rl/home successfully resized.

[hp@localhost ~]$ sudo lvs
  LV   VG Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home rl -wi-ao---- <62.99g                                                    
  root rl -wi-ao----  10.00g                                                    
  swap rl -wi-ao----   1.00g                                                    
  var  rl -wi-ao----   5.00g                                                    
[hp@localhost ~]$ sudo resize2fs /dev/mapper/rl-home
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/mapper/rl-home is mounted on /home; on-line resizing required
old_desc_blocks = 3, new_desc_blocks = 8
The filesystem on /dev/mapper/rl-home is now 16512000 (4k) blocks long.

[hp@localhost ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   40G  0 disk 
├─sda1        8:1    0   21G  0 part 
│ ├─rl-root 253:0    0   10G  0 lvm  /
│ ├─rl-swap 253:1    0    1G  0 lvm  [SWAP]
│ ├─rl-home 253:2    0   63G  0 lvm  /home
│ └─rl-var  253:3    0    5G  0 lvm  /var
├─sda2        8:2    0    1G  0 part /boot
└─sda3        8:3    0   18G  0 part 
  └─rl-home 253:2    0   63G  0 lvm  /home
sdb           8:16   0   40G  0 disk 
└─rl-home   253:2    0   63G  0 lvm  /home
sr0          11:0    1 1024M  0 rom  
sr1          11:1    1 1024M  0 rom  
[hp@localhost ~]$ df -h
Filesystem           Size  Used Avail Use% Mounted on
devtmpfs             4.0M     0  4.0M   0% /dev
tmpfs                882M     0  882M   0% /dev/shm
tmpfs                353M  5.0M  348M   2% /run
/dev/mapper/rl-root  9.8G  967M  8.3G  11% /
/dev/sda2            974M  182M  725M  21% /boot
/dev/mapper/rl-home   62G   22G   38G  37% /home
/dev/mapper/rl-var   4.9G  104M  4.5G   3% /var
tmpfs                177M     0  177M   0% /run/user/1000
[hp@localhost ~]$ 


```

## II. Gestion de users

### 🌞 Gestion basique de users

```
[toto@localhost ~]$ sudo cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
tss:x:59:59:Account used for TPM access:/:/usr/sbin/nologin
sssd:x:998:995:User for sssd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/sbin/nologin
chrony:x:997:994:chrony system user:/var/lib/chrony:/sbin/nologin
systemd-oom:x:992:992:systemd Userspace OOM Killer:/:/usr/sbin/nologin
hp:x:1000:1000:hp:/home/hp:/bin/bash
alice:x:1001:1002::/home/alice:/bin/bash
bob:x:1002:1003::/home/bob:/bin/bash
charlie:x:1003:1004::/home/charlie:/bin/bash
eve:x:1004:1005::/home/eve:/bin/bash
backup:x:1005:1006::/var/backup:/usr/sbin/nologin
toto:x:1006:1007::/home/toto:/bin/bash
[toto@localhost ~]$ 
 
```

### 🌞 La conf sudo doit être la suivante


question Leo :pourquoi le user eve devrait pouvoir utiliser une commande en temps qu'utilisateur d'un autre groupe ??
```
[hp@localhost etc]$ sudo cat sudoers
## Sudoers allows particular users to run various commands as
## the root user, without needing the root password.
##
.
.
.
.
.
## Allows members of the users group to shutdown this system
# %users  localhost=/sbin/shutdown -h now

## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment)
#includedir /etc/sudoers.d
%admins ALL=(ALL) NOPASSWD: ALL
eve ALL=(backup) /bin/ls
[hp@localhost etc]$ 

```
### 🌞 Le dossier /var/backup

il a pas deja été créé quand on a créé le groupe et l'utilisateur backup ? On peut creer un groupe en lui specifiant un homedir qui n'existe pas ?

```
[hp@localhost home]$ sudo mkdir /var/backup
[hp@localhost home]$ sudo chmod 700 /var/backup
[hp@localhost home]$ sudo chown backup:backup /var/backup
[hp@localhost home]$ sudo touch /var/backup/precious_backup
[hp@localhost home]$ sudo chmod 740 /var/backup/precious_backup
[hp@localhost home]$ sudo chmod 640 /var/backup/precious_backup
[hp@localhost home]$ ls -l /var/backup/precious_backup
ls: cannot access '/var/backup/precious_backup': Permission denied
[hp@localhost home]$ sudo ls -l /var/backup/precious_backup
-rw-r-----. 1 root root 0 Feb  5 22:21 /var/backup/precious_backup
[hp@localhost home]$ sudo chown backup:backup /var/backup/precious_backup
[hp@localhost home]$ sudo ls -l /var/backup/precious_backup
-rw-r-----. 1 backup backup 0 Feb  5 22:21 /var/backup/precious_backup
[hp@localhost home]$ sudo ls -l /var/backup
total 0
-rw-r-----. 1 backup backup 0 Feb  5 22:21 precious_backup
[hp@localhost home]$ 


```

Question : si le fichier precious_backup est dans /var/backup qui lui meme appartient a backup; est il necessaire de changer son proprietaire ?(voir code au dessus; c'etait necessaire dans ce cas)


### 🌞 Mots de passe des users, prouvez que

Jcrois que j'ai oublié de definir un mot de passe pour les Users 😅, mais on avait creer un utilisateur "toto" ensemble  : on peut remarquer $6$ pour SHA512, et le salage vC/pT2WmKuJaC.CC

```
[hp@localhost home]$ sudo cat /etc/shadow
root:$6$F2zj58AiOTOugZxm$Lh4T/KbR5/aeD/cuz1.ZgAStm87cVea8CJmzhIA4C74uNINDgd/2e9xsMyyfux/McIp0UDRrwsasj5uLGr2zC/::0:99999:7:::
bin:*:19469:0:99999:7:::
daemon:*:19469:0:99999:7:::
adm:*:19469:0:99999:7:::
lp:*:19469:0:99999:7:::
sync:*:19469:0:99999:7:::
shutdown:*:19469:0:99999:7:::
halt:*:19469:0:99999:7:::
mail:*:19469:0:99999:7:::
operator:*:19469:0:99999:7:::
games:*:19469:0:99999:7:::
ftp:*:19469:0:99999:7:::
nobody:*:19469:0:99999:7:::
systemd-coredump:!!:19748::::::
dbus:!!:19748::::::
tss:!!:19748::::::
sssd:!!:19748::::::
sshd:!!:19748::::::
chrony:!!:19748::::::
systemd-oom:!*:19748::::::
hp:$6$DwmmquHcwy6/rTk1$ZADOqZxrn.rvw8zrkciQYR28FhVunsm7fHm9NksCq7TqRi9yEnZ7yRUD8e9DakJU5W9f9vlY.Qhx0utYhZe371::0:99999:7:::
alice:!!:19754:0:99999:7:::
bob:!!:19754:0:99999:7:::
charlie:!!:19754:0:99999:7:::
eve:!!:19754:0:99999:7:::
backup:!!:19754:0:99999:7:::
toto:$6$vC/pT2WmKuJaC.CC$cnvTEemQtFRDDEFVb/9Y8I4QT1bFsgdnh8ctUymrg8FNK8sA0F8YCZxkbbMQiuz9M.HB8iuhoaFXfIVpmZafU/:19754:0:99999:7:::

```
### 🌞 User eve

```
[hp@localhost home]$ sudo -l -User eve
sudo: unknown user: ser
[hp@localhost home]$ sudo -l -U eve
[sudo] password for hp: 
Matching Defaults entries for eve on localhost:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset,
    env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR
    USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT
    LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME
    LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User eve may run the following commands on localhost:
    (backup) /bin/ls

```

### 🌞 Je vous laisse gérer le bail vous-mêmes

Je suppose que c'st le service ntpd.service
```
[hp@localhost home]$ systemctl list-units -t service -a
  UNIT                                                                                      LOAD      >
  auditd.service                                                                            loaded    >
● autofs.service                                                                            not-found >
  chronyd.service                                                                           loaded    >
  crond.service                                                                             loaded    >
  dbus-broker.service                                                                       loaded    >
● display-manager.service                                                                   not-found >
 .
 .
 . 
 .
 .
 ● network.service                                                                           not-found >
  NetworkManager-wait-online.service                                                        loaded    >
  NetworkManager.service                                                                    loaded    >
  nftables.service                                                                          loaded    >
  nis-domainname.service                                                                    loaded    >
● ntpd.service                                                                              not-found >
● ntpdate.service                                                                           not-found >
● plymouth-quit-wait.service                                                                not-found >
● plymouth-start.service                                                                    not-found >
● polkit.service                                                                            not-found >
  rc-local.service                                                                          loaded    >
  rescue.service                                                                            loaded    >
  rsyslog.service                                                                           loaded    >
  selinux-autorelabel-mark.service                                                          loaded    >
lines 26-54


```


```
[hp@localhost home]$ sudo timedatectl set-ntp false
[sudo] password for hp: 
[hp@localhost home]$ sudo ntpdate 0.fr.pool.ntp.org
sudo: ntpdate: command not found
[hp@localhost home]$ sudo dnf install ntpdate
Last metadata expiration check: 0:50:47 ago on Mon 05 Feb 2024 10:04:12 PM CET.
No match for argument: ntpdate
Error: Unable to find a match: ntpdate
[hp@localhost home]$ sudo apt-get install ntpdate
sudo: apt-get: command not found
[hp@localhost home]$ sudo dnf install ntp
Last metadata expiration check: 0:51:33 ago on Mon 05 Feb 2024 10:04:12 PM CET.
No match for argument: ntp
Error: Unable to find a match: ntp
[hp@localhost home]$ sudo dnf install chrony
Last metadata expiration check: 0:51:41 ago on Mon 05 Feb 2024 10:04:12 PM CET.
Package chrony-4.3-1.el9.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
[hp@localhost home]$ sudo chronyd -q 'server 0.fr.pool.ntp.org iburst'
2024-02-05T21:56:13Z chronyd version 4.3 starting (+CMDMON +NTP +REFCLOCK +RTC +PRIVDROP +SCFILTER +SIGND +ASYNCDNS +NTS +SECHASH +IPV6 +DEBUG)
2024-02-05T21:56:13Z Initial frequency -6850.608 ppm
2024-02-05T21:56:19Z System clock wrong by 0.129204 seconds (step)
2024-02-05T21:56:19Z chronyd exiting
[hp@localhost home]$ date
Mon Feb  5 10:57:14 PM CET 2024
[hp@localhost home]$ sudo systemctl enable --now chronyd
Created symlink /etc/systemd/system/multi-user.target.wants/chronyd.service → /usr/lib/systemd/system/chronyd.service.
[hp@localhost home]$ chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* shogun.ruselabs.com           2   6     7     1  +5857us[  +94ms] +/-  231ms
^+ ntp01.ix1.inferno.net.uk      2   6     7     1  -5766us[-5766us] +/-  224ms
^? cheddar.halon.org.uk          2   6     5     3    -51ms[  +37ms] +/-  243ms
^- time.rdg.uk.as44574.net       3   6     7     2  +7699us[+7699us] +/-  246ms
[hp@localhost home]$ timedatectl
               Local time: Mon 2024-02-05 22:57:45 CET
           Universal time: Mon 2024-02-05 21:57:45 UTC
                 RTC time: Mon 2024-02-05 21:57:44
                Time zone: Europe/Paris (CET, +0100)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
[hp@localhost home]$ 

```