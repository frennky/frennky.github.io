---
layout: post
title:  "Migracija sa Selenium RC na Selenium WebDriver"
date:   2012-02-14 09:28:30 +0100
---

Kako je WebDriver naslednik Selenium RC-a, svakako da je bolja investicija pisati nove testove korišćenjem novog API-ja. Međutim, šta raditi sa testovima koji već postoje i koji su napisani za Selenium RC. Tvorci WebDriver-a su ovaj problem imali u vidu i zbog toga je moguće izvršiti migraciju na novi API na prilično bezbolan način.

Ukoliko je Selenium Server već startovan, potrebno je dodati u test još dve linije koda:

```csharp
var selenium = new DefaultSelenium("localhost", 4444, "*firefox", "http://www.google.com");
selenium.Start();
```

Eto toliko je jednostavno :)

## A zašto bi prešli na WebDriver?

WebDriver bolje oponaša korisničke akcije. Za razliku od Selenium RC-a koji koristi JavaScript za automatizaciju, WebDriver se oslanja na ugrađenu podršku za automatizaciju. Ovo je moguće zbog toga što u razvoju WebDriver framework-a aktivno učestvuju i proizvođači pretraživača, kao što su Opera, Mozilla i Google. Drugim rečima, podrška za WebDriver se ugrađuje direktno u pretraživač, što doprinosi brzini i stabilnosti. Na kraju, WebDriver ima manji API što ga čini lakšim za učenje, ali to nikako ne znači da ima manje mogućnosti.
