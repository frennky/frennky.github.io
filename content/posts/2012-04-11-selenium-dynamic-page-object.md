---
title:  "Selenium Dynamic Page Object"
date:   2012-04-11 21:54:30 +0100
---

O Page Object klasama sam pisao ranije u postu [Page Object Pattern](http://ivanfranjic.net/2011/11/page-object-pattern), ovaj post ima za cilj da prikaže kako se može napraviti dinamička Page Object klasa. Zahvaljujući novinama koje je .Net 4.0 doneo sa sobom, pre svega dinamički tip, moguće je jednostavan način implementirati dinamički Page Object.

## Dynamic Page Object

Sve što treba jeste da naša Page Object klasa nasledi `DynamicObject` i da napišemo override metode TryGetMember:

```csharp
public class DynamicPageObject : DynamicObject
{
    private readonly IWebDriver driver;
 
    public DynamicPageObject(IWebDriver driver)
    {
        this.driver = driver;
    }
 
    public override bool TryGetMember(GetMemberBinder binder, out object result)
    {
        try
        {
            result = driver.FindElement(By.Id(binder.Name));
            return true;
        }
        catch (NoSuchElementException)
        {
            result = null;
            return false;
        }
    }
}
```

Ograničenje ove Page Object klase je to što se za nalaženje elemenata koristi vrednost id atributa. Kako se u praksi najčešće koristi id atribut za nalaženje elemenata, ovo ograničenje retko kad predstavlja problem. Ovakvom implementacijom gubimo IntelliSense podršku, ali zato možemo jednu instancu Page Object klase iskorititi za sve stranice u aplikaciji.

Ukoliko nam je cilj da brzo napišemo Selenium testove, ova klasa može dosta u tome da pomogne. Međutim, ako planiramo veliki broj testova, investiranje u zasebne Page Object klase više se isplati sa aspekta održavanja testova. S druge strane, prelaz sa upotrebe Dynamic Page Object klase na zasebne Page Object klase je relativno jednostavan jer ne zavisi od logike testa.

## Upotreba

```csharp
public class Program
{
    static void Main(string[] args)
    {
        var driver = new FirefoxDriver();
 
        driver.Navigate().GoToUrl(@"http://www.google.com/");
 
        dynamic page = driver.AsDynamicPage();
 
        page.gbqfq.SendKeys("Ivan Franjic .Net");
 
        driver.Quit();
    }
}
```
