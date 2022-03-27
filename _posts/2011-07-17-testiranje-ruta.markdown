---
layout: post
title:  "Testiranje ruta"
date:   2011-07-17 23:36:30 +0100
---

ASP.NET MVC framework je osmišljen tako da može lako da se testira. Medutim, u nekim slucajevima testiranje MVC aplikacije može biti složeniji posao.

Mehanizam za rutiranje se direktno oslanja na ASP.NET i napravljen je tako da radi i sa ASP.NET WebForms i sa ASP.NET MVC framework-om. Zbog toga je za testiranje ruta potrebno kreirati dosta Mock objekata. Iako ovo nije teško uraditi, može da bude iscrpljujuće.

Na svu sreću postoji bibiloteka koja pomaže u instanciranju kontrolera i pravilnom inicijalizovanju njegovih članica:

- HttpContext
- HttpRequest
- HttpResponse
- HttpSession
- Form
- TempData
- QueryString
- ApplicationPath
- PathInfo
- itd.

U okviru Codeplex projekta MvcContrib postoji biblioteka [TestHelper](http://mvccontrib.codeplex.com/wikipage?title=TestHelper&referringTitle=Documentation) koja sadrži sve što nam je potrebno za testiranje ruta. MvcContrib TestHelper može da se doda u test projekat kao NuGet paket:

```
PM> Install-Package MvcContrib.Mvc3.TestHelper-ci
```

Nakon što smo dodali referencu na TestHelper biblioteku, potrebno je da učitamo rute. To ćemo učiniti u okviru ClassInitialize metode:

```csharp
[ClassInitialize]
public static void MyClassInitialize(TestContext testContext)
{
    RouteTesting.Demo.MvcApplication.RegisterRoutes(RouteTable.Routes);
}
```

Sada možemo da pišemo testove za rute:

```csharp
[TestMethod]
public void GET_Account_LogOn()
{
    "~/Account/LogOn".ShouldMapTo<AccountController>(ac => ac.LogOn());
}
 
[TestMethod]
public void POST_Account_LogOn()
{
    "~/Account/LogOn".WithMethod(HttpVerbs.Post).ShouldMapTo<AccountController>(ac => ac.LogOn(null, null));
}
```
