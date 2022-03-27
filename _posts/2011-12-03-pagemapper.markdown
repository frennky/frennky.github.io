---
layout: post
title:  "PageMapper"
date:   2011-12-03 19:53:30 +0100
---

U prethodnom postu sam govorio o [Page Object paternu](http://ivanfranjic.net/2011/11/page-object-pattern) kao načinu pisanja UI testova i dao primer koji se oslanja na Selenium WebDriver. Iako ovaj pristup smanjuje dupliranje koda i olakšava održavanje, implementacija Page Object klasa može da bude prilično zamorna.

Da bi koristli Paga Object kalse, potrebno je da izvršimo mapiranje korisničkog interfejsa aplikacije koja se testira. Ako uzmemo u obzir da jedna netrivijalna aplikacija može da ima nekoliko desetina, pa i par stotina, stranica a da pri tom svaka stranica može da ima veći broj UI kontrola koje podržavaju neki vid interakcija, jasno je da UI mapiranje može da preraste u veliki posao.

Kako bih donekle ublažio ovaj bolni proces, napravio sam jednostavan alat za mapiranje web stranica. PageMapper je inicijalno planiran kao alat koji treba da mapira web stranice za Selenium WebDriver C# biblioteku. Međutim, kako alat koristi [Razor](http://razorengine.codeplex.com/) za generisanje Page Object klase, dodavanjem novih templejta mogu se generisati i klase za druge programske jezike, pa čak i za druge framework-e za UI testiranje.

Podrazumevani templejt generiše stranicu koja se oslanja na [PageFactory](http://code.google.com/p/selenium/wiki/PageFactory) klasu, iz WebDriver Support biblioteke, za kreiranje Page Object-a. Naravno, plan je da se dodaju i templejti za ostale jezike, pa ako želite da doprinesete slobodno mi se javite.

Pre nego što skinete ovaj alat treba da znate da je još uvek u razvoju. Trenutna verzija je 0.9, i iako je funkcionalna, nedostaju neke stvari kao što su validacija i error logging. Međutim, uz pravilnu upotrebu alat neće praviti nikakve probleme :)

Nemojte se iznenaditi ako se otvore dve forme kada pokrenete alat. Nakon demo verzije, koja je imala jednu formu, shvatio sam da mi je jedna forma suviše mala. Tako je nova verzija alat osmišljena da se koristi sa dva monitora, gde bi na jednom monirotu bila browser forma a glavna forma na drugom.

Browser forma se koristi za navigaciju kroz web sajt čije stranice treba da se mapiraju, a da bi se izvršila selekcija nekog UI elemnta potrebno je držati CRTL i mišem preći preko željenog UI elemnta. Na glavnoj formi se prikazuju informacije o selektovanom elementu, koji nakon toga možemo dodati u listu. Svi web elementi koji se nalaze u listi biće generisani kako propertiji Page Object klase.

Za sva pitanja ili sugestije u vezi alata možete me slobodno kontaktirati.

BitBucket: [PageMapper](https://github.com/frennky/PageMapper)
