---
layout: post
title:  "Dependency Injection"
date:   2011-05-12 22:54:30 +0100
---

U skoro svakom tutorijalu o ASP.NET MVC framework-u se pominje [Dependency Injection](http://martinfowler.com/articles/injection.html). Međutim, IoC framework-a ima dosta, a svako ima neki omiljeni.

U svim tim tutorijalima kada pišu o DI, autori uvek prezentuju jedan od IoC framwork-a, tako da je teško naći tutorijal ili blog na kome su dati primeri za više IoC framework-a. Zbog toga je teško odlučiti se koji koristiti, naročito ako ste tek počeli da učite.

Zbog toga sam odlučio da napišem seriju postova u kojima ću prikazati neke od najpopularnijih IoC framework-a.

Primeri u ovoj seriji postova obuhvataju sledeće IoC framework-e:

- [Unity](http://ivanfranjic.net/2011/5/dependency-injection-unity)
- [Ninject](http://ivanfranjic.net/2011/5/dependency-injection-ninject)
- [Autofac](http://ivanfranjic.net/2011/5/dependency-injection-autofac)
- [StructureMap](http://ivanfranjic.net/2011/6/dependency-injection-structuremap)
- [Castle Windsor](http://ivanfranjic.net/2011/6/dependency-injection-castle-windsor)

Jedno od glavnih poboljšanja koje nam MVC 3 donsi jeste bolja integracija sa IoC/DI kontejnerima. U ranijim verzijama MVC framework-a bilo je potrebno napraviti custom ControllerFactory. U MVC 3 potrebno je samo implementirati jedan jednostavan interfejs:

```csharp
public interface IDepndencyResolver
{
    object GetService(Type serviceType);
    IEnumerable<object> GetServices(Type serviceType);
}
```

Treba napomenuti da ukoliko pri pozivu GetService metode IoC kontejner nije u stanju da vrati servis, treba da vrati null. Slično tome ukoliko pri pozivu GetServices nije u stanju da vrati servise treba da vrati praznu kolekciju. Ovo je vrlo bitno, jer ukoliko dođe do greške ona će biti vraćena korisniku kao run-time greška.
