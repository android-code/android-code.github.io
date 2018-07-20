---
layout: post
title: "Łańcuch zobowiązań"
date:  2018-08-13
categories: ["Wzorce projektowe"]
image: chain_responsibility
github: chain-responsibility
description: "Wzorce projektowe / behawioralny"
keywords: "łańcuch zobowiązań, łańcuch odpowiedzialności, chain of responsibility, wzorzec, wzorce projektowe, wzorzec behawioralny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Łańcuch zobowiązań` (ang. `Chain of responsibility`) (wzorzec behawioralny) zwany także `Łańcuchem odpowiedzialności` umożliwia przetwarzanie żądania przez łańcuch różnych obiektów. Łańcuch oparty jest o strukturę `listy jednokierunkowej`, tzn. obiekty mają ustaloną kolejność i znają swojego następnika. Klient zleca wykonanie operacji pierwszemu elementowi łańcucha, a gdy ten nie jest w stanie lub nie jest uprawniony do zakończenia żądania to przekazuje odpowiedzialność jego realizacji swojemu następnikowi. Jeśli następnik nie może podjąć działania wówczas również przekazuje je kolejnemu elementowi. Przetwarzanie i przekazywanie zadania działa rekurencyjnie dla każdego obiektu w łańcuchu. Elementy łańcucha analizują zadania i zapewniają jego obsługę lub przekazują zadanie dalej. Dzięki temu struktura łańcucha jest otwarta na modyfikacje, może być dynamicznie zmieniana, a także zmniejsza się liczba zależności klientem, a potencjalnymi odbiorcami. Spełnia zasadę `SRP` - pojedynczej odpowiedzialności oraz `OCP` - otwarte/zamknięte.

## Ograniczenia
Klient nie zna ostatecznego wykonawcy. Co więcej `Łańcuch zobowiązań` nie gwarantuje obsługi zleconego zadania. Ponadto w zależności od implementacji, żądanie może być częściowo obsługiwane przez różne elementy. Debugowanie staje się zatem utrudnione, a wydajność maleje.

## Użycie
Wzorzec używany jest w mechanizmach przetwarzania podobnych żądań, dla których proces klasyfikacji powinien przebiegać w ustalonym porządku. Klient nie musi znać dokładnego wykonawcy oraz szczegółów implementacji dzięki czemu wykorzystywany jest w sytuacjach dynamicznie zmieniających się wymagań.

## Implementacja
Abstrakcyjna klasa `Handler` definiuje szablon metody przepływu obsługi żądania, natomiast klasy ją rozszerzające `ConcreteHandler1`, `ConcreteHandler2` itd dostarczają szczegółów implementacji zadania.

![Łańcuch zobowiązań diagram](/assets/img/diagrams/chain_responsibility.svg){: .center-image }

Poniższy listing przedstawia implementację `Łańcucha zobowiązań` dla żądania `Request` i wykonawców typu `Handler`.

{% highlight java %}
public abstract class Handler {

    protected Handler successor;
    //some fields

    protected abstract void action(Request request);
    protected abstract boolean canExecute(Request request);

    public void setSuccessor(Handler successor) {
        this.successor = successor;
    }

    public void handle(Request request) {
        if(canExecute(request))
            action(request);
        else if(successor != null)
            successor.handle(request);
    }

    //some other methods
}

public class ConcreteHandler1 extends Handler {

    @Override
    protected void action(Request request) {
        //do specific action for ConcreteHandler1
    }

    @Override
    protected boolean canExecute(Request request) {
        if(request.getType() == Type.TYPE1)
            return true;
        else
            return false;
    }
}

public class ConcreteHandler2 extends Handler {

    @Override
    protected void action(Request request) {
        //do specific action for ConcreteHandler2
    }

    @Override
    protected boolean canExecute(Request request) {
        if(request.getType() == Type.TYPE2)
            return true;
        else
            return false;
    }
}

public class ConcreteHandler3 extends Handler {

    @Override
    protected void action(Request request) {
        //do specific action for ConcreteHandler3
    }

    @Override
    protected boolean canExecute(Request request) {
        if(request.getType() == Type.TYPE3)
            return true;
        else
            return false;
    }
}
{% endhighlight %}

Klient tworzy `Łańcuch zobowiązań` i zleca obsługę żądania.

{% highlight java %}
//create chain of responsibility depends
Handler main = new ConcreteHandler1();
Handler handler2 = new ConcreteHandler2();
Handler handler3 = new ConcreteHandler3();
main.setSuccessor(handler2);
handler2.setSuccessor(handler3);

//carry out request to chain
Request request1 = new Request("Content", Type.TYPE2);
main.handle(request1); //handler2 will act

Request request2 = new Request("Content", Type.TYPE4);
main.handle(request2); //no handler acts
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
Metoda `log` klasy `Logger` standardowej biblioteki `Java` wykorzystuje wzorzec `Łańcuch zobowiązań`.
