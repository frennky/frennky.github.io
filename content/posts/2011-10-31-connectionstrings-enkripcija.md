---
title:  "ConnectionStrings enkripcija"
date:   2011-10-31 23:48:30 +0100
---

S obzirom da se celokupna konfiguarcija ASP.NET aplikacije nalazi u čistom obliku u web.config fajlu, uključujući i connection stringove, ovo svakako predstavlja sigurnosnu pretnju.

Na svu sreću .Net poseduje mehanizme kojima se ova pretnja lako može otkloniti. Postoje provajderi enkripcije koji mogu da izvrše šifromanje i dešifrovanje određenih web.config sekcija.

U primeru koji sledi, prikazana je upotreba RSAProtectedConfigurationProvider-a za šifrovanje i dešifrovanje connectionStrings sekcije.

Napravićemo novu ASP.NET MVC aplikaciju i izmeniti Index akciju HomeController-a tako da sadrži sledeći kod:

```csharp
public ActionResult Index()
{
    ViewBag.ConnectionStringsSection = GetConnectionStringsSection();
    return View();
}
```

Zatim ćemo u HomeController-u napraviti privatnu metodu GetConnectionStringsSection:

```csharp
private string GetConnectionStringsSection()
{
    var config = WebConfigurationManager.OpenWebConfiguration(Request.ApplicationPath);
    var xml = XDocument.Load(config.FilePath);
    var connectionStringsSection = xml.Descendants().SingleOrDefault(e => e.Name == "connectionStrings");
    return connectionStringsSection != null ? connectionStringsSection.ToString() : string.Empty;
}
```

A potom ćemo promeniti Index view tako da izgleda ovako:

```html
<div>
    <h2>Connection string encryption example</h2>
    <p>@Html.ActionLink("Encrypt", "Encrypt")</p>
    <p>@Html.ActionLink("Decrypt", "Decrypt")</p>
</div>
<div>
    <h3>Connection Strings section</h3>
    <p>@ViewBag.ConnectionStringsSection</p>
</div>
```
## Enkripcija

Za enkripciju connectionStrings sekcije napravićemo Encryption akciju u HomeController-u:

```csharp
public ActionResult Encrypt()
{
    EncryptConnectionStrings();
    return RedirectToAction("Index");
}
```

Suština je naravno u metodi EncryptConnectionStrings koja izgleda ovako:

```csharp
public void EncryptConnectionStrings()
{
    var config = WebConfigurationManager.OpenWebConfiguration(Request.ApplicationPath);
    var section = config.ConnectionStrings;
    if (section.SectionInformation.IsProtected) return;
 
    section.SectionInformation.ProtectSection("RsaProtectedConfigurationProvider");
    config.Save();
}
```

## Dekripcija

Slično enkripciji, za dekripciju connectionStrings sekcije napravićemo Decryption akciju u HomeController-u:

```csharp
public ActionResult Decrypt()
{
    DecryptConnectionStrings();
    return RedirectToAction("Index");
}
```

I naravno, suština je u metodi DecryptConnectionStrings koja izgleda ovako:

```csharp
private string GetConnectionStringsSection()
{
    var config = WebConfigurationManager.OpenWebConfiguration(Request.ApplicationPath);
    var xml = XDocument.Load(config.FilePath);
    var connectionStringsSection = xml.Descendants().SingleOrDefault(e => e.Name == "connectionStrings");
    return connectionStringsSection != null ? connectionStringsSection.ToString() : string.Empty;
}
```

Treba imati na umu da su enkripcija i dekripcija skupe operacije, kao i da svaka izmena web.config fajla dovodi do restarta aplikacije.

Pored ovakvog, programskog načina enkripcije i dekripcije, moguće je ovo postići i pomoću aspnet_regiis.exe konzole. Više informacija o tome možete naći na MSDN-u.
