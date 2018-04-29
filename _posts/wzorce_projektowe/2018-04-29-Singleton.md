---
layout: post
title: "Singleton"
date:  2018-04-29
categories: ["Wzorce projektowe"]
image: singleton
---

## Zastosowanie
`Singleton` (wzorzec konstrukcyjny) ma za zdanie przede wszystkim ograniczenie tworzenia obiektów danej klasy do jednej instancji oraz zapewnieniu do niego globalnego dostępu. Swoje pochodzenie zawdzięcza `C++`, gdzie pełnił rolę następnika zmiennej globalnej używanej przez preprocesor. Jest wzorcem, który wzbudza wiele kontrowersji - z pozoru prosty, łatwy w implementacji, jednakże źle wykorzystany stanowi popularny antywzorzec. Dzieje się tak, gdy jego rola jest sprowadzona do zmiennej globalnej bez uwzględnienia kontekstu wykorzystania. Singleton można zastosować tam, gdzie rzeczywiście istnieje pewność występienia tylko jednej współdzielonej instancji klasy.

## Ograniczenia
`Singleton` tworzony jest w oparciu o prywatny konstruktor, który uniemożliwia rozszerzenie klasy. Poza zarządzaniem swoim cyklem życia pełni rolę logiki biznesowej czym łamie zasadę `SRP` - pojedynczej odpowiedzialności. Co więcej łamie zasadę `OCP` - otwarte/zamknięte, która mówi, że klasy powinnyć być otwarte na rozszerzenie lecz zamknięte na modyfikacje. Singleton utrudnia testowanie ponieważ przed wywołaniem testów należy pamiętać, aby był on właściwie zainicjonowany. Ze względu na architekturę systemu `Android`, `Singleton` nie jest zalecanym wzorcem. `Singleton Mutable` może utracić swój stan, gdy system potrzebuje zwolnić zasoby pamięci. Natomiast `Singleton Immutable` może być często zastąpiony klasami typu `Utility` (z metodami statycznymi).

## Użycie
Wzorzec ten jest wykorzystywany w dostępie do `SharedPreferences`. Popularną praktyką jest również jego implementacja dla obiektów typu `Logger` czy API dostępu do danych np.: `Retrofit`.

## Implementacja
Istnieje wiele implementacji tego wzorca. Jedna z najprostszych opiera się na inicjalizowaniu obiektu poprzez publiczną metodę `getInstance` dopiero w momencie jego pierwszego wywołania. Konstruktor jest niewidoczna spoza klasy. 

![Singleton diagram](/assets/img/diagrams/singleton.svg){: .center-image }

Poniższe rozwiązanie może rodzić problemy w wielowątkowym środowisku.

{% highlight java %}
public class Singleton {
 
    private static Singleton instance;
 
    private Singleton() {
    }
 
    public static Singleton getInstance() {
        if(instance == null){
            instance = new Singleton();
        }
        return instance;
    }
}
{% endhighlight %}


Singleton Holder dobrze sprawdza się w środowisku wielowątkowym, ponieważ nie wymaga synchronizacji oraz zapewnia leniwe tworzenie instancji.

{% highlight java %}
public class SingletonHolder {
 
    private SingletonHolder() {
    }
 
    private static class Holder {
        private static final SingletonHolder INSTANCE = new SingletonHolder();
    }
 
    public static SingletonHolder getInstance() {
        return Holder.INSTANCE;
    }
}
{% endhighlight %}

## Biblioteki
Właściwym wykorzystaniem `Singleton` jest zastosowanie frameworka `Dagger 2`, implementującego wzorzec `Dependency Injection` (Wstrzykiwanie Zależności).