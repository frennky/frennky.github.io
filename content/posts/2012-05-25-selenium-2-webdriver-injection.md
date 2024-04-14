---
title:  "Selenium 2 WebDriver injection"
date:   2012-05-25 12:01:30 +0100
---

Selenium 2 WebDriver ima podršku za nekoliko browser-a:

- FireFox 3.6 - 10
- Internet Explorer 7 - 9
- Opera 8 - 9
- Chrome
- itd.

Kako svi drajver implementiraju `IWebDriver` interfejs, prirodno je da se testovi pišu korišćenjem ovog interfejsa a ne konkretnog drajvera. Na taj način testovi će raditi sa bilo kojim internet pretraživačem. Međutim, tu se nameće problem kako instancirati konkretan drajver.

## Podešavanje Unity framework-a

Konfigurisanje DI kontejnera, odnosno registrovanje tipova možemo definisati u baznoj test klasi. Unutar bazne test klase iskoristićemo ClassInitailize metodu za registrovanje tipova:

```csharp
[TestClass]
public class BaseTestClass
{
    private static IUnityContainer container;
    public TestContext TestContext { get; set; }
    public IWebDriver Driver { get; private set; }
 
    [ClassInitialize]
    public static void Initialize(TestContext testContext)
    {
        container = new UnityContainer()
            .RegisterType<IWebDriver, FirefoxDriver>("Default", new InjectionConstructor())
            .RegisterType<IWebDriver, FirefoxDriver>("FireFox", new InjectionConstructor())
            .RegisterType<IWebDriver, ChromeDriver>("Chrome", new InjectionConstructor())
            .RegisterType<IWebDriver, InternetExplorerDriver>("InternetExplorer", new InjectionConstructor());
    }
}
```

Ova konfiguracija obezbeđuje da se pri svakom razrešavanju vraća nova instanca IWebDriver-a, što je upravo ono što nam treba ukoliko želimo da pravimo testove koji su izolovani, odnosno testvi koji mogu paralelno da se izvršavaju.

## Kreiranje instance IWebDriver-a

Samo kreiranje, odnosno razrešavanje je najbolje implementirati na jednom mestu, u okviru TestInitialize metode bazne test klase.Najjednostavniji način da izvršimo razrešavanje tipa bio bi na osnovu vrednosti konfiguracione promenljive:

```csharp
[TestInitialize]
public virtual void TestInitialize()
{
    Driver = container.Resolve<IWebDriver>(Configuration.Browser);
}
```

U ovom primeru, Browser predstavlja string ("FireFox", "Chrome", "InternetExplorer" itd.) na osnovu koga će se izvršiti razrešavanje tipa. Drugi, možda zanimljiviji način, bio bi da koristimo deo imena testa za razrešavanje tipa:

```csharp
[TestInitialize]
public virtual void TestInitialize()
{
    var registration = container.Registrations.Skip(1).FirstOrDefault(r => TestContext.TestName.Contains(r.Name));
    Driver = container.Resolve<IWebDriver>(registration != null ? registration.Name : "Default");
}
```

Evo i par testova da proverimo da li razrešavanje radi kako treba:

```csharp
[TestMethod]
public void Test_with_Default()
{
    Assert.IsInstanceOfType(Driver, typeof(FirefoxDriver));
}
 
[TestMethod]
public void Test_with_FireFox()
{
    Assert.IsInstanceOfType(Driver, typeof(FirefoxDriver));
}
 
[TestMethod]
public void Test_with_Chrome()
{
    Assert.IsInstanceOfType(Driver, typeof(ChromeDriver));
}
 
[TestMethod]
public void Test_with_InternetExplorer()
{
    Assert.IsInstanceOfType(Driver, typeof(InternetExplorerDriver));
}
```

Cilj ovog posta bio je da prikaže par zanimljivih načina za instanciranje IWebDriver-a. Svakako da ima još načina, i vrlo rado bih čuo koji je vaš omiljeni način za rešavanje ovog problema.

Umesto implementacije neke factory klase, iskoristićemo IoC kontejner za kreiranje instance konkretnog drajvera. Za primer uzećemo Unity framework.
