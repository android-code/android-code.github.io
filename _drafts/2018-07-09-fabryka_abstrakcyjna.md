---
layout: post
title: "Fabryka abstrakcyjna"
date:  2018-07-09
categories: ["Wzorce projektowe"]
image: abstract_factory
github: abstract_factory
description: "Wzorce projektowe / kreacyjny"
keywords: "fabryka abstrakcyjna, abstract factory, fabryka, factory, wzorzec, wzorce projektowe, wzorzec kreacyjny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Fabryka abstrakcyjna` (ang. `Abstract factory`) (wzorzec kreacyjny) dostarcza mechanizm do tworzenia powiązanych bądź zależnych obiektów różnych typów bez dokładnej znajomości ich klas. Fabryka abstrakcyjna definiuje rodzinę produktów możliwych do wygenerowania z danej fabryki. Konkretne fabryki mają za zadanie stworzyć obiekty z rodziny produktów. //TODO

## Ograniczenia

## Użycie

## Implementacja
Konkretna fabryka implementuje abstrakcyjną fabrykę `AbstractFactory`, która deklaruje dostarczenie grupy produktów ze sobą powiązanych. Każda fabryka odpowiada za stworzenie produktów na swój ustalony sposób. Produkty rozszerzają klasę abstrakcyjną produktu `AbstractProduct` lub implementują jej interfejs.

![Fabryka abstrakcyjna diagram](/assets/img/diagrams/abstract_factory.svg){: .center-image }

Poniższy listing przedstawia implementację fabryki abstrakcyjnej w oparciu o dwie fabrykie i dwie rodziny produktów.

{% highlight java %}
public class FactoryA extend AbstractFactory {
	
	@Override
	public AbstractProduct1 createProduct1(Product1Type type) {
		//check args and set AbstractProduct1 type
		return new Product1A();
	}

	@Override
	public AbstractProduct2 createProduct2(Product2Type type) {
		//check args and set AbstractProduct2 type
		return new Product2A();
	}
}

public class FactoryB extend AbstractFactory {
	
	@Override
	public AbstractProduct1 createProduct1(Product1Type type) {
		//check args and set AbstractProduct1 type
		return new Product1B();
	}

	@Override
	public AbstractProduct2 createProduct2(Product2Type type) {
		//check args and set AbstractProduct2 type
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

## Biblioteki
Ze względu na specyfikację wzorca, implementacja `Fabryka abstrakcyjna` spoczywa na programiście. 