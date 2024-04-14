---
title:  "Anatomija test klase"
date:   2011-04-25 22:20:30 +0100
---

Sve test klase moraju biti dekorisane sa atributom [TestClass]. Na ovaj nacin Visual Studio prepoznaje da ta klasa sadrži testove. Pored ovog atributa, test klasa mora da bude javna i mora da ima bezparametarski javni konstruktor.

```csharp
using System;
using System.Text;
using System.Collections.Generic;
using Microsoft.VisualStudio.TestTools.UnitTesting;
namespace TestProject1
{
    [TestClass]
    public class UnitTest1
    {
        [TestMethod]
        public void TestMethod1()
        {
        }
    }
}
```

Unutar test klase, svaka test metoda mora da bude javna, bezparametarska i da ne vraca vrednost. Da bi se smatrala test metodom, mora da bude dekorisana sa [TestMethod] atributom. Ukoliko neki od atributa nedostaje, nece biti prijavljene nikakve greške, vec  jednostavno nece biti testova za izvršavanje. Medutim, ukoliko postoje atributi, ali nisu ispunjeni neki drugi zahtevi bice prijavljene greške.

## Pripremne i završne akcije

U odredenim situacijama potrebno je izvšiti neke pripremne i završne akcije kako bi testovi mogli da se izvrše. Na primer, ukoliko deo koda koji testiramo zavisi od nekog izvora podataka, potrebno je pripremiti podatke bilo   putem lažnog (fake) izvora podataka ili iz pravog izvora podataka. Ovaj pripremni skup akcija možemo izdvojiti u zasebnu metodu ukoliko ih je potrebno izvršiti pre svakog testa.

Za tu svrhu postoji nekoliko atributa kojima se mogu dekorisati odgovarajuce metode. Tu spadaju:

- AssemblyInitialize
- AssemblyCleanup
- ClassInitialize
- ClassCleanup
- TestInitialize
- TestCleanup

Metode sa [AssemblyInitialize]/[AssemblyCleanup] atributom se pozivaju jednom pre nego što se pozove ijedan test u celoj test biblioteci, dok se cleanup poziva nakon svih testova. Može da postoji samo po jedna ovakva metoda u celoj test biblioteci.  Primer bi izgledao ovako:

```csharp
[AssemblyInitialize]
public static void AssemblyInit(TestContext context)
{
}
```

Metode dekorisane sa atributima [ClassInitlialize]  se pokrecu pre prvog testa unutar neke test klase, dok se metode dekorisane sa [ClassCleanup] pozivaju nakon poslednjeg testa unutar klase:

```csharp
[ClassInitialize]
public static void ClassInit(TestContext context)
{
    //...
}
 
[ClassCleanup]
public static void Cleanup()
{
    //...
}
```

Ove metode inicijalizacije i cleanup se pozivaju pre i posle svake test metode. Pored toga, ovo su metode instance a ne staticke metode:

```csharp
[TestInitialize]
public void TestInit()
{
}
 
[TestCleanup]
public void TestCleanup()
{
}
```

## Prolaznost testova

Sam test može imati neko od nekoliko mogucih rezultata, odnosno statusa. Pored dva najbitnija, prošao (passed) i pao (failed), test može imati i neodreden ishod (inconclusive) u slucaju da nismo u mogucnosti da procenimo da li je testirana funkcionalnost ispunjena. Pored ovih ishoda, testovi mogu imati i status obustavljen (aborted), neizvršen (not executed) i neizvršen do kraja usled isteka ocekivanog vremena trajanja (time out).

Da bi test prošao test metoda jednostavno ne sme da ispusti izuzetak. Ne postoji eksplicitni nacin da se kaže da je test prošao, za razliku od ishoda pao (failed) i neodredenog ishoda (inconclusive).

Za evaluaciju rezultata testova koristi se Assert klasa. Ona sadrži veliki broj metoda za proveru validnosti testa. Pomocu ovih metoda možemo proveriti da li kod koji testiramo generiše vrednost koju ocekujemo.

```csharp
public class Calculate
{
    public static int Add(int x, int y)
    {
        return (x + y);
    }
}
 
[TestClass]
public class UnitTest1
{
    private int x;
    private int y;
 
    [TestInitialize]
    public void Initialize()
    {
        var random = new Random(DateTime.Now.Millisecond);
        x = random.Next();
        y = random.Next();
    }
 
    [TestCleanup]
    public void Cleanup()
    {
    }
 
    [TestMethod]
    public void Add_Test()
    {
        var result = Calculate.Add(x, y);
        var expected = x + y;
        Assert.AreEqual(expected, result);
    }
 
    [TestMethod]
    public void Substract_Test()
    {
        Assert.Inconclusive("Method not implemented.");
    }
}
```

Pored pomenutih atributa i metoda, Visual Studio Unit Testing framework sadrži još mnogo toga, kao na primer, podršku za DDT (Data Driven Testing), više o tome možete procitati u postu [Data Driven Testing - XML, SQLCE](http://ivanfranjic.net/2011/4/data-driven-testing-xml-sqlce).
