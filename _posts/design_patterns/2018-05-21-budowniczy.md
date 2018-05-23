---
layout: post
title: "Budowniczy"
date:  2018-05-21
categories: ["Wzorce projektowe"]
image: builder
github: builder
description: "Wzorce projektowe / kreacyjny"
keywords: "budowniczy, builder, fluent builder, wzorzec, wzorce projektowe, wzorzec kreacyjny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Budowniczy` (ang. `Builder`) (wzorzec kreacyjny) rozdziela sposób tworzenia obiektów od ich reprezentacji. Dzieli proces wytwórczy obiektu na etapy. Każdy z etapów może być zaimplementowany na wiele sposobów co umożliwia tworzenie różnych reprezentacji obiektów tej samej klasy. Spełnia zasadę `SRP` - pojedynczej odpowiedzialności, `OCP` - otwarty/zamknięty oraz `DIP` - odwrócenia zależności, dzięki czemu zmiana sposobu tworzenia obiektów jest elastyczna i odbywa się niskim kosztem. Ponadto kod jest łatwiejszy w testowaniu, utrzymaniu, zapewnieniu obsługi wyjątków oraz zapobiega duplikacji. Wzorzec ten działa na podobnej zasadzie co plan budowy. `Dyrektor` odpowiedzialny za całą budowę zleca wykonanie poszczególnej pracy konkretnym `Budowniczym`. W efekcie połączenia ich pracy powstaje finalny `Produkt`. Mówiąc o wzorcu `Budowniczy` nie sposób nie wspomnieć o jego odmianie tzw. `Fluent Builder`. Jego rola sprowadza się do tworzenia obiektów (z dużą ilością parametrów) w czytelny sposób poprzez zastępienie konstruktora.

## Ograniczenia
Stosowanie wzorca wymusza tworzenie konkretnego `Budowniczego` dla każdego typu `Produktu`. Ponadto nie ma gwarancji inicjalizacji pól klasy, a `Wstrzykiwanie Zależności` może być utrudnione.

## Użycie
`Budowniczy` używany jest w implementacji złożonych obiektów (`kompozytów`), które mogą być budowane na różne sposoby, a ich inicjalizacja jest procesem wieloetapowym. Zastosowanie wzorca pozwala uniknąć tworzenia super klasy o rozbudowanej odpowiedzialności. Wykorzystywany jest w inicjalizacji `kompozytów` w wielu bibliotekach zewnętrznych oraz także systemowych. W sytuacjach, gdy konstruktor klas ma długą listę parametrów (ok 5) należy rozważyć zastosowanie `Fluent Builder`.

## Implementacja
W klasycznej wersji wzorca obiekt klasy `Director` zleca wykonanie produktu (`Product`) konkretnemu builderowi (`ConcreteBuilder`) nadzorując jego pracę. `Director` przechowuje referencje do `Buildera` i w momencie budowania produktu wywołuje na nim poszczególne operacje. Każdy budowniczy implementuje wspólne zachowania abstrakcyjnej klasy `Builder` lub interfejsu, tworząc produkt na swój określony sposób. `Builder` zawiera referencję do produktu i zwraca go nadzorcy w momencie zakończenia pracy.

![Budowniczy diagram](/assets/img/diagrams/builder.svg){: .center-image }

Poniższy listing przedstawia implementacje klasycznej postaci wzorca `Budowniczy`.

{% highlight java %}
public class Product {
	
    private String part1;
    private String part2;

    public void setPart1(String part1) {
    	this.part1 = part1;
    }

    public void setPart2(String part2) {
    	this.part2 = part2;
    }
}

public abstract class Builder {
    
    protected Product product;

    public Product getProduct() {
    	return product;
    }

    public abstract void buildPart1();
    public abstract void buildPart2();
}

public class ConcreteBuilder extends Builder {
    
    public void buildPart1() {
    	product.setPart1("oval");
    }

    public void buildPart2() {
    	product.setPart2("green");
    }
}

public class Director {
	
    private Builder builder;

    public Director(Builder builder) {
	    this.builder = builder;
    }

    public Product build() {
        builder.buildPart1();
        builder.buildPart2();
        return builder.getProduct();
    }
}
{% endhighlight %}

Klient korzystając z nadzorcy wykorzystuje zaimplementowany wzorzec w następujący sposób.

{% highlight java %}
Builder concreteBuilder = new ConcreteBuilder();
Director director = new Director(concreteBuilder);
Product product = director.build();
{% endhighlight %}

W wersji `Fluent Builder`, instancja klasy `Product` tworzona jest poprzez wywołanie statycznej klasy `Builder`. Stosując ten sposób tworzenia obiektów, należy zadbać o inicjalizacje wszystkich wymaganych pól oraz jej kolejność. Poniższy listing pokazuje podstawową implementacje wzorca w wariancie `Fluent Builder`.

