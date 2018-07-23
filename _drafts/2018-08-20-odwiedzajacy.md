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
`Odwiedzający` (ang. `Visitor`) (wzorzec behawioralny) 

## Ograniczenia
TODO

## Użycie
TODO

## Implementacja
TODO

![Odwiedzajacy diagram](/assets/img/diagrams/visitor.svg){: .center-image }

TODO

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

TODO

{% highlight java %}
//create some elements
List<Element> elements = new ArrayList();
elements.add(new Element1());
elements.add(new Element2());
elements.add(new Element3());

//choose visitor
Visitor visitor = new ConcreteVisitor1();
for(Element element : elements) 
	element.visit(visitor); //do specific job for ConcreteVisitor1

//change visitor type
visitor = new ConcreteVisitor2();
for(Element element : elements) 
	element.visit(visitor); //do specific job for ConcreteVisitor2
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
TODO
