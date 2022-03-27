---
layout: post
title:  "Generic Repository"
date:   2012-01-17 22:28:30 +0100
---

U ASP.NET MVC zajednici jako je popularna upotreba Repository paterna za rad sa podacima, zbog toga gotovo da ne postoje primeri koji se ne oslanjaju na ovaj patern. Više informacija o Reposiotry paternu može se naći na [MSDN](http://msdn.microsoft.com/en-us/library/ff649690.aspx)-u ili na sajtu [Martin-a Fowler-a](http://martinfowler.com/eaaCatalog/repository.html), a u ovom postu je prikazan kod koji ilustruje kako može da se implementira generički Repository nad Entity Framework-om.

## IRepository<T>

Ovaj generički interfejs definiše metode koje zahtevaju sve konkretne Repository klase.

```csharp
public interface IRepository<T> where T : class
{
    IQueryable<T> All();
    IQueryable<T> GetAll(Func<T, bool> exp);
    T Get(Func<T, bool> exp);
    T Get(int id);
    void Add(T entity);
    void Update(T entity);
    void Delete(int id);
}
```

## EntityRepository<T>

EntityRepository predstavlja baznu klasu za sve konkretne Repository klase, implementira IRepository interfejs i dodaje jednu apstraktnu metodu - Map(T entity). Ovu metodu je potrebno implementirati u konkretnoj Repository klasi, što ćemo kasnije i videti, i namerno je uvedena jer se u praksi pokazalo da često ne želimo da dozvolimo ažuriranje svih propertija entiteta.

```csharp
public abstract class EntityRepository<T> : IRepository<T> where T : EntityObject
{
    protected MyEntities db;
    private string entityKeyName;
    private ObjectSet<T> entitySet;
 
    protected EntityRepository(MyEntities context)
    {
        this.db = context;
        this.entitySet = this.db.CreateObjectSet<T>();
        // only simple keys are supported!
        this.entityKeyName = this.entitySet.EntitySet.ElementType.KeyMembers[0].Name;
    }
 
    protected abstract void Map(T entity);
 
    public virtual IQueryable<T> All()
    {
        return this.entitySet;
    }
 
    public virtual IQueryable<T> GetAll(Func<T, bool> exp)
    {
        return this.entitySet.Where(exp).AsQueryable<T>();
    }
 
    public virtual T Get(Func<T, bool> exp)
    {
        return this.entitySet.SingleOrDefault<T>(exp);
    }
 
    public virtual T Get(int id)
    {
        var itemParameter = Expression.Parameter(typeof(T), "item");
        var binaryExpression = Expression.Equal(
                Expression.Property(itemParameter, this.entityKeyName),
                Expression.Constant(id));
        var expression = Expression.Lambda<Func<T, bool>>(binaryExpression, new[] { itemParameter });
        return Get(expression.Compile());
    }
 
    public virtual void Add(T entity)
    {
        this.entitySet.AddObject(entity);
        this.db.SaveChanges();
    }
     
    public virtual void Update(T entity)
    {
        Map(entity);
        this.db.SaveChanges();
    }
 
    public virtual void Delete(int id)
    {
        this.entitySet.DeleteObject(Get(id));
        this.db.SaveChanges();
    }
}
```

Get metoda možda deluje zastrašujuće ali zapravo radi vrlo jednostavnu stvar - kreira lambda izraz na osnovu koga će se filtrirati Repository. Na primer, ako imamo entitet Category koji ima primarni ključ CategoryId, izraz koji će se generisati je item => item.CategoryId == id, gde id ima neku konkretnu vrednost.

Očigledna prednost ovakvog generičkog dizajna je u tome što nam omogućava upotrebu ovog koda za bilo koji projekat. S druge strane, mana je to što se neke stvari podrazumevaju, odnosno zasniva se na konvenciji. Na primer, podrazumeva da svi entiteti imaju prost primarni ključ. Ako pogledamo malo bolje Get metodu u kojoj se kreira lambda izraz, može se primetiti da se koristi samo jedan parametar kao naziv ključa.

Naravno, moguće je implementirati kreiranje izraza koji sadrži i složeni ključ ali to pored usložnjavanja utiče i na performanse metode. Ovde je namerno izabrana pomenuta konvencija jer je to sasvim dovljno za jednostavne web aplikacije.

## CategoryRepository

Za primer konkretne Repository klase uzećemo Repository koji radi sa Category entitetima. CategoryRepository nasleđuje baznu EntityRepository klasu i implementira apstraktnu Map metodu.

```csharp
public class CategoryRepository : EntityRepository<Category>
{
    public CategoryRepository(MyEntities context)
        : base(context)
    {
    }
 
    protected override void Map(Category original, Category entity)
    {
        original.Name = entity.Name;
        original.Description = entity.Description;
        original.Modefied = entity.Modefied;
    }
}
```

Category entitet sadrži još neke propertije koje ne želimo da ažuriramo (npr. Created) i upravo je zbog toga potrebno implementirati Map metodu koja će ažurirati samo željene propertije.

Na kraju, da pomenem da se ova implementacija oslanja na Entity Framework (Model first), te da ukoliko bi koristili EF Code first pristup imali bi i jednostavniju implementaciju.Eto nešto o čemu možete da razmišljate :)

Sledećom prilikom ćemo se pozabaviti generičkim kontrolerom koji se oslanja na ovaj generički Repository.
