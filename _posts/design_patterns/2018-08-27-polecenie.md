---
layout: post
title: "Polecenie"
date:  2018-08-27
categories: ["Wzorce projektowe"]
permalink: /blog/wzorce/:title/
image: patterns/command
github: design-patterns/tree/master/command
description: "Wzorce projektowe / behawioralny"
keywords: "polecenie, command, wzorzec, wzorce projektowe, wzorzec behawioralny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Polecenie` (ang. `Command`) (wzorzec behawioralny) enkapsuluje żądanie wykonania określonej czynności (metody) do postaci obiektu zwanego Poleceniem. Obiekty mogą być parametryzowane w zależności od typu odbiorcy oraz rejestrowane w kolejkach wywołań czy też dziennikach zdarzeń. Klasy wywołujące polecenia są rozdzielone od klas, które je wykonują dzięki czemu obiekt wywołujący (`Invoker`) operacje (`Command`) nie musi nic wiedzieć na jej temat, ponieważ wszystkie informacje zawarte są w przekazanym obiekcie. `Invoker` wywołuje odpowiedni obiekt `Command` bez wiedzy nt sposobu jego działania, natomiast obiekt `Command` dostarcza implementacji żądanej operacji bez wiedzy o tym kiedy zostanie wywołana.

## Ograniczenia
Zwiększa poziom skomplikowania kodu z uwagi na dużą ilość dodatkowych klas. Zapewniając możliwość cofania operacji należy liczyć się ze zwiększonym zużyciem zasobów pamięci.

## Użycie
Wzorzec `Polecenie` wykorzystywany jest w sytuacjach, gdzie obiekt wywołujący operacje nie zna jej implementacji oraz argumentów, a jego odpowiedzialnością jest wywołanie właściwego polecenia w odpowiednim momencie (może być odroczone w czasie). Ponadto znajduje zastosowanie, gdy wymagane jest kolejkowanie lub śledzenie żądań.

## Implementacja
Klasa `Invoker` decyduje o wyborze właściwego obiektu `Command` na którym wywołuje polecenie. W przypadku wymaganego kolejkowania bądź dziennika zdarzeń dodatkowo zarządza kolekcją poleceń. Klasy poleceń `Command1`, `Command2` implementują interfejs `Command` poprzez delegowanie wykonania operacji do obiektu pomocniczego `Receiver`. Jednakże w niektórych przypadkach obiekt `Receiver` jest pomijany, a implementacja operacji spoczywa wyłącznie na klasie polecenia.

![Polecenie diagram](/assets/img/diagrams/patterns/command.svg){: .center-image }

Poniższy listing przedstawia sposób wywołania operacji przy użyciu wzorca `Polecenie`.

{% highlight java %}
public class Invoker {

    //could be list of commands to provide additional history control if needed
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void start(Command command) {
        command.execute();
    }
}

public class Command1 implements Command {

    private Receiver receiver;

    public Command1(Receiver receiver) {
        this.receiver = receiver;
    }
    
    @Override
    public void execute() {
        //run specific action by itself or using receiver
        receiver.action1();
    }
}

public class Command2 implements Command {

    private Receiver receiver;

    public Command2(Receiver receiver) {
        this.receiver = receiver;
    }
    
    @Override
    public void execute() {
        //run specific action by itself or using receiver
        receiver.action2();
    }
}

public class Receiver {
    
    //fields and constructors

    public void action1() {
        //do action
    }

    public void action2() {
        //do action
    }
}

interface Command {
    
    void execute();
}
{% endhighlight %}

Klient w odpowiedzi na zdarzenie przekazuje odpowiedni obiekt polecenia.

{% highlight java %}
Invoker invoker = new Invoker();
Command command1 = new Command1();
Command command2 = new Command2();

//if user click something
invoker.setCommand(command1);
invoker.start();

//if user click something else
invoker.setCommand(command2);
invoker.start();
{% endhighlight %}

