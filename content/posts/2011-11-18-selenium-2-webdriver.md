---
title:  "Selenium 2 - WebDriver"
date:   2011-11-18 22:39:30 +0100
---

Selenium 2 (WebDriver) pretstavlja jedno od rešenja za automatsko testiranje web aplikacija. Ima jednostavan API koji se brzo i lako uči, svakako lakše nego Selenium-RC (1.0) API, što naravno čini testove lakšim za razumevanje i održavanje. Ne zavisi ni od jednog test framework-a tako da se može se koristiti uz bilo koji unit testing framework (NUnit, xUnit, MSTest itd.).

## Kako to zapravo radi

Kada se jednom doda u test projekat, WebDriver se koristi kao i bilo koja druga biblioteka i za razliku od Selenium RC nisu potrebne nikakve dodatne instalacije ili startovanje dodatnih procesa. WebDriver radi tako što direktno komunicira sa internet pretraživačem koristeći podršku za automatizaciju koja je ugrađena u sam internet pretraživač (da dobro ste pročitali današnji pretraživači imaju ugrađenu podršku za automatizaciju).

Za razliku od WebDriver-a, Selenium-RC se oslanjao na Selenium Server koji je ubacivao javascript u internet pretraživač kako bi ga automatizovao. Dakle, testovi su bili pisani u nekoj klijentskoj Selenium biblioteci, a zatim su se te komande prevodile u javascript. Na svu sreću, od kad su se proizvođači internet pretraživača uključili u celu priču, stvari su postale dosta jednostavnije.

## Kako početi

Preko [NuGet](http://www.nuget.org/) servisa može se brzo i jednostavno skinuti najnovija verzija [WebDriver](http://www.nuget.org/List/Packages/Selenium.WebDriver) biblioteke. Pored osnovne biblioteke preko NuGet servisa može da se skine i Support biblioteka, o kojoj ću nekom drugom prilikom više pisati.

Ovo vam je dovoljno da možete da počnete da pišete auto testove za FireFox i InternetExplorer. Za Chrome vam treba driver koji može [ovde](http://code.google.com/p/chromium/downloads/list) da se skine.

## Primer

U ovom primeru se koristi MSTest kao testni framework, ali naravno može se koristiti i bilo koji drugi. Nadam se da ćete posle ovog primera uvideti kako je jednostavno pisati auto testove pomoću Selenium WebDriver biblioteke.

```csharp
using OpenQA.Selenium.Firefox;
using OpenQA.Selenium;
 
[TestClass]
public class GoogleTest
{
    [TestMethod]
    public void Test1()
    {
        IWebDriver driver = new FirefoxDriver();
 
        driver.Navigate().GoToUrl("http://www.google.com/");
 
        IWebElement query = driver.FindElement(By.Name("q"));
        query.SendKeys("Ivan Franjic .Net");
 
        IWebElement resultLink = driver.FindElement(By.PartialLinkText("ivanfranjic.net"));
        resultLink.Click();
         
        Assert.IsTrue(driver.Title.Contains("Ivan Franjic Blog"), "Ooops, wrong blog opened.");
        driver.Quit();
    }
}
```

Toliko za sad, vidimo se uskoro :)
