---
title:  "GZip i shared hosting"
date:   2012-05-30 23:58:30 +0100
---

Upotreba kompresije je svakako jedna od tehnika optimizacije web aplikacija. Ukoliko nemamo kontrolu nad IIS konfiguracijom, ili nemamo dovoljno prava, kao npr. u slučaju shared hosting rešenja, možemo uključiti GZip kompresiju kroz web.config same web aplikacije.

Sve što je potrebno jeste dodati ovaj node u web.config fajl i to će uključiti GZip kompresiju za datu web aplikaciju.

```xml
<system.webServer>
    <httpCompression directory="%SystemDrive%\inetpub\temp\IIS Temporary Compressed Files">
        <scheme name="gzip" dll="%Windir%\system32\inetsrv\gzip.dll"/>
        <dynamicTypes>
            <add mimeType="text/*" enabled="true"/>
            <add mimeType="application/javascript" enabled="true"/>
            <add mimeType="*/*" enabled="false"/>
        </dynamicTypes>
        <staticTypes>
            <add mimeType="text/*" enabled="true"/>
            <add mimeType="application/javascript" enabled="true"/>
            <add mimeType="*/*" enabled="false"/>
        </staticTypes>
    </httpCompression>
    <urlCompression doStaticCompression="true" doDynamicCompression="true"/>
</system.webServer>
```

Obratite pažnju da je potrebno definisati mime tipove za koje će biti omogućena, odnosno isključena kompresija.
