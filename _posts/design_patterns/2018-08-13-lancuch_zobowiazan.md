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
Kantor wymiany walut wykorzystuje aplikację `Exchanger` do optymalizacji wydawanych nominałów w przeprowadzanych transakcjach. Kasjer otrzymuje zapytanie od klienta, a następnie definiuje zasady wydawanych nominałów. Na podstawie zleconej transakcji i zasad przeprowadzana jest symulacja, która w rezultacie podaje informacje o nominałach, które należy wypłacić klientowi. Do realizacji tego zadania wykorzystany został wzorzec `Łańcuch zobowiązań` do prezentuje poniższy listing. 

{% highlight java %}
public abstract class Nominal {

    protected Nominal successor;
    //some fields

    protected abstract void withdraw(Exchange transaction);
    protected abstract boolean canWithdraw(Exchange transaction);

    public void setSuccessor(Nominal successor) {
        this.successor = successor;
    }

    public Exchange calculate(Exchange transaction) {
        if(canWithdraw(transaction))
            withdraw(transaction);
        if(transaction.getLeftAmount() > 0 && successor != null)
            return successor.calculate(transaction);
        else
            return transaction;
    }

    //some other methods
}

public class ConcreteNominal extends Nominal {

    private final int NOMINAL;

    public Nominal(int nominal) {
        this.NOMINAL = nominal;
    }

    @Override
    protected void withdraw(Exchange transaction) {
        if(transaction.getLeftAmount() >= NOMINAL) {
            int count = transaction.getLeftAmount() / NOMINAL;
            int rest = transaction.getLeftAmount() % NOMINAL;
            transaction.setLeftAmount(rest);
            transaction.addOperation(count + " x Nominal " + NOMINAL);
        }
    }

    @Override
    protected boolean canWithdraw(Exchange transaction) {
        if(transaction.getLeftAmount() >= NOMINAL && isDepositEnough(transaction))
            return true;
        else
            return false;
    }

    private boolean isDepositEnough(Exchange transaction) {
        //check is concrete nominal enough in deposit 
        return true; //mocked
    }
}
{% endhighlight %}

Kasjer na podstawie zaleceń oraz stanu depozytu definiuje priorytety wydawanych nominałów. Wprowadza zlecenie klienta do systemu i dokonuje symulacji.

{% highlight java %}
//create default nominals scheme
Nominal nominal100 = new ConcreteNominal(100);
Nominal nominal50 = new ConcreteNominal(50);
Nominal nominal20 = new ConcreteNominal(20);
Nominal nominal10 = new ConcreteNominal(10)
Nominal nominal5 = new ConcreteNominal(5);
Nominal nominal2 = new ConcreteNominal(2);
Nominal nominal1 = new ConcreteNominal(1);

//cashier gets client's transaction
Exchange transaction1 = new Exchange(1500, Currency.PLN, Currency.USD);

//cashier defines default nominals scheme
nominal100.setSuccessor(nominal50);
nominal50.setSuccessor(nominal20);
nominal20.setSuccessor(nominal10);
nominal10.setSuccessor(nominal5);
nominal5.setSuccessor(nominal2);
nominal2.setSuccessor(nominal1);

//simulate transaction
Exchange result1 = nominal100.calculate(transaction1);
if(result.getLeftAmount() == 0) {
    //success - can withdraw
}
else {
    //fail - can not withdraw this transaction based on current nominals scheme
}
//show nominals to withdraw
result.getOperations();
//for course 3.67 the result is:
//4x100, 1x5, 1x2, 1x1

//cashier gets new transaction
Exchange transaction2 = new Exchange(5000, Currency.USD, Currency.GBP);
//nominals GBP are the same like USD so no need to update the chain

//in deposit there is a lot of nominal 1
//so cashier wants to spent nominal 1 before nominal 5 and 2
nominal10.setSuccessor(nominal1);
nominal1.setSuccessor(nominal2);
nominal2.setSuccessor(nominal5);

//simulate transaction like above
nominal100.calculate(transaction2);
//for course 0.761 and 100x20, 50x30, 10x20 nominals available in deposit to withdraw the result is:
//20x100, 30x50, 10x20, 10x10, 5x1 
{% endhighlight %}

## Biblioteki
Metoda `log` klasy `Logger` standardowej biblioteki `Java` wykorzystuje wzorzec `Łańcuch zobowiązań`.
