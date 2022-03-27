---
layout: post
title:  "Generic Controller"
date:   2012-01-30 22:38:30 +0100
---

Kao nastavak na prethodni članak, ovde je opisan jedan od načina realizacije generičkog rešenja za kontrolere. Ono što je zajedničko za većinu kontrolera jeste da sadrže CRUD (Create, Read, Update, Delete) operacije. Međutim, kontroleri se oslanjaju na različite Repository implementacije i manipulišu različitim entitetima. Zbog toga je logično napraviti neku baznu klasu koja obuhvata CRUD operacija ali je ujedno i generička kako bi mogla da radi sa različitim entitetima. Ovaj članak se nadovezuje na prethodni - [Generic Repository](http://ivanfranjic.net/2012/1/generic-repository), tako da je preporučljivo prethodno pročitati i taj članak.

## Generic Controller

Generički kontroler je namerno definisan kao apstraktna klasa, a sve metode kao virtuelne kako bi morali da kreiramo konkretan kontroler za svaki od entiteta i ukoliko je potrebno kreiramo override virtuelne metode.

```csharp
public abstract class GenericController<TModel, TRepository> : Controller where TModel : EntityObject, new() where TRepository : IRepository<TModel>
{
    protected TRepository db;
 
    protected GenericController(IRepository<TModel> repository)
    {
        this.db = (TRepository)repository;
    }
 
    public virtual ActionResult Index()
    {
        var entities = this.db.All();
        return View(entities);
    }
 
    public virtual ActionResult Create()
    {
        return View(new TModel());
    }
 
    [HttpPost]
    public virtual ActionResult Create(TModel entity)
    {
        if (ModelState.IsValid)
        {
            this.db.Add(entity);
            TempData["Success"] = string.Format("New {0} successfully created.", entity.GetType().Name);
            return RedirectToAction("Index");
        }
        else
        {
            TempData["Error"] = string.Format("Unable to create new {0}.", entity.GetType().Name);
            View(entity);
        }
    }
 
    public virtual ActionResult Edit(int id)
    {
        var entity = this.db.Get(id);
        return View(entity);
    }
 
    [HttpPost]
    public virtual ActionResult Edit(TModel entity)
    {
        if (ModelState.IsValid)
        {
            this.db.Update(entity);
            TempData["Success"] = string.Format("{0} successfully updated.", entity.GetType().Name);&
            return RedirectToAction("Index");
        }
        else
        {
            TempData["Error"] = string.Format("Unable to update {0}.", entity.GetType().Name);
            return View(entity);
        }
    }
 
    public virtual ActionResult Delete(int id)
    {
        var entity = this.db.Get(id);
        return View(entity);
    }
 
    [HttpPost]
    public virtual ActionResult Delete(TModel entity)
    {
        this.db.Delete(entity);
        TempData["Success"] = string.Format("{0} successfully deleted.", entity.GetType().Name);
        return RedirectToAction("Index");
    }
}
```

## CateogryController

CateogryController predstavlja konkretan kontroler za Category entitet. Ukoliko su nam dovoljne metode generičkog kontrolera dovoljno je samo definisati konstruktor. Ako se pitate zašto parametarski konsturktor, svrha toga jeste da se omogući [DI](http://ivanfranjic.net/2011/5/dependency-injection).

```csharp
public class CategoryController : GenericController<Category, CategoryRepository>
{
    public CategoryController(IRepository<Category> repository)
        : base(repository)
    {
    }
 
    [HttpPost]
    public override ActionResult Delete(Category category)
    {
        //Performe some logic
        return base.Delete(category);
    }
}
```

Ukoliko postoji razlika u logici nekog od generičkih metoda možemo jednostavno da napišemo override te metode kao što je to urađeno se Delete(Category category) metodom.

## Korak dalje

Ukoliko su nam dovoljne samo metode definisane u GenericController klasi, i ukoliko smatramo da neće biti potrebe za override-ovanjem tih metoda možemo otići korak dalje i kreirati kontroler u run-time-u. Da bi to bilo moguće potrebno je da uradimo dve stvari. Pre svega, potrebno je da GenericController bude konkretna a ne apstraktna klasa kao u primeru gore. A zatim je potrebno kreirati custom ControllerFactory sa sledećom implementacijom CreateController metode:

```csharp
public IController CreateController(RequestContext requestContext, string controllerName)
{
    Type controllerType = Type.GetType("GenericController").MakeGenericType(Type.GetType(controllerName), Type.GetType(controllerName + "Repository"));
    return Activator.CreateInstance(controllerType) as IController;
}
```

Activator će kreirati `GenericController<controllerName, controllerName + "Repository">`, gde je controllerName isto što i ime entiteta. Drugim rečima umesto CategoryController kao iz primera sa početka posta, dobićemo `GenericController<Category, CaterogyRepository>`. Na ovaj način bi se oslobodili potrebe da kodiramo konkretan kontroler za svaki od entiteta.
