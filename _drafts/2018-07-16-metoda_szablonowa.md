---
layout: post
title: "Metoda szablonowa"
date:  2018-07-16
categories: ["Wzorce projektowe"]
image: template_method
github: template-method
description: "Wzorce projektowe / behawioralny"
keywords: "metoda szablonowa, template method, szablon, template, wzorzec, wzorce projektowe, wzorzec behawioralny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Metoda szablonowa` (ang. `Template method`) (wzorzec behawioralny) ma za zadanie zdefiniować szkielet algorytmu, który jest otwarty na wprowadzanie zmian do wybranych jego części. Szablon ten składa się z operacji niezmiennych zdefiniowanych w klasie bazowej oraz z metod zmiennych tzw. `haczyków`, które przyjmują implementacje zależną od klas pochodnych. Dzięki temu zmniejsza się nadmiarowość kodu, a dokonanie w nim zmian staje się mniej kosztowne. Można powiedzieć, że metoda szablonowa dostarcza pewien standard realizacji tego samego zadania na różne sposoby.

## Ograniczenia
Wykorzystując wzorzec `Metoda szablonowa` należy zapewnić odpowiednią kontrolę dostępu do `metod haczyków` poprzez wymuszenie nadpisania (`metoda abstrakcyjna`) lub użycie chronionego modyfikatora dostępu. Metody niezmienne należy ukryć. Ze względu na konieczność przesłonięcia abstrakcyjnych metod haczyków w klasach pochodnych, warto dążyć do ich minimalizacji. Pozostałe haczyki mogą, ale nie muszą posiadać domyślnej implementacji. Decydując się na opcjonalne nadpisanie metod należy mieć pewność, że nie zaburzy to działania metody szablonowej.

## Użycie
`Metoda szablonowa` wykorzystywana jest w celu eliminacji powtórzeń kodu w metodach o tym samym przeznaczeniu, ale o różnej implementacji (w pewnych częściach) zależnej od podklas. Ponadto używa się jej tam, gdzie od metod o różnej implementacji oczekuje się podobnego sposobu działania.

## Implementacja
Klasa abstrakcyjna `AbstractClass` definiuje szkielet metody szablonowej `templateMethod`, dostarcza implementacje dla metod niezmiennych o prywatnym modyfikatorze dostępu oraz wymusza nadpisanie abstrakcyjnej metody zmiennej. Klasy rozszerzające dostarczają implementacji dla wszystkich metod zmiennych.

![Metoda szablonowa diagram](/assets/img/diagrams/template_method.svg){: .center-image }

Poniższy listing przedstawia implementację metody szablonowej w klasie `AbstractClass` oraz rozszerzenie implementacji w klasach pochodnych.

{% highlight java %}
public abstract class AbstractClass {

  public void templateMethod() {
    primitiveOperation1();
    primitiveOperation2();
    primitiveOperation3();
  }

  protected abstract void primitiveOperation2();

  private void primitiveOperation1() {
    //do something always
  }

  protected void primitiveOperation3() {
    //do something optional
  }
}

public class ConcreteClass1 extends AbstractClass {

  @Override
  protected void primitiveOperation2() {
    //do custom work for ConcreteClass1
  }

  @Override
  protected void primitiveOperation3() {
    //do custom work for ConcreteClass2 by overriding default
  }
}

public class ConcreteClass2 extends AbstractClass {

  @Override
  protected void primitiveOperation2() {
    //do custom work for ConcreteClass2
  }
}
{% endhighlight %}

Klient tworzy obiekt żądanego typu i wywołuje na nim metodę szablonową, która wykonuje operacje w sposób dedykowany dla implementacji swojej klasy.

{% highlight java %}
AbstractClass object = new ConcreteClass1();
object.templateMethod();
{% endhighlight %}

## Przykład

{% highlight java %}
//TODO
{% endhighlight %}

## Biblioteki
Wzorzec `Metody szablonowej `wykorzystywany jest m.in. w bibliotekach wspomagających automatyzację testów jednostkowych np.: `jUnit`. Ponadto idea wzorca znajduje zastosowanie tam, gdzie zachodzi pewien stały cykl życia obiektu. Cykl życia Aktywności w Androidzie może być rozpatrywany w kontekście idei `Metody szablonowej
