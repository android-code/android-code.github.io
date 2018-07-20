---
layout: post
title: "Fabryka abstrakcyjna"
date:  2018-07-09
categories: ["Wzorce projektowe"]
image: abstract_factory
github: abstract-factory
description: "Wzorce projektowe / kreacyjny"
keywords: "fabryka abstrakcyjna, abstract factory, fabryka, factory, wzorzec, wzorce projektowe, wzorzec kreacyjny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Fabryka abstrakcyjna` (ang. `Abstract factory`) (wzorzec kreacyjny) dostarcza interfejs do tworzenia powiązanych bądź zależnych obiektów różnych typów bez dokładnej znajomości ich klas. Fabryka abstrakcyjna definiuje rodzinę produktów możliwych do wygenerowania z danej fabryki. Konkretne fabryki mają za zadanie zdecydować o typie i stworzyć obiekty z jednej rodziny produktów. Implementacja wzorca `Fabryki abstrakcyjnej` może być utożsamiana ze zbiorem `Metod wytwórczych` dla każdej abstrakcji produktu. Jednakże w przeciwieństwie do `Metody wytwórczej`, której celem jest tworzenie obiektów o jednym wspólnym typie, `Fabryka abstrakcyjna` realizuje zadanie tworzenia rodziny powiązanych ze sobą obiektów. Proces inicjalizacji obiektów jest centralizowany, kod otwarty na zmiany, a klient odizolowany od szczegółów implementacji rodziny produktów. Klient nie musi znać typów tworzonych produktów, ponieważ cała odpowiedzialność spoczywa na fabryce. Spełnia zasadę `OCP` - otwarte/zamknięte, a także `DIP` - odwrócenia zależności.

## Ograniczenia
Wprowadzenie nowego abstrakcyjnego produktu do rodziny produktów wymusza rozbudowę wszystkich fabryk o metodę zwracającą kolejną abstrakcje, natomiast dodanie nowego typu pociąga za sobą modyfikację metod w niektórych fabrykach.

## Użycie
`Fabryka abstrakcyjna` znajduje zastosowanie tam, gdzie zachodzi potrzeba tworzenia zbioru powiązanych ze sobą produktów o wspólnych typach, ale różnej implementacji, a klient ma być zwolniony ze znajomości szczegółów implementacji. Ponadto dokładne typy obiektów nie muszą być znane w trakcie tworzenia kodu, a decyzja o tym jakie obiekty zostaną wygenerowane zapada dynamicznie.

## Implementacja
Konkretna fabryka implementuje abstrakcyjną fabrykę `AbstractFactory`, która deklaruje dostarczenie grupy produktów ze sobą powiązanych. Każda fabryka odpowiada za stworzenie produktów na swój ustalony sposób. Produkty rozszerzają klasę abstrakcyjną produktu `AbstractProduct` lub implementują jej interfejs.

![Fabryka abstrakcyjna diagram](/assets/img/diagrams/abstract_factory.svg){: .center-image }

Poniższy listing przedstawia implementację fabryki abstrakcyjnej `AbstractFactory` w oparciu o dwie fabrykie `FactoryA`, `FactoryB` i dwie rodziny produktów `AbstractProduct1`, `AbstractProduct2`.

{% highlight java %}
public class FactoryA extend AbstractFactory {

    @Override
    public AbstractProduct1 createProduct1(Product1Type type) {
        //set args to constructor based on type
        if(type == Product1Type.BIG)
            return new Product1A("BIG");
        else
            return new Product1A();
    }

    @Override
    public AbstractProduct2 createProduct2(Product2Type type) {
        //set args to constructor based on type
        if(type == Product2Type.CIRCLE)
            return new Product2A("CIRCLE");
        else
            return new Product2A();
    }
}

public class FactoryB extend AbstractFactory {

    @Override
    public AbstractProduct1 createProduct1(Product1Type type) {
        //set args to constructor based on type
        if(type == Product1Type.SMALL)
            return new Product1B("SMALL");
        else
            return new Product1B();
    }

    @Override
    public AbstractProduct2 createProduct2(Product2Type type) {
        //ignore args for this type
        return new Product2B();
    }
}

public abstract class AbstractFactory {

    public abstract AbstractProduct1 createProduct1(Product1Type type);
    public abstract AbstractProduct2 createProduct2(Product2Type type);
}

public abstract class AbstractProduct1 {

    //some fields and methods
}

public abstract class AbstractProduct2 {

    //some fields and methods
}
{% endhighlight %}

Klient decyduje z której fabryki skorzystać w celu wygenerowania obiektów. Obiekty można generować bezpośrednio z fabryki lub przekazać do obiektu docelowego fabrykę jako parametr konstruktora.

{% highlight java %}
//check conditions and choose factory
AbstractFactory factory = new FactoryA();
CompleteProduct product = new CompleteProduct(factory);

//where CompleteProduct looks like below
public class CompleteProduct {

    private AbstractProduct1 product1;
    private AbstractProduct2 product2;

    public CompleteProduct(AbstractFactory factory) {
        product1 = factory.createProduct1();
        product2 = factory.createProduct2();
    }

    //other methods
}
{% endhighlight %}

## Przykład
Sklep internetowy `JerseySport` prowadzi sprzedaż zestawów odzieżowych `Jersey` (skarpetki `Socks`, spodenki `Shorts`, koszulka `Shirt`) dla różnych dyscyplin sportu. Klient może skonfigurować zestawy pod względem rozmiaru, koloru oraz dla niektórych dyscyplin także długości rękawków koszulek. Ze względu na mnogość konfiguracji parametrów odzieży oraz ich przeznaczenia, sposób składania zamówienia realizowany jest poprzez wzorzec `Fabryki abstrakcyjnej`. Klasa reprezentująca kompletny sportowy zestaw odzieży wraz ze strukturą poszczególnych elementów zestawu przedstawiona jest na poniższym listingu.

{% highlight java %}
public class Jersey {

    private Socks socks;
    private Shorts shorts;
    private Shirt shirt;

    public Jersey(Socks socks, Shorts shorts, Shirt shirt) {
        this.socks = socks;
        this.shorts = shorts;
        this.shirt = shirt;
    }

    public int getPrice() {
        return socks.getPrice() + shorts.getPrice() + shirt.getPrice();
    }

    public String getDescription() {
        return "Jersey: " + socks.getName() + " " + shorts.getName() + " " + shirt.getName();
    }

    public boolean isAvailable() {
        return socks.isAvailable() && shorts.isAvailable() && shirt.isAvailable();
    }
}

public abstract class Socks extend Clothes {

    private Color color;

    public Socks(Color color) {
        this.color = color;
    }
}

public abstract class Shorts extend Clothes {

    private Color color;
    private Size size;

    public Shorts(Color color, Size size) {
        this.color = color;
        this.size = size;
    }
}

public abstract class Shirt extend Clothes {

    private Color color;
    private Size size;
    private boolean longSleeve;

    public Shirt(Color color, Size size, boolean longSleeve) {
        this.color = color;
        this.size = size;
        this.longSleeve = longSleeve;
    }
}

public abstract Clothes {

    public abstract int getPrice();
    public abstract String getName();
    public abstract boolean isAvailable();
}
{% endhighlight java %}

Elementy zestawu odzieżowego `Socks`, `Shorts`, `Shirt` w zależności od sportu różnią się ceną, właściwościami oraz sposobem sprawdzania dostępności. W związku z tym dla każdej dyscypliny sportowej tworzona jest dedykowana konfiguracja elementów zestawu odzieżowego. Poniższy listing przedstawia implementacje dla piłkarskiej odzieży.

{% highlight java %}
public FootballSocks extend Socks {

    @Override
    public int getPrice() {
        return 10;
    }

    @Override
    public String getName() {
        return "Footbal socks " + color.toString();
    }

    @Override
    public boolean isAvailable() {
        //connect with server database and providers in specific way and check availability
        return true; //mock
    }
}

