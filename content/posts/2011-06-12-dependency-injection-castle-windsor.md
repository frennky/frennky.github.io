---
title:  "Dependency Injection - Castle Windsor"
date:   2011-06-12 15:33:30 +0100
---

Poslednji IoC kontejner u ovom serijalu je [Castle Windsor](http://www.castleproject.org/container/). Ovaj IoC kontejner je deo [Castle](http://www.castleproject.org/castle/projects.html) projekta iza koga stoji grupa iskusnih .Net developer-a.

Jednostavna implementacija `IDependencyResolver` interfejsa pomoću Castle Windsor IoC kontejnera bi izgledala ovako:

```csharp
public class CastleWindsorDependencyResolver : IDependencyResolver
{
    private IKernel kernel;
 
    public CastleWindsorDependencyResolver(IKernel kernel)
    {
        this.kernel = kernel;
    }
 
    public object GetService(Type serviceType)
    {
        try
        {
            return this.kernel.Resolve(serviceType);
        }
        catch
        {
            return null;
        }
    }
 
    public IEnumerable<object> GetServices(Type serviceType)
    {
        try
        {
            return this.kernel.ResolveAll(serviceType).Cast<IEnumerable<object>>();
        }
        catch
        {
            return new List<object>();
        }
    }
}
```

Na kraju, konfigurisanje Castle Windsor IoC kontejnera treba izvršiti unutar `Global.asax.cs` fajla:

```csharp
protected void Application_Start()
{
    RegisterCastleWindsorDependencyResolver();
    RegisterGlobalFilters(GlobalFilters.Filters);
    RegisterRoutes(RouteTable.Routes);
}
 
private void RegisterCastleWindsorDependencyResolver()
{
    var container = new WindsorContainer()
        .Register(Component.For<HomeController>())
        .Register(Component.For<IRepository>().ImplementedBy<ConcreteRepository>());
         
    DependencyResolver.SetResolver(
        new CastleWindsorDependencyResolver(container.Kernel));
}
```
