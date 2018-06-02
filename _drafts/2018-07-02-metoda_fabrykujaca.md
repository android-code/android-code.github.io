---
layout: post
title: "Metoda wytwórcza"
date:  2018-07-02
categories: ["Wzorce projektowe"]
image: factory_method
github: factory-method
description: "Wzorce projektowe / kreacyjny"
keywords: "metoda wytwórcza, factory method, fabryka, factory, wzorzec, wzorce projektowe, wzorzec kreacyjny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Metoda wytwórcza` (ang. `Factory method`) (wzorzec kreacyjny) umożliwia tworzenie obiektów o wspólnym typie poprzez dostarczenie interfejsu do tworzenia nieokreślonych instancji jednego typu. Zatem fabryki `Factory` `Metody wytwórczej` są ściśle związane z rodziną produktów jednego typu `Product`. Na podstawie przekazanych argumentów bądź innych warunków zależnych od implementacji fabryki, tworzy ona egzemplarz konkretnego typu. Dzięki temu centralizuje oraz hermetyzuje się proces tworzenia obiektów co ułatwia wprowadzanie zmian w istniejącym kodzie. Dodatkowo niweluje zależnosci między implementacją, a zastosowaniem produktu. Klient nie musi znać dokładnego typu produktu oraz wywołania konstruktora, cała odpowiedzialność spada na `Metodę fabrykującą`. Spełnia zasadę `OCP` - otwarte/zamknięte, a także `DIP` - odwrócenia zależności.

## Ograniczenia
Wzorzec ten może wprowadzać zbyt duży poziom abstrakcji, tzn. klient nie wie z jakim obiektem ma doczynienia. Należy uważać, aby fabryka nie stała się super klasą, a jej rola została ograniczona do generowania obiektów. `Metoda wytwórcza` może być nadużywana, należy zatem umieścić ją tam gdzie rzeczywiście przyniesie ona korzyści, a nie wprowadzi tylko dodatkowy poziom skomplikowania.

## Użycie
`Metoda wytwórcza` jest stosowana tam gdzie pożądane jest ujednolicenie sposobu tworzenia obiektów danej rodziny oraz odcięcie klienta od szczegółów implementacji tworzenia obiektów. Ponadto dokładny typ obiektu nie musi być znany w trakcie tworzenia kodu, a decyzja o tym jaki obiekt zostanie wygenerowany zapada dynamicznie. Wzorzec ten warto użyć także, gdy klasa ma skomplikowany konstruktor.

## Implementacja
Konkretna fabryka `ProductFactory1` ma zadanie stworzyć obiekt pochodzący ze wspólnej rodziny. Realizuje to poprzez implementacje interfejsu fabryki danej rodziny produktów `ProductFactory`. Produkty `Product1`, `Product2,` itd muszą rozszerzać klasę bazową rodziny dla której zostanie dedykowana fabryka `ProductFactory`.

![Metoda wytwórcza diagram](/assets/img/diagrams/factory_method.svg){: .center-image }

Poniższy listing przedstawia implementacja wzorca `Metoda wytwórcza` dla produktów klasy `Product`.

{% highlight java %}
public class ProductFactory1 implements ProductFactory {
	
	@Override
	public Product createProduct(Type type) {
		switch(type) {
			case TYPE1: 
				return new Product1();
			case TYPE2:
				return new Product2();
			case TYPE3:
				return new Product3();
		}
		throw new IllegalArgumentException("Product type not recognized");
	}
}

public class Product1 extends Product {
	
	//other fields

	public Product1() {
		name = "Product1";
	}

	@Override
	public void action() {
		//do something
	}

	//other methods
}

public interface ProductFactory {
	
	Product createProduct(Type type);
}

public abstract class Product {

	String name;
	
	public String getName() {
		return name;
	}

	public abstract void action();
}
{% endhighlight %}

Klient może skorzystać wygenerować produkty za pomocą fabryki w następujący sposób.

{% highlight java %}
ProductFactory factory = new ProductFactory1();
Product product1 = factory.createProduct(TYPE1);
Product product2 = factory.createProduct(TYPE2);
{% endhighlight %}

## Przykład
//TODO

## Biblioteki
Implementacja `Metody wytwórczej` spoczywa na barkach programisty, dlatego nie biblioteki tego wzorca nie mają sensu.
