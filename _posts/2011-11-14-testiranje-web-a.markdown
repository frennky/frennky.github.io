---
layout: post
title:  "Testiranje web-a"
date:   2011-11-14 10:53:30 +0100
---

Problemi koji postoje u automatskom testiranju web aplikacija su slični problemima pri testiranju desktop aplikacija. Kod web aplikacija imamo različite internet pretraživače kao i različite verzije pretraživača. S druge strane, kod desktop aplikacija imamo različite platforme (Delphi, Java, .Net), a postoje i značajne razlike u pristupu automatizaciji u zavisnosti od UI framework-a (kao npr. Win32, WinForms, WPF). Zbog toga retko koji framework za testiranje UI-a podržava više različitih platformi ili pretraživača.

Pre par godina imao sam prvi put priliku da se oprobam u testiranju web aplikacija. Do data sam već imao dobrog iskustva u testiranju desktop aplikacija. Kako smo za svoje potrebe kreirali framework za testiranje WinForms i Delphi aplikacija, znao sam dobro koliko je to veliki posao. Zbog toga sam prvo konsultovao Google kako bih saznao da li postoji neki framework za web automatizaciju.

Nažalost u to vreme open source nije nudio mnogo rešenja. Među nekoliko rešenja, istakli su se Selenium RC i WatiN. Međutim, Selenium smo odbacili iz ne sećam se kojih razloga, dok je WatiN bio još uvek na početku razvoja. Na kraju smo ipak odlučili da napravimo sopstveno lightweight rešenje. Sve što je bilo potrebno jeste da se domognemo DOM-a, a posle je sve išlo lako. Framework koji smo kreirali se oslanjao na [mshtml](http://msdn.microsoft.com/en-us/library/aa741312.aspx) biblioteku i podržavao je automatizaciju Internet Explorer-a.

Posle nekog vremena prešao sam da radim na auto testovima za WPF aplikacije, a naš web framework je ostao zaboravljen. Od tada je prošlo dosta vremena, a ja sam opet dobio priliku da radim na autotestovima za web aplikacije.

Po pitanju framework-a za testiranje web-a, situacija je danas znatno bolja. WatiN je prilično zreo framework, sadrži apstrakcije UI kontrola i stranica, tako da prirodno podržava Page pattern.

S druge strane, Selenium je pretrpeo znatnu izmenu u API-ju sa novom verzijom. Selenium 2, ili kako ga još zovu WebDriver, ima mali i vrlo jednostavan API tako da se vrlo brzo uči. Za Selenium WebDriver postoji pomoćna biblioteka koja omugćava kreiranje Page objekata, odnosno primenu Page paterna.

Iako WatiN možda bolje koristi mogućnosti C# jezika, ipak se Selenium WebDriver pokazao prijatniji za rad. Pored toga, ne treba zaboraviti da Selenium ima prilično veliku zajednicu, kao i da podržava veći broj internet pretraživača.

U narednom periodu potrudiću se da pišem o problemima na koje sam nailazio prilikom kreiranja testova pomoću Selenium WebDriver-a.
