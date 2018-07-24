---
layout: post
title: "Odwiedzający"
date:  2018-08-20
categories: ["Wzorce projektowe"]
image: visitor
github: visitor
description: "Wzorce projektowe / behawioralny"
keywords: "odwiedzajacy, visitor, wzorzec, wzorce projektowe, wzorzec behawioralny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Odwiedzający` (ang. `Visitor`) (wzorzec behawioralny) ma za zadanie odsperować implementację algorytmu od struktury istniejących klas bez konieczności modyfikacji ich bieżącego kodu. Klasy rozszerzone o nową funkcjonalność akceptują obiekt `Odwiedzającego` (parametr implementowanej metody) i przenoszą na niego odpowiedzialność realizacji zadania. Obiekt `Odwiedzający` dostarcza implementacje algorytmu zgodnie z typem obiektu `Odwiedzonego`, który został przekazany jako parametr (`double dispatch`). Dodanie funkcjonalności do klas wymaga minimalnego nakładu zmian bez modyfikacji ich aktualnej struktury oraz logiki, a implementacja obiektów `Odwiedzających` może być wymienna. Spełnia zasadę `OCP` - otwartę/zamknięte.

## Ograniczenia
Należy unikać stosowania wzorca w przypadku niestabilnej, często zmieniającej się struktury i hierarchii klas, ponieważ zmiany te pociągają za sobą modyfikację klas `Odwiedzających`. Co więcej wzorzec narusza hermetyzację komponentów.

## Użycie
`Odwiedzający` ma zastosowanie w sytuacjach, gdy wymagane jest dodanie funkcjonalności dla zbioru klas, a modyfikacja struktury istniejących klas jest utrudniona lub niedozwolona.

## Implementacja
Klasy które wymagają rozszerzenia o nową funkcjonalność implementują interfejs `Element`. W metodzie akceptującej wizytację obiektu typu `Visitor`, przekazują referencje do własnej instancji i delegują wykonanie zadania obiektowi `Odwiedzającemu`. Klasy obiektów `Odwiedzających` implementują interfejs `Visitor`, dostarczając realizację tego samego zadania dla różnych typów obiektów `Odwiedzanych`. 

![Odwiedzający diagram](/assets/img/diagrams/visitor.svg){: .center-image }

Poniższy listing przedstawia wizytację obiektów typu Visitor dla klas typu Element.

{% highlight java %}
public class Element1 implements Element {

	@Override
	public void accept(Visitor visitor) {
		visitor.visit(this);
	}

	public void operation() {
		//do some work
	}
}

public class Element2 implements Element {

	@Override
	public void accept(Visitor visitor) {
		visitor.visit(this);
	}

	public void action() {
		//do some work
	}
}

public class Element3 implements Element {

	@Override
	public void accept(Visitor visitor) {
		visitor.visit(this);
	}

	public void work() {
		//do some work
	}
}

public interface Element {

	void accept(Visitor visitor);
}

public class ConcreteVisitor1 {

	@Override
	public void visit(Element1 element) {
		//do something specific for ConcreteVisitor1
		element.operation();
	}

	@Override
	public void visit(Element2 element) {
		//do something specific for ConcreteVisitor1
		element.action();
	}

	@Override
	public void visit(Element3 element) {
		//do something specific for ConcreteVisitor1
		element.work();
	}
}

public class ConcreteVisitor2 {

	@Override
	public void visit(Element1 element) {
		element.operation();
		//do something specific for ConcreteVisitor2
	}

	@Override
	public void visit(Element2 element) {
		element.action();
		//do something specific for ConcreteVisitor2
	}

	@Override
	public void visit(Element3 element) {
		element.work();
		//do something specific for ConcreteVisitor2
	}
}

public interface Visitor {

	void visit(Element1 element);
	void visit(Element2 element);
	void visit(Element3 element);
}
{% endhighlight %}

Klient dokonuje realizacji zadania poprzez wybór obiektu `Odwiedzającego` i wywołaniu metody akceptacji dla kolekcji elementów `Odwiedzanych`.

{% highlight java %}
//create some elements
List<Element> elements = new ArrayList();
elements.add(new Element1());
elements.add(new Element2());
elements.add(new Element3());

//choose visitor
Visitor visitor = new ConcreteVisitor1();
for(Element element : elements)
	element.accept(visitor); //do specific job for ConcreteVisitor1

//change visitor type
visitor = new ConcreteVisitor2();
for(Element element : elements)
	element.accept(visitor); //do specific job for ConcreteVisitor2
{% endhighlight %}

## Przykład
TODO

{% highlight java %}
TODO
{% endhighlight %}

TODO

{% highlight java %}
TODO
{% endhighlight %}

## Biblioteki
Przykładem biblioteki implementującej wzorzec `Odwiedzający` są klasy `ElementVisitor` oraz Element (ze standardowego pakietu `Java`). Ze względu na ich generyczność, mogą być z powodzeniem zastosowane w wielu przypadkach bez konieczności tworzenia własnych interfejsów.