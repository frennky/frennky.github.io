---
title:  "CDN - Content Delivery Network"
date:   2011-06-23 23:19:30 +0100
---

CDN predstavlja mrežu servera koji imaju isti sadržaj ali su geografski razmešteni na više lokacija. Kada krajnji korisnik pošalje zahtev za nekom stranicom, CDN dinamički izračunava koji CDN server mu je geografski najbliži i šelje odgovor sa tog servera. Sofisticiraniji CDN sistemi mogu da vrše i balansiranje opterećenja ukoliko u datom trenutku dođe do velikog saobraćaja na nekoj geografskoj lokaciji.

Bez CDN-a web sajt se servira sa jedne geografske lokacije na kojoj se nalazi jedan server. Veliki saobraćaj ili velika distanca između servera i krajnjeg korisnika može negativno da utiče na brzinu učitavanja stranica web sajta.

Posetioci sajta najčešće nisu voljni da čekaju više od par sekundi za učitavanje stranice. Zbog toga je jako važno smanjiti vreme odziva servera na klijentski zahtev.

Na neki način, CDN igra sličnu ulogu za servere kao proxy za klijente. Proxy server je često fizički bliže krajnjem korisniku ili, što je još bitnije, ima bržu konekciju sa krajnjim korisnikom, tako da u slučaju da ima u svom kešu traženu stranicu (ili bilo koji drugi sadržaj) može brže da je isporuči krajnjem korisniku nego izvorni server. CDN povećava ukupni protok zbog toga što se sastoji od većeg broja servera sa istim sadržajem, a s druge strane, smanjuje vreme odgovora usled manje geografske distance između servera i krajnjeg korisnika.

## Prednosti upotrebe CDN

- Povećava propusnu moć
- Povećava redudantnost
- Smanjuje latenciju

Kada se serviraju fajlovi malih veličina, CDN doprinosi bržem učitavanju stranica usled manje latencije, a u slučaju velikih fajlova CDN obezbeđuje veće brzine download-a usled povećane propusne moći.

Redudantnost podataka obezbeđuje veću dostupnost, odnosno postepenu degradaciju u slučaju otkaza servera ili zagušenja saobraćaja na internetu.

## Upotreba CDN-a

Zbog svoje velike popularnosti neke JavaScript biblioteke kao što su jQuery i jQuery UI su dostupni preko Google ili Microsoft CDN-a.

S obzirom da browser-i keširaju sav statički sadržaj kao što su CSS i JavaScript fajlovi, ukoliko se korite neki od ovih CDN-a, vrlo je verovatno da će se neke od popularnih JavaScript biblioteka već naći u kešu krajnjeg korisnika.

Da bi koristili Google CDN-u za jQuery bibilioteku potrebno je samo referencirati skriptu sa odgovarajuće adrese:

```html
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.6.2/jquery.js"></script>
```

ili za minimizaranu verziju:

```html
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.6.2/jquery.min.js"></script>
```

Takođe, možemo i upotrebiti Google libraries API i pozvati load funkciju za učitavanje željene biblioteke:

```javascript
google.load("jquery", "1.6.2");
```

Za više informacija o Google CDN-u i ostalim JavaScript bibliotekama možete pogledati zvaničnu [dokumentaciju](http://code.google.com/apis/libraries/devguide.html).

Pored Google CDN-a i Microsoft ima besplatan CDN za popularne JavaScript biblioteke:

```html
<script src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-1.6.2.js"></script>
<script src="http://ajax.aspnetcdn.com/ajax/jquery.ui/1.8.14/jquery-ui.js"></script>
```

ili za minimizaranu verziju:

```html
<script src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-1.6.2.min.js"></script>
<script src="http://ajax.aspnetcdn.com/ajax/jquery.ui/1.8.14/jquery-ui.min.js"></script>
```

Za više informacija posetite ASP.NET CDN [dokumentaciju](http://www.asp.net/ajaxlibrary/cdn.ashx).
