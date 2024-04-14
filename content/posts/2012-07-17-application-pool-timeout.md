---
title:  "Application pool timeout usled neaktivnosti web aplikacije"
date:   2012-07-17 12:01:30 +0100
---

Ukoliko sajt nema posetu duže od 20 minuta, application pool će se ugasiti i osloboditi sistemske resurse. Kada sledeći zahtev dođe, IIS će automatski startovati application pool i poslati traženu starnicu.

Ovo je odoličan način da se čuvaju resursi, ali to takođe znači da prvi zahtev, onaj koji dovodi do startovanja aplikacije, je vrlo spor. Spor je zato što proces mora da se startuje, učita sve potrebne biblioteke, izvrši Application_Start metodu i na kraju procesira zahtev. U zavisnosti od veličine i složenosti aplikacije, ovo može da potraje od nekoliko sekundi do više od pola minuta.

Podrazumevana dužina perioda neaktivnosti nakon koga dolazi do timeout-a je 20 minuta i predstavlja vreme koje worker process provede neaktivan pre nego što dođe do isključivanja aplikacije. Worker process se smatra neaktivnim ako ne obrađuje zahteve i ne primi nijedan novi zahtev.

Ukoliko je potrebno da se timeout isključi u potpunosti, potrebno je setovati ga na 0, što zanči da se application pool neće nikad ugasiti ukoliko nema poseta.

Svrha ove funkcionalnosti IIS-a jeste oslobađanje resursa ukoliko je aplikacija neaktivna. Ovo je posebno korisno ukoliko se na IIS serveru nalazi više sajtova i aplikacija, a pritom je podešeno deljenje resursa između sajtova i aplikacija.

Podešavanje željene vrednosti timeout-a se može izvršiti pomoću sledeće komande:

```
%windir%\system32\inetsrv\appcmd set config -section:applicationPools -applicationPoolDefaults.processModel.idleTimeout:00:00:00
```

Windows Azure
Ukoliko koristite Windows Azure, da bi podesili timeout na nula, potrebno je dodati deklaraciju startup task-a u ServiceDefinition.csdef:

```xml
<Startup> 
    <Task commandLine="startup\disableTimeout.cmd" executionContext="elevated" />
</Startup>
```

Nakon toga potrebno je dodati disableTimeout.cmd u folder startup. Sadržaj ovog fajla treba da bude poziv appcmd konzoli da setuje application pool idle timeout na nula.

Više detalja može se naći na ovoj [adresi](http://technet.microsoft.com/en-us/library/cc771956.aspx).
