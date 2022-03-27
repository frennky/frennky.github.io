---
layout: post
title:  "WebDriver - ITakeScreenshot"
date:   2011-12-15 22:19:30 +0100
---

Prilikom izvršavanja auto testa često sam bio u situaciji da mi je potreban screenshot aplikacije. Razloga za to može biti više, ali najčešći razlog jeste potreba da se uhvati trenutak u kome je test pao kako bi se lakše utvrdio uzrok.

Pored toga neke stvari je neophodno testirati, ali samo testiranje nije moguće u potpunosti automatizovati. Na primer, potrebno je testirati da li je layout ili uopšte neki vizuelni aspekt aplikacije korektan.

Koji god razlog da je, pomouću Selenium 2 WebDriver-a moguće je napraviti screenshot internet pretraživača na vrlo jednostavan način.

Primer koji sledi prikazuje kako se može uhvatiti screenshot ukoliko test padne:

```csharp
[TestClass]
public class MyTests
{
    public TestContext TestContext { get; set; }
    public IWebDriver Driver { get; private set; }
     
    /// <summary>
    /// Test initialize
    /// </summary>
    /// <remarks>This method is run before each Test</remarks>
    [TestInitialize]
    public void TestInitialize()
    {
        Driver = new FirefoxDriver();
    }
 
    /// <summary>
    /// Test cleanup
    /// </summary>
    /// <remarks>This method is run after each Test</remarks>
    [TestCleanup]
    public void TestCleanup()
    {
        var screenShotDriver = Driver as ITakesScreenshot;
        if (screenShotDriver == null) return;
 
        // Make filename unique in case this is a cleanup for data-driven test ;)
        var filename = string.Format("screenshot-{0}.png", DateTime.Now.Ticks);
        var fullFilename = Path.Combine(TestContext.ResultsDirectory, filename);
         
        var screenShot = screenShotDriver.GetScreenshot();
        screenShot.SaveAsFile(fullFilename, ImageFormat.Png);
        TestContext.AddResultFile(fullFilename);
         
        Driver.Quit();
    }
 
    [TestMethod]
    public void ScreenshotTest()
    {
        Assert.Fail("I just want to take a screenshot!");
    }
}
```
