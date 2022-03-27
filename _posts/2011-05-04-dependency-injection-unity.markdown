---
layout: post
title:  "Dependency Injection - Unity"
date:   2011-05-04 21:39:30 +0100
---

Jedno od glavnih poboljšanja koje nam MVC 3 donsi jeste bolja integracija sa [IoC](http://en.wikipedia.org/wiki/Inversion_of_control)/[DI](http://en.wikipedia.org/wiki/Dependency_injection) kontejnerima. U ranijim verzijama MVC framework-a bilo je potrebno napraviti custom ControllerFactory. Međutim, postoji veliki broj IoC framework-a tako da je prirodnije da se komunikacija sa MVC framework-om odvija preko nekog interfejsa.

Za tu svrhu postoji jedan vrlo jednostavan interfejs - [IDependencyResolver](http://msdn.microsoft.com/en-us/library/system.web.mvc.idependencyresolver.aspx):

```csharp
public interface IDepndencyResolver
{
    object GetService(Type serviceType);
    IEnumerable<object> GetServices(Type serviceType);
}
```

I upravo implementacije ovog interfejsa treba da delegiraju zahteve za tipovima konkretnom IoC kontejneru.

UnityDependencyResolver:

```csharp
public class UnityDependencyResolver : IDependencyResolver
{
    private readonly IUnityContainer container;
 
    public UnityDependencyResolver(IUnityContainer container)
    {
        this.container = container;
    }
 
    public object GetService(Type serviceType)
    {
        try
        {
            return this.container.Resolve(serviceType);
        }
        catch
        {
            return null;
    }
 
    public IEnumerable<object> GetServices(Type serviceType)
    {
        try
        {
            return this.container.ResolveAll(serviceType);
        }
        catch
        {
            return new List<object>();
        }
    }
}
```

Ovde treba obratiti pažnju na to da ukoliko GetService nije u mogućnosti da vrati servis, treba da vrati null. I slično, ukoliko GetServices ne može da vrati servise treba da vrati praznu kolekciju. Ovo je važno jer ukoliko dođe do izuzetka on će biti vraćen korisniku kao run-time greška.

Nakon sto smo implementirali `IDependencyResolver` interfejs, potrebno je napuniti IoC kontejner:

```csharp
protected void Application_Start()
{
    RegisterDependencyResolver();
    RegisterGlobalFilters(GlobalFilters.Filters);
    RegisterRoutes(RouteTable.Routes);
}
 
private void RegisterDependencyResolver()
{
    var container = new UnityContainer()
                        .RegisterType<IRepository, ConcreteRepository>()
                        .RegisterType<IService, ConcreteService>();
    DependencyResolver.SetResolver(new UnityDependencyResolver(container));
}
```
