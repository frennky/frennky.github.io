---
title:  "Skrivanje interfejsa, klasa i metoda"
date:   2012-02-27 22:27:30 +0100
---

Sigurno ste nekom prilikom napravili biblioteku na koju ste bili ponosni, a onda totalno razočarani pretrpanim IntelliSense-om kada pokušate da je koristite. Veliki broj interfejsa, klasa ili metoda čine vašu biblioteku teškom za korišćenje. Pravilnom upotrebom modifikatora pristupa ovaj problem se može samo delimično sanirati, jer neki tipovi jednostavno moraju biti javni. Ovaj problem je naročito očigledan ako ste pravili biblioteku koja izlaže fluent API.

Na svu sreću postoji jedan atribut u okviru `System.ComponentModel` namespace-a koji nam može u ovome pomoći.

## EditorBrowsableAttribute klasa

Ovim atributom mogu se dekorisati gotovo svi elementi aplikacije, osim asemblija, modula, parametra, genričkog parametra i povratne vrednosti. A pomoću State property-ja definiše se vidiljivost elementa koji je dekorisan atributom. Skrivanje se vrši samo na nivou IntelliSense-a, odnosno, svi skriveni elementi su i dalje dostupni ali se ne vide u IntelliSense-u.

## Primer

S obzirom da sve korisničke klase imaju kao bazni `Object` klasu, kada kreiramo korisničku klasu, bez ijedne metode, imaćemo pristup dobro poznatim metodama `Equals`, `GetHashCode`, `GetType` i `ToString`. Ukoliko ove metode namaju smisla u kontekstu biblioteke koju pravimo, možemo ih sakriti i na taj način smanjiti listu metoda u IntelliSense-u.

```csharp
[EditorBrowsable(EditorBrowsableState.Never)]
public interface IHidden
{
    [EditorBrowsable(EditorBrowsableState.Never)]
    bool Equals(object obj);
 
    [EditorBrowsable(EditorBrowsableState.Never)]
    int GetHashCode();
 
    [EditorBrowsable(EditorBrowsableState.Never)]
    Type GetType();
 
    [EditorBrowsable(EditorBrowsableState.Never)]
    string ToString();
}
 
public class MyClass : IHidden
{
    public void DoSomething()
    {
        // some implementation
    }
 
    public void DoSomethingElse()
    {
        // some other implementation
    }
}
```

Atribut nad samim interfejsom obezbeđuje da sam interfejs ne bude izlistan kada listamo namespace u kome se nalazi, dok atribut nad metodama `Equals`, `GetHashCode`, `GetType` i `ToString` skriva ove metode kada pristupamo članicama MyClass kalse.