public FootballShorts extend Shorts {

    @Override
    public int getPrice() {
        return 20;
    }

    @Override
    public String getName() {
        return "Footbal shorts " + color.toString() + " " + size.toString();
    }

    @Override
    public boolean isAvailable() {
        //connect with server database and providers in specific way and check availability
        return true; //mock
    }
}

public FootballShirt extend Shirt {

    @Override
    public int getPrice() {
        return 40;
    }

    @Override
    public String getName() {
        if(longSleeve)
            return "Footbal shirt long sleeve " + color.toString() + " " + size.toString();
        else
            return "Footbal shirt " + color.toString() + " " + size.toString();
    }

    @Override
    public boolean isAvailable() {
        //connect with server database and providers in specific way and check availability
        return true; //mock
    }
}

//implement BasketballSocks, BasketballShorts, BasketballShirt and others sport jersey in similar way
{% endhighlight java %}

Do składania kompletnego zestawu służą dedykowane dla każdej dyscypliny sportu fabryki (`FootballFactory`, `BasketballFactory` itp) implementujące abstrakcyjnę fabrykę `JerseyFactory`.

{% highlight java %}
public class FootballFactory extend JerseyFactory {

    @Override
    public Socks createSocks(Color color) {
        return new FootballSocks(color);
    }

    @Override
    public Shorts createShorts(Size size, Color color) {
        return new FootballShorts(size, color);
    }

    @Override
    public Shirt createShirt(Size size, Color color, boolean longSleeve) {
        return new FootballShirt(size, color, longSleeve);
    }
}

public class BasketballFactory extend JerseyFactory {

    @Override
    public Socks createSocks(Color color) {
        return new BasketballSocks(color);
    }

    @Override
    public Shorts createShorts(Size size, Color color) {
        return new BasketballShorts(size, color);
    }

    @Override
    public Shirt createShirt(Size size, Color color, boolean longSleeve) {
        return new BasketballShirt(size, color, false);
    }
}

public abstract class JerseyFactory {

    public abstract Socks createSocks(Color color);
    public abstract Shorts createShorts(Size size, Color color);
    public abstract Shirt createShirt(Size size, Color color, boolean longSleeve);

    public Jersey createJersey(Size size, Color color, boolean longSleeve) {
        Socks socks = createSocks(color);
        Shorts shorts = createShorts(size, color);
        Shirt shirt = createShirt(size, color, longSleeve);
        return new Jersey(socks, shorts, shirt);
    }
}
{% endhighlight java %}

Proces kompletowania zamówienia przez klienta może przebiegać następująco.

{% highlight java %}
List<Jersey> orders = new ArrayList();

//order football jersey
JerseyFactory factory = new FootballFactory();
orders.add(factory.createJersey(Size.M, Color.RED);
orders.add(factory.createJersey(Size.L, Color.RED);
orders.add(factory.createJersey(Size.XL, Color.RED);

//order basketball jerseys
factory = new BasketballFactory();
orders.add(factory.createJersey(Size.XL, Color.BLUE);
orders.add(factory.createJersey(Size.S, Color.BLUE);
orders.add(factory.createJersey(Size.L, Color.BLUE);

//order custom basketball jersey
Socks socks = factory.createSocks(Color.WHITE);
Shorts shorts = factory.createShorts(Color.BLACK, Size.M);
Shirt shirt = factory.createShirt(Color.RED, Size.M, true);
orders.add(new Jersey(socks, shorts, shirt));

//check availability, price and get order summary before confirm order
int totalPrice = 0;
String summary;
for(Jersey jersey : orders) {
    totalPrice = totalPrice + jersey.getPrice();
    summary = summary + jersey.getDescription() + "\n";
    if(!jersey.isAvailable()) {
        //show warning about longer delivered time
    }
}
//show total price and summary
{% endhighlight java %}

## Biblioteki
Ze względu na specyfikację wzorca, implementacja `Fabryka abstrakcyjna` spoczywa na programiście.
