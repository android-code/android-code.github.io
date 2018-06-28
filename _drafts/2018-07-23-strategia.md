---
layout: post
title: "Strategia"
date:  2018-07-23
categories: ["Wzorce projektowe"]
image: strategy
github: strategy
description: "Wzorce projektowe / behawioralny"
keywords: "strategia, strategy, wzorzec, wzorce projektowe, wzorzec behawioralny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Strategia` (ang. `Strategy`) (wzorzec behawioralny) definiuje rodzinę wymiennych algorytmów (w postaci klas), które służą do rozwiązywania tego samego problemu na kilka różnych sposobów. Klient może dynamicznie wybrać strategie dla obiektu i dowolnym momencie zmienić ją na inną. Szczegóły implementacji konkretnych strategii są ukryte dla klienta, a ewentualna modyfikacja wiążę się zmianą kodu w klasie konkretnej strategii. Ponadto zwiększa możliwość niezależnego testowania strategii i klienta. W celu usprawnienia procesu wyboru strategii wzorzec ten może być łączony ze wzorcem `Metoda wytwórcza`. Spełnia zasdę `OCP` - otwarte/zamknięte.

## Ograniczenia
Ze względu na podobieństwa w strukturze, wzorzec ten może być mylony ze wzorcami `Adapter` i `Most`, których zastosowanie wynika z realizacji innych celów. Należy zatem się upewnić, że wybór wzorca `Strategia` jest zasadny. W sytuacji, gdy obiekt kontekstu nie wymaga przekazania strategii w konstruktorze, należy zadbać o jej domyślną implementacje. Dodatkowo wzrasta koszt komunikacji między obiektami, a w przypadku wielu implementacji algorytmu powstaje wiele klas.

## Użycie
Wzorzec ten podobnie jak `Adapter` wykorzystuje mechanizm delegowania operacji do innego obiektu. Ponadto używa podobnej struktury jak `Most`, jednakże celem wzorca `Most` jest zaprojektowanie odpowiedniej struktury klas projektu, natomiast `Strategia` skupia się zmienności zachowania obiektu. `Strategia` ma zastosowanie w sytuacjach, gdzie dane zadanie może zostać zrealizowane na wiele różnych sposobów, a decyzja o wyborze implementacji zapada dynamicznie.

## Implementacja
Klasa kontekstu `Context` zawiera referencję do obiektu klasy strategii `AbstractStrategy`. Obiekt strategii może zostać wstrzyknięty przez konstruktor bądź metodę dostępową. Metoda klasy kontekstu wykorzystuje obiekt strategii w celu finalizacji operacji. Klasy strategii `ConcreteStrategy1`, `ConcreteStrategy2` itd. implementują metody interfejsu `AbstractStrategy`.

![Strategia diagram](/assets/img/diagrams/strategy.svg){: .center-image }

Poniższy listing przedstawia implementację wzorca `Strategia` wykorzystywaną w obiektach klasy `Context` w oparciu o dwa warianty `ConcreteStrategy1` oraz `ConcreteStrategy2`.

{% highlight java %}
public class Context {

  private AbstractStrategy strategy;

  public Context(AbstractStrategy strategy) {
    this.strategy = strategy;
  }

  public void setStrategy(AbstractStrategy strategy) {
    this.strategy = strategy;
  }

  public void run(Object args) {
    strategy.action(args);
  }
}

interface AbstractStrategy {

  void action(Object args);
}

public class ConcreteStrategy1 implements AbstractStrategy {

  @Override
  public void action(Object args) {
    //do action specific for ConcreteStrategy1
  }
}

public class ConcreteStrategy2 implements AbstractStrategy {

  @Override
  public void action(Object args) {
    //do action specific for ConcreteStrategy2
  }
}
{% endhighlight %}

Klient przed wywołaniem metody docelowej dokonuje wyboru wstrzykniętej strategii w zależności od stanu i spełnionych warunków.

{% highlight java %}
Context context = new Context(new ConcreteStrategy1());
String arg = "args passed to strategy";
context.run(arg);

//conditions have changed
context.setStrategy(new ConcreteStrategy2());
arg = "new args for strategy";
context.run(arg);
{% endhighlight %}

## Przykład
//TODO

## Biblioteki
Przykładem biblioteki realizującej implementacje wzorca `Strategia` może być metoda `sort` z pakietu `Collections` do której wstrzykiwany jest `Comparator`. Wybór odpowiednich zasobów aplikacji (język, widoki, wielkości itp) w `Androidzie` w swojej idei przypomina wzorzec `Strategia`.
