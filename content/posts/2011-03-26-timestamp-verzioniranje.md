---
title:  "Timestamp verzioniranje"
date:   2011-03-26 21:19:30 +0100
---

Pre neki vikend za potrebe verzioniranja ove blog aplikacije napravio sam jednostavan alat. S obzirom da je u pitanju aplikacija na kojoj radim samo ja i koja se trenutno sastoji od jednog projekta, nisam imao potrebu ni za kakvim komplikovanim verzioniranjem, već mi je bila dovoljna verzija u vidu timestamp-a.

Alat je krajnje jednostavan ali baš zbog toga mi se i sviđa. U pitanju je console aplikacija koja ažurira AssemblyInfo.cs fajl sa odgovarajućim formatom timestamp-a. Inicijalno je parsiranje argumenata rešeno preko Linq-a, ali sam posle pretraživanja interneta pronašao jedan vrlo zgodan komad koda za te svrhe ([NDesk.Options](http://www.ndesk.org/Options)).

Potrebno je proslediti DateTime formate za Major, Minor, Build i Version i alat će ažurirati `AssemblyInfo.cs` fajl. Pored ova 4 parametra, može se proslediti naziv i putanja do fajla koji sadrži verziju, ukoliko nije u pitanju podrazumevana putanja - `Properties\AssemblyInfo.cs`.

Da bi automatizovali ovaj proces, potrebno je u podešavanjima projekta podesiti Pre-build event na sledeći način:

```powershell
"C:\Tools\Versioner.exe" /M=yyyy /m=MM /b=dd /r=HHmm /a="$(ProjectDir)Properties\AssemblyInfo.cs"
```

Ova komanda će pri svakom build-u automatski ažurirati verziju. Ukoliko vam ne odgovara ovakav format, može se proslediti bilo koji string koji je validan za formatiranje DateTime vrednosti. Interno za svaki od prosleđenih argumenata (Major, Minor, Build i Version), poziva se `DateTime.Now.ToString(versionFormat)`. Na kraju još da napomenem da nije potrebno sve argumente proslediti već samo one koje želte da ažurirate. Nadam se da će još nekome ovo koristiti.

Alat možete preuzeti ovde.