## Przykład
Aplikacja bankowa `BankOnline` pozwala swoim klientom na wykonywania podstawowych operacji informacyjnych takich jak sprawdzanie stanu konta, wyciągów czy podsumowań. Za pośrednictwem aplikacji klient może również dokonać przelewu, zlecić operacje stałe czy wziąć pożyczkę. Operacje finansowe w zależności od ustawień użytkownika wymagają opcjonalnej autoryzacji. Klient może autoryzować każdą operacje pojedynczo lub wykonać autoryzację zbiorczą. Ze względu na możliwość tworzenia operacji i ich późniejszego zbiorczego wykonania użyty został wzorzec `Polecenie`.

{% highlight java %}
public class AccountManager {

    private List<Order> orders = new ArrayList<>();

    public void takeOrder(Order order) {
    	//authorize
    	//if authorization success then execue order
    	order.execute();
    }

    public void takeOrders() {
    	//authorize
    	//if authorization success then execue orders
        for(Order order : orders) 
    		order.execute();
    }

    public void addOrder(Order order) {
    	orders.add(order);
    }

    public void clearOrders() {
    	orders.clear();
    }
}

public class StandingOrder implements Order {

    private OrderManager manager;
    private String recipient;
    private int amount, intervalDays;

    public StandingOrder(OrderManager manager, String recipient, int amount, int intervalDays) {
        this.manager = manager;
    }
    
    @Override
    public void execute() {
        manager.scheduleStandingOrder(recipient, amount, intervalDays);
    }
}

public class Transfer implements Order {

    private OrderManager manager;
    private String recipient, title;
    private int amount;

    public Transfer(OrderManager manager, String recipient, int amount, String title) {
        this.manager = manager;
    }

    @Override
    public void execute() {
        manager.transferMoney(recipient, amount, title);
    }
}

public class Loan implements Order {

    private OrderManager manager;
    private int amount, repaymentDays;

    public Loan(OrderManager manager, int amount, repaymentDays) {
        this.manager = manager;
    }
    
    @Override
    public void execute() {
        manager.takeLoan(amount, repaymentDays);
    }
}

public class OrderManager {

    //constructors and fields

    public void scheduleStandingOrder(String recipient, int amount, int intervalDays) {
        //do action
        Log.log("Standing order to: " + recipient + " for: " + amount + " with interval: " + intervalDays + " days");
        //save in history
    }

    public void transferMoney(String recipient, int amount, String title) {
        //do action
        Log.log("Transfer to: " + recipient + "for: " + amount + " with title: " + title);
        //save in history
    }

    public void takeLoan(int amount, int repaymentDays) {
        //do action
        Log.log("Loan for: " + amount + " to repayment until: " + manager.getRepaymentDate(repaymentDays));
        //save in history
    }

    public String getRepaymentDate(int repaymentDays) {
        return LocalDateTime.now().plusDays(repaymentDays).toString();
    }
}

interface Order {

    void execute();
}
{% endhighlight %}

Klient przechodzi do ekranu zlecania operacji na którym wprowadza dane i zleca ich wykonanie.

{% highlight java %}
//user navigate to account's orders view
AccountManager accountManager = new AccountManager();
OrderManager orderManager = new OrderManager();

//user create and execute single order
accountManager.takeOrder(new Loan(orderManager, 10000, 90)); //authorize and execute

//user decided to create orders and authorizate all of them once
accountManager.addOrder(new Transfer(orderManager, "12 3456 7890", 500, "For trip"));
accountManager.addOrder(new Transfer(orderManager, "30 4050 6070", 500, "Auction nr 987"));
accountManager.addOrder(new StandingOrder(orderManager, "11 2222 3333", 80, "Netflix"));

//authorize and take all orders
accountManager.takeOrders();
{% endhighlight %}

## Biblioteki
Klasa `Thread` oraz `Runnable` standardowego pakietu `Java` są przykładem implementacji wzorca `Polecenie`.