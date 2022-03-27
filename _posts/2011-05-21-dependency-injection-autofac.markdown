---
layout: post
title:  "Dependency Injection - Autofac"
date:   2011-05-21 13:31:30 +0100
---

[Autofac](http://code.google.com/p/autofac/) je još jedan IoC kontejner za .Net, takođe vrlo popularan. Odlikuje ga manja veličina biblioteka u odnosu na ostale IoC kontejnere, ali ne i nedostatak funkcionalnosti.

Jednostavna implementacija [IDependencyResolver](http://msdn.microsoft.com/en-us/library/system.web.mvc.idependencyresolver.aspx) interfejsa pomoću Autofac framework-a bi izgledala ovako:

```csharp
public class AutofacDependencyResolver : IDependencyResolver
{
    private IContainer container;
 
    public AutofacDependencyResolver(Autofac.IContainer container)
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
    }
 
    public IEnumerable<object> GetServices(Type serviceType)
    {
        try
        {
            Type type = typeof(IEnumerable).MakeGenericType(
            new Type[]
            {
                serviceType
            });
            object obj = this.container.Resolve(type);
            return (IEnumerable<object>)obj;
        }
        catch
        {
            return new List<object>();
        }
    }
}
```

Setup Autofac IoC kontejnera treba staviti u `Global.asax.cs` fajl:

```csharp
protected void Application_Start()
{
    RegisterAutofacDependencyResolver();
    RegisterGlobalFilters(GlobalFilters.Filters);
    RegisterRoutes(RouteTable.Routes);
}
 
private void RegisterAutofacDependencyResolver()
{
    var builder = new ContainerBuilder();
    builder.RegisterType<HomeController>();
    builder.RegisterType<ConcreteRepository>().As<IRepository>();
 
    DependencyResolver.SetResolver(
    new AutofacDependencyResolver(builder.Build()));
}
```
