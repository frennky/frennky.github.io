---
layout: post
title:  "Custom HandleErrorAttribute"
date:   2011-10-15 15:05:30 +0100
---

U okviru ASP.NET MVC framework-a postoji `HandleErrorAttribute` koji se može koristiti za centralizovanu obradu izuzetaka u MVC aplikaciji. Međutim, već prilikom razvoja prve ASP.NET MVC aplikacije pokazalo se da ovaj atribut nema sve mogućnosti koje su mi potrebne.

Ono što ne dostaje jeste mogućnost da se na neki način kontroliše nivo informacija o grešci koji će se prikazati. Drugim rečima, želim da prilikom debug-a mogu da vidim ceo stack trace, dok za release, odnosno za krajnjeg korisika, želim da se prikazuje neka generička poruka o grešci.

## HandleExceptionAttribute

Da bih rešio gore navedeni problem na MVC način, napravio sam novu klasu koja proširuje HandleErrorAttribute.

```csharp
public class HandleExceptionAttribute : HandleErrorAttribute
{
    public HandleExceptionAttribute()
    {
        this.Master = "_MyLayout";
        this.View = "Error";
    }
 
    public override void OnException(ExceptionContext filterContext)
    {
        if (filterContext.IsChildAction) return;
 
        if (IsDebugMode())
        {
            base.OnException(filterContext);
        }
        else
        {
            filterContext.Result = new ViewResult
            {
                ViewName = this.View,
                MasterName = this.Master,
                ViewData = filterContext.Controller.ViewData,
                TempData = filterContext.Controller.TempData
            };
            filterContext.HttpContext.Response.Clear();
            filterContext.HttpContext.Response.StatusCode = 500; //Internal Server Error 
            filterContext.HttpContext.Response.TrySkipIisCustomErrors = true;
            filterContext.ExceptionHandled = true;
        }
    }
 
    public bool IsDebugMode()
    {
        var compilationSection = WebConfigurationManager.GetSection("system.web/compilation") as CompilationSection;
        return compilationSection != null ? compilationSection.Debug : false;
    }
 
}
```

Suština ovog atributa je u IsDebugMode metodi koja proverava da li je aplikacija u debug modu tako što proveri podešavanja u web.config fajlu. Ukoliko jeste, exception se obrađuje na isti način kao i kod HandleErrorAttribute, u suprotnom prikazujemo neki pogled sa generičkom porukom o grešci.

Na kraju, da bi ovo funkcinisalo bez potrebe za ručnim setovanjem debug atributa u `web.config` fajlu, možemo definisati web.config transformaciju. U samom web.config fajlu podesićemo:

```xml
<system.web>
    <compilation  debug="true" targetFramework="4.0" defaultLanguage="c#" />
</system.web>
```

A zatim u `web.Release.config` fajlu:

```xml
<system.web>
    <compilation xdt:Transform="SetAttributes" debug="false" />
</system.web>
```
