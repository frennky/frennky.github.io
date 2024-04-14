---
title:  "Data Driven testing - XML, SQLCE"
date:   2011-04-17 22:24:30 +0100
---

Vrlo često se prilikom testiranja javlja potreba za izvršavanjem istog testa sa različitim ulaznim podacima.  Visual Studio Unit Testing Framework ima ugrađenu podršku za Data Driven Testing (DDT).

Data Driven testovi se razlikuju od ostalih testova po tome što su dekorisani sa DataSource atributom. Pomoću ovog atributa definiše se izvor ulaznih podataka. Izvor podataka može da bude CSV fajl, XML, SQL Server, SQLCE baza i slično. Kada se pokrene test, podaci se čitaju red po red, i test se izvršava za svaki red posebno, a tekućem redu se pristupa preko TestContext-a.

```csharp
[TestMethod]
[DataSource("System.Data.SqlClient", "Data Source=.;Initial Catalog=TestData;Integrated Security=True; MultipleActiveResultSets=True", "Table1", DataAccessMethod.Random)]
public void DDT_with_SQL()
{
   //
   // TODO: Add test logichere
   //
}
 
[TestMethod]
[DataSource("System.Data.SqlClient", "Data Source=.;Initial Catalog=TestData;Integrated Security=True;MultipleActiveResultSets=True", "MyView", DataAccessMethod.Sequential)]
public void DDT_with_SQL2()
{
   //
   // TODO: Add test logichere
   //
}
```

Ukoliko koristimo SQL Server kao izvor podataka, treba imati na umu da pri izvršavanju testova može da dođe do gubitka konekcije, što će izazvati pad testova. Takođe, može se desiti i da SQL Server na kome se nalaze ulazni podaci jednostavno nije dostupan računaru na kome želimo da izvršimo testove (recimo ako želimo da izvršimo testove na računaru klijenta).

Pored toga, ponekad je potrebno promeniti ulazne podatke, te ukoliko baza ne poseduje neku vrstu history mehanizma ili backup, može se desiti da izgubimo vezu između prethodnih rezultata testova i ulaznih podataka.

```csharp
[TestMethod]
[DeploymentItem(@"TestProject1\XMLFile1.xml")]
[DataSource("Microsoft.VisualStudio.TestTools.DataSource.XML", "XMLFile1.xml", "DDT_with_XML", DataAccessMethod.Sequential)]
public void DDT_with_XML()
{
   //
   // TODO: Add test logic here
   //
}
 
[TestMethod]
[DeploymentItem(@"TestProject1\TestDatabase.sdf")]
[DataSource("System.Data.SqlServerCe.3.5", "data source=|DataDirectory|\\TestDatabase.sdf;password=1234", "Table1", DataAccessMethod.Sequential)]
public void DDT_with_SQLCE()
{
   //
   // TODO: Add test logic here
   //
}
```

Zbog toga je bolje koristiti XML ili SQLCE bazu kao izvor podataka kako bismo lakše mogli da vršimo deployment ulaznih podataka, kao i da ih sačuvamo zajedno sa rezultatima.

Struktura XML fajla koji predstavlja DataSource testa:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<testsuite>
   <DDT_with_XML>
      <Year>2011</Year>
      <Month>4</Month>
      <Day>22</Day>
      <Result>true</Result>
   </DDT_with_XML>
   <DDT_with_XML>
      <Year>2011</Year>
      <Month>12</Month>
      <Day>20</Day>
      <Result>false</Result>
   </DDT_with_XML>
</testsuite>
```