{% highlight java %}
public class Product {
    
    private String part1;
    private String part2;

    private Product(Builder builder) {
        this.part1 = builder.part1;
        this.part2 = builder.part2;
    }

    public static class Builder {
        
        private String part1;
        private String part2;

        public Builder part1(String part1) {
            this.part1 = part1;
            return this;
        }
        
        public Builder part2(String part2) {
            this.part2 = part2;
            return this;
        }

        public Product build() {
            return new Product(this);
        }
    }
}
{% endhighlight %}

Dzięki takiemu zabiegowi zamiast tworzyć obiekt poprzez wywołanie jego konstruktora można posłużyć się jego budowniczym.

{% highlight java %}
Product product = new Product.Builder()
        .part1("oval")
        .part2("green")
        .build();
{% endhighlight %}

## Przykład
Aplikacja wspomaga pracowników sieci gastronomicznej w przyjmowaniu i wydawaniu zamówień. Pracownik obsługi `Staff` wybiera na ekranie wskazane przez klienta zdefiniowane zestawy obiadowe `Meal`. Sztab kucharzy `Cook` w którym każdy specjalizuje się w konkretnym daniu otrzymuje listę zamówień do realizacji. Po przygotowaniu posiłku przekazuje go do obsługi, a stamtąd trafia ono do klienta. Poniższy listing przedstawia sposób realizacji składania zamówień przy użyciu `Budowniczego`.

{% highlight java %}
public class Meal {

    private Burger burger;
    private Beverage beverage;
    private Extra extra;

    public void addBurger(Burger burger) {
        this.burger = burger;
    }

    public void addBeverage(Beverage beverage) {
        this.beverage = beverage;
    }

    public void addExtra(Extra extra) {
        this.extra = extra;
    }
}

public abstract class Cook {

    protected Meal meal;

    public Meal getMeal() {
        return meal;
    }

    public abstract void prepareBurger();
    public abstract void prepareBeverage();
    public abstract void prepareExtra();
}

public class BigMeal extends Cook {

    @Override
    public void prepareBurger() {
        meal.addBurger(new Burger("Bacon Burger"));
    }

    @Override
    public void prepareBeverage() {
        meal.addBeverage(new Beverage("Lager Beer"));
    }

    @Override
    public void prepareExtra() {
        meal.addExtra(new Extra("French fries"));
    }
}

public class KidsMeal extends Cook {

    @Override
    public void prepareBurger() {
        meal.addBurger(new Burger("Cheeseburger"));
    }

    @Override
    public void prepareBeverage() {
        meal.addBeverage(new Beverage("Orange Juice"));
    }

    @Override
    public void prepareExtra() {
        meal.addExtra(new Extra("Apple"));
    }
}

public class Staff {

    private Cook builder;

    public Staff(Cook builder) {
        this.builder = builder;
    }

    public Meal makeMeal() {
        builder.prepareBurger();
        builder.prepareBeverage();
        builder.prepareExtra();
        return builder.getMeal();
    }
}
{% endhighlight %}

Dodatkowo klient może skomponować swój własny zestaw. W tym celu pracownik obsługi ręcznie wybiera produkty. Poniższy listing przedstawia implementację manualnego tworzenia zestawu przy użyciu `Fluent Builder`.

{% highlight java %}
public class SpecificMeal {

    private Burger burger;
    private Beverage beverage;
    private Extra extra;

    private SpecificMeal(Chef chef) {
        this.burger = chef.burger;
        this.beverage = chef.beverage;
        this.extra = chef.extra;
    }

    public static class Chef {

        private Burger burger;
        private Beverage beverage;
        private Extra extra;

        public Chef prepareBurger(Burger burger) {
            this.burger = burger;
            return this;
        }
        
        public Chef prepareBeverage(Beverage beverage) {
            this.beverage = beverage;
            return this;
        }
        
        public Chef prepareExtra(Extra extra) {
            this.extra = extra;
            return this;
        }

        public SpecificMeal makeMeal() {
            return new SpecificMeal(this);
        }
    }
}
{% endhighlight %}

Proces wyboru zestawów wygląda następująco.

{% highlight java %}
Staff john = new Staff(new BigMeal());
Meal definedMeal = john.makeMeal();
SpecificMeal specificMeal = new SpecificMeal.Chef()
        .prepareBurger(new Burger("Double Cheesburger"))
        .prepareBeverage(new Beverage("Cola"))
        .prepareExtra(new Extra("Onion Rings"))
        .makeMeal();
//give meals to client
{% endhighlight %}

## Biblioteki
Wiele bibliotek zewnętrznych jak np.: `Retrofit` czy elementów systemu np.: `AlertDialog`, `Notification` korzysta z `Budowniczego`. Jednakże do samej implementacji wzorca przeważnie nie używa się bibliotek. 