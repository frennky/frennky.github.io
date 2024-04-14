---
title:  "Baseless merge"
date:   2012-03-12 23:27:30 +0100
---

Cilj ovog članka jeste da objasni šta je baseless merge i kako sprovesti takav merge. Razlog zašto pišem baš o ovome proizliazi i toga da sam više puta bio u situaciji da mi se developeri, pa čak i oni sa više iskustva, obraćaju sa istim problemom: kako izvršiti spajanje grana koje nisu u direktnoj (roditelj-dete) vezi.

## Šta je zapravo baseless merge?

Baseless merge je proces spajanja (merging) dva fajla (ili celih foldera) koji se nalaze na granama koje nisu u direktnoj vezi.

Potreba za ovakvim spajanjem grana može se javiti, na primer, ako želimo da spojimo fajlove/foldere sa dve srodne (sibling) grane, a da pri tom ne vršimo spajanje sa izvornom (roditeljskom) granom.

Baseless merge, s obzirom da ne nosi dovoljno informacija o vezama izmedju fajlova/foldera, može zahtevati više ručnog rešavanja konflikata nego prilikom običnog spajanja grana. Tako na primer, ukoliko je neki fajl preimenovan, ova operacija će se shvatiti kao brisanje fajla i dodavanje novog fajla. Zbog toga, pre svakog baseless merge-a treba dobro razmisliti da li je to stvarno potrebno ili to možemo rešiti na neki drugi način.

## Kako sprovesti baseless merge?

Kako potreba za sprovođenjem baseless merge-a najčešće ukazuje na postojanje propusta u procesu razvoja, a pored toga ovaj proces se retko sprovodi, nije ni čudo što opcija za baseless merge nije dostupna direktno iz Visual Studio-a. Da bi sproveli baseless merge, moramo to učiniti pomoću tf console aplikacije. U tf console aplikaciji pri pozivu komande za obično spajanje grana, potrebno je dodati flag /baseless kako bi naznačili da smo svesni da grane nisu u direktnoj vezi i da ipak želimo da sprovedemo spajanje.

```
tf merge /baseless /recursive <source> <destination>
```

Ukoliko dođe do konflikata, otvoriće se UI za rešavanje konflikata, a nakon što rešimo sve konflikte potrebno je uraditi checkin.

Više informacija o merge komandi možete videti na [MSDN](http://msdn.microsoft.com/en-us/library/bd6dxhfy.aspx)-u.
