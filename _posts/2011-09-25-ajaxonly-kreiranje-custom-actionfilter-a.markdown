---
layout: post
title:  "AjaxOnly - Kreiranje custom ActionFilter-a"
date:   2011-09-25 22:48:30 +0100
---

ActionFilter sadrži logiku koja se može pozvati u nekoliko predefinisanih faza izvršavanja action methode. ActionFilter je apstraktna klasa koja nasleđuje FilterAttribute i implementira dva interfejsa: `IActionFilter` i `IResultFilter`. Implementacijom ova dva interfejsa zapravo možemo definisati u kojoj fazi izvršavanja action metode treba da se izvrši logika filtera.

IActionFilter definiše dve metode:

- OnActionExecuting - ova metoda se izvršava pre action metode
- OnActionExecuted - ova metoda se izvršava neposredno po završetku action metode

IResultFilter definiše dve metode i one se izvršavaju nakom metoda IActionFilter interfejsa:

- OnResultExecuting - ova metoda se izvršava pre renderovanja ActionResult-a
- OnResultExecuted - ova metoda se izvršava nakon renderovanja ActionResult-a

Kao što je rečeno ActionFilter nasledjuje FilterAttribute koja kao baznu ima Attribute klasu. Drugim rečima, ActionFilter se koristi kao i svaka druga .Net atribut klasa, s tom razlikom da je namenjen za dekorisanje action metoda.

```csharp
[ChildActionOnly]
public ActionResult MyAction()
{
    // some action logic
}
```

ASP.NET MVC sadrži nekoliko predefinisanih atributa koji se mogu iskoristiti za dekorisanje action metoda:

- Authorize
- ChildActionOnly
- OutputCache
- ValidateAntiForderyToken
- itd.

Filteri se mogu primeniti nad kontrolerima i u tom slučaju će važiti za sve action metode kontrolera.

```csharp
[Authorize]
public class MyController : Controller
{
    // ...
}
```

Pored toga, sa ASP.NET MVC 3 verzijom moguće je definisati i globalne filtere koji će važiti za sve kontrolere, odnosno njihove action metode. Za tu svrhu potrebno je u Application_Start metodi dodati željene filtere.

```csharp
protected void Application_Start()
{
    RegisterGlobalFilters(GlobalFilters.Filters);
    AreaRegistration.RegisterAllAreas();
    RegisterRoutes(RouteTable.Routes);
}
 
public static void RegisterGlobalFilters(GlobalFilterCollection filters)
{
    filters.Add(new HandleErrorAttribute());
}
```

Osnovna ideja filtera je da se određena logika centralizuje kako bi se mogla primeniti na više action metoda. Svrha ActionFilter klase je da posluži kao bazna klasa za custom filtere.

## AjaxOnly

Kao primer kreiranja custom filtera, kreiraćemo AjaxOnly filter koji ima za cilj da filtrira zahteve tako da se action metoda dekorisana sa ovim atributom može okinuti samo u slučaju ajax zahteva.

```csharp
public class AjaxOnlyAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        if (!filterContext.RequestContext.HttpContext.Request.IsAjaxRequest())
        {
            filterContext.HttpContext.Response.RedirectToRoute("PageNotFound");
        }
    }
}
```

Kada neku action metogu dekorišemo sa ovim atributom, pre njenog izvršavanja okinuće se ovaj filter, odnosno OnActionExecuting metoda. Filter proverava da li je u pitanju Ajax zahtev. U slučaju da nije vrši redirekciju na rutu PageNotFound koja, pretpostavljate, se mapira na action metodu koja renderuje PageNotFound pogled.

U slučaju da jeste Ajax zahtev, filter ne radi ništa, odnosno tok izvršavanja se nastavlja normalnim putem i action metoda se izvršava a zatim renderuje odgovarajući pogled.

```csharp
[AjaxOnly]
public ActionResult Vote(int value)
{
    if (ModelState.IsValid)
    {
        //save vote in db
        //return ok
    }
    else
    {
        //return fail
    }
}
```
