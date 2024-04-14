---
title:  "Page Object Pattern"
date:   2011-11-30 23:03:30 +0100
---

Od testova se očekuje da se brzo prilagode promenama korisničkog interfejsa, koje su nažalost vrlo česte. Pored toga, od testova se očekuje da rade sa više različitih korisničkih interfejsa za različite uređaje, počev od web stranica za desktop browser-e pa do mobilnih verzija stranica za mobile browser-e.

Za ove probleme postoji univerzalno rešenje u vidu Page Object paterna koji se može primeniti kako na web tako i na desktop aplikacije, odnosno na bilo koju vrstu GUI aplikacija.

## Šta je Page Object patern?

Suština Page Object paterna jeste u mapiranju UI strane na Page klasu, gde strana može biti HTML strana, WPF forma i sl. Korisničke akcije nad UI stranom se izlažu kroz metode, a sami UI elementi kroz propertije Page klase. Na taj način dobija se jedan sloj između samog testa i aplikacije koja se testira, a posledica svega toga je lakše kreiranje novih testova i održavanje test koda kada dođe do promena na aplikaciji.

Kada govorimo o automatskom testiranju, stranica ne predstavlja ništa više do strane koju korisnik vidi. Svaka stranica se sastoji od proizvoljnog broja različitih komponenti, od kojih neki samo prikazuju podatke (npr. labele), dok drugi obezbeđuju interakciju (npr. dugmići). Stranice, međutim, mogu imati neki zajednički sadržaj kao što su zaglavlja, polja za pretragu ili menije.

Upotreba Page klasa bi trebalo da obezbedi lakšu izmenu testova ukoliko dođe do promene aplikacije koja se testira, jer je potrebno napraviti izmenu samo na jednom mestu. Pored toga, kreiranje i održavanje testova je znatno lakše, jer sadrže samo logiku koja je potrebna za test.

Ovaj patern nije vezan isključivo za web aplikacije, ali ću iskoristiti priliku da ga objasnim i prikažem kroz primer koji se oslanja na Selenium WebDriver.

## Selenium WebDriver PageFactory

Implementacija ovog paterna podrazumeva kreiranje klase, sa odgovarajućim metodama, za svaku stranicu ili deo stranice (zaglavlje, meni itd.) aplikacije koja se testira. Metode ovih klasa treba da odgovaraju akcijama koje se izvršavaju nad stranicama. Takođe, klase treba da sadrže propertije za UI elemente sa kojima korisnik ima interakciju.

Za podršku Page Object paterna u okviru WebDriver Support biblioteke postoji klasa [PageFactory](http://code.google.com/p/selenium/wiki/PageFactory). Jednostavan primer, sličan primeru iz prethodnog posta, bi izgledao ovako:

```csharp
using OpenQA.Selenium;
using OpenQA.Selenium.Support.PageObjects;
 
namespace Tests.Pages
{
    public class GoogleHomePage
    {
        private string url = "http://www.google.com/";
        private IWebDriver driver;
 
        [FindsBy(How = How.Name, Using = "q")]
        public IWebElement QueryTextBox { get; set; }
 
        [FindsBy(How = How.Name, Using = "btnK")]
        public IWebElement SearchButton { get; set; }
 
        public GoogleHomePage(IWebDriver webDriver)
        {
            driver = webDriver;
            PageFactory.InitElements(driver, this);
        }
 
        public void Navigate()
        {
            driver.Navigate().GoToUrl(url);
        }
    }
}
 
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Tests.Pages;
using OpenQA.Selenium.Firefox;
 
namespace Tests
{
    [TestClass]
    public class GoogleTests
    {
        [TestMethod]
        public void Test1()
        {
            var driver = new FirefoxDriver();
 
            var googleHomePage = new GoogleHomePage(driver);
 
            googleHomePage.QueryTextBox.SendKeys("Ivan Franjic .Net");
 
            googleHomePage.SearchButton.Click();
 
            Assert.IsTrue(driver.Title.Contains("Ivan Franjic .Net"));
 
            driver.Quit();
        }
    }
}
```

Treba napomenuti da se WebElement-i instanciraju po pozivu. Odnosno, ukoliko se nekom WebElement-u Page Object klase nikad ne pristupi, nikad se neće pozvati FindElement za taj WebElement.

S obzirom da stranica enkapsulira specifičnosti test framework-a (u ovom slučaju Selenium WebDriver), ako promenimo ovaj test framework potrebno je samo implementirati Page klase dok logika testa ostaje netaknuta. Dakle, dolazimo do toga da se izlažu stvari koje mogu da se vide i urade na stranici, a skrivaju detalji kako se to zapravo izvodi. Takođe, sa Page klasama je jasno gde se može naći postojeći i gde staviti novi kod.

Na kraju, može se izvesti zaključak da ovakav način pisanja test koda promoviše upotrebljivost i smanjuje dupliranje koda čineći testove čitljivijim, što naravno poboljšava održavanje naročito u situacijama kada se aplikacija brzo razvija.
