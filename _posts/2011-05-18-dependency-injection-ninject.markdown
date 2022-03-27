---
layout: post
title:  "Dependency Injection - Ninject"
date:   2011-05-18 22:12:30 +0100
---

[Ninject](http://ninject.org/) je open source DI framework, besplatan za ličnu i komercijalnu upotrebu. U .Net zajednici možda i najpopularniji. Odlikuje ga jednostavna i simpatična sintaksa, kao i vrlo dobra [dokumentacija](https://github.com/ninject/ninject/wiki) koja do detalja objašnjava njegovu upotrebu.

Jednostavna implementacija [IDependencyResolver](http://msdn.microsoft.com/en-us/library/system.web.mvc.idependencyresolver.aspx) interfejsa pomoću Ninject DI framework-a bi izgledala ovako:

```csharp
public class NinjectDependencyResolver : IDependencyResolver
{
    private readonly IKernel kernel;
 
    public NinjectDependencyResolver(IKernel kernel)
    {
        this.kernel = kernel;
    }
 
    public object GetService(Type serviceType)
    {
        try
        {
            return this.kernel.Get(serviceType);
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
            return this.kernel.GetAll(serviceType);
        }
        catch
        {
            return new List<object>();
        }
    }
}
```

Na kraju, potrebno je podesiti IoC kontejner unutar `Global.asax.cs` fajla:

```csharp
protected void Application_Start()
{
    RegisterNinjectDependencyResolver();
    RegisterGlobalFilters(GlobalFilters.Filters);
    RegisterRoutes(RouteTable.Routes);
}
 
private void RegisterNinjectDependencyResolver()
{
    var kernel = new StandardKernel();
    kernel.Bind<IRepository>().To<ConcreteRepository>();
    DependencyResolver.SetResolver(new NinjectDependencyResolver(kernel));
}
```
