---
title:  "Compile-time view checking"
date:   2011-03-18 20:09:30 +0100
---

Prilikom razvoja web aplikacija, pogledi (views) su deo aplikacije koji se najčešće menjaju. Vrlo lako se može napraviti slovna greška koja može da poremeti prikaz celog pogleda. Sam kod koji se nalazi u pogledu, se ne kompajlira sve dok IIS ne pokuša da renderuje pogled.

Sigurno se i vama desilo da izmenite neki Html helper, možda samo potpis metode, a da ne izmenite sve pozive ovog helpera u pogledima koji ga koriste. Ovo dovodi do pucanja stranice u neočekivanim trenutcima, možda par dana nakon izmene, nakon što ste i zaboravili da ste menjali Html helper.

Na svu sreću, postoji način da Visual Studio kompajlira poglede prilikom builda aplikacije. Međutim, ova funkcionalnost nije podržana kroz GUI, već je potrebno izmeniti .csproj fajl.U okviru .csproj fajla, koji se može otvoriti pomoću nekog xml editora, ili običnog text editora, nalazi se MvcBuildViews element, čije je podrazumevano podešavanje false i treba ga podesiti na true. Nakon što se sačuva ova izmena, Visual Studio će kompajlirati poglede prilikom svakog builda.

```xml
<MvcBuildViews>true</MvcBuildViews>
```

Nažalost, ovo je potrebno podesiti ručno za svaki novi projekat, a pored toga povećava compile-time i do nekoliko puta (u zavisnosti koliko pogleda ima). Rešenje je da se compile-time view checking opcija uključi za Release konfiguraciju builda. Što se može postići definisanjem MvcBuildViews elementa u PropertyGroup-i koja ima odgovarajući uslov.

```xml
<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Release|AnyCPU' ">
```
