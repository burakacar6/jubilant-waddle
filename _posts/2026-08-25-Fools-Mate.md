---
title: Fools Mate Writeup
description: >-
  Can you bypass the engine?
author: ruh hastasi
date: 2026-08-25 00:56:00 +0800
categories: [CTF, Tryhackme, Easy]
tags: [CTF]
pin: false
---

## Karşılama
EndgameTrainer adında bir satranç sayfasıyla karşılaştım. Umarım satranç oynamam gerekmez, ilkokul turnuvasında sonuncu olmuştum.

![alt text](/assets/img/EndgameTrainer_index.png)

## Keşif aşaması
Satranç tahtasında taşları hareket ettirdiğimde aldığım Response dikkarimi çekti. `fen` adında bir parametre alıyordum. fen dışında ayrıca `move`, `status`, `turn` ve `botMove` dikkatimi çekti.

![alt text](/assets/img/fen_parametreleri.png)

Taşlarımı mat konumuna getirdiğimde ise bana bot engel oluyordu. Bilgisayarımı kapatacağını söyleyen bir uyarı.

![alt text](/assets/img/tehdit_mesaji_satranc.png)

Directory'lerde bir şey bulamadım bu yüzden bulduğumuz parametreleri değiştirmeye çalışacağım.

![alt text](/assets/img/Endgame_trainer_directory.png)

## Exploit denemesi

Satranç tahtasında her taş hareket ettirdiğimizde response aldığımız gibi request'de yolluyoruz. 

Taş, nerden (`from`) nereye (`to`) olarak bir istek yolluyoruz.

![alt text](/assets/img/Endgame_Trainer_request1.png)

Hesaplarıma göre şuan taşı **a1**'dan **a8**'e yollarsam belki oyunu kazanabilirim.

![alt text](/assets/img/endgametrainer_flag.png)
