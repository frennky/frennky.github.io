---
layout: post
title:  "Dependency Injection - StructureMap"
date:   2011-06-01 14:46:30 +0100
---

[StructureMap](http://structuremap.net/structuremap/) je jedan od najstarijih a samim tim i najzrelijih IoC kontejnera za .Net. Prva stabilna verzija izašla je još 2004., a trenutno aktuelna verzija 2.6.1 je iz februara 2010.

Jednostavna implementacija `IDependencyResolver` interfejsa pomoću StructureMap IoC kontejnera bi izgledala ovako:

```csharp
public class StructureMapDependencyResolver : IDependencyResolver
{
    private readonly IContainer container;
 
    public StructureMapDependencyResolver(IContainer container)
    {
        this.container = container;
    }
 
    public object GetService(Type serviceType)
    {
        try
        {
            return this.container.GetInstance(serviceType);
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
            return this.container.GetAllInstances(serviceType).Cast<object>();
        }
        catch
        {
            return new List<object>();
        }
    }
}
```

Konfigurisanje SturctureMap kontejnera treba izvršiti unutar `Global.asax.cs` fajla:

```csharp
protected void Application_Start()
{
    RegisterStructureMapDependencyResolver();
    RegisterGlobalFilters(GlobalFilters.Filters);
    RegisterRoutes(RouteTable.Routes);
}
 
private void RegisterStructureMapDependencyResolver()
{
    var container = new Container(
        x => x.For<IRepository>().Use<ConcreteRepository>());
             
    DependencyResolver.SetResolver(
        new StructureMapDependencyResolver(container));
}
```
