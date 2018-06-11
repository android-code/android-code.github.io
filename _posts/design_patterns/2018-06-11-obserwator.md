---
layout: post
title: "Obserwator"
date:  2018-06-11
categories: ["Wzorce projektowe"]
image: observer
github: observer
description: "Wzorce projektowe / behawioralny"
keywords: "obserwator, observer, wzorzec, wzorce projektowe, design patterns, android, java, programowanie, programming, rxandroid, rxjava, szyna zdarzeń, eventbus, event bus, otto"
---

## Zastosowanie
`Obserwator` (ang. `Observer`) (wzorzec behawioralny) służy do powiadamiania obiektów zainteresowanych - subskrybentów o zmianie stanu obiektu śledzonego - obserwowanego. Obiekty nasłuchujące mogą być zależne od stanu obiektu obserwowanego w związku z czym musi istnieć mechanizm komunikacji między nimi. Zmiana stanu może być utożsamiana ze zmianą wartości pól obiektu. Wzorzec `Obserwator` może być stosowany również do powiadamiania o stanie podjętej operacji poprzez wygenerowanie odpowiedniego zdarzenia. Obiekt może być obserwowany przez wielu subskrybentów, a także sam może być obserwatorem innych obiektów. Komunikaty nadawane przez obiekt obserwowany trafiają do wszystkich obserwatorów.

## Ograniczenia
Obserwatorzy otrzymują komunikat z którego wiadomo, że coś się zmieniło, ale nie zawsze wiadomo co. Ponadto komunikat ten trafia do wszystkich subskrybentów, którzy niekoniecznie są zainteresowani daną zmianą stanu czy zdarzenia co wymusza na obserwatorach filtrowanie otrzymanego powiadomienia. Obserwatorzy nie wiedzą o istnieniu innych obserwatorów co może prowadzić do komplikacji. Jeśli obiekt obserwowany jest także obserwatorem wówczas istnieje ryzyko zapętetlęnia mechanizmu powiadamiania. 

## Użycie
`Obserwator` wykorzystywany jest tam, gdzie istnieje potrzeba informowania wielu obiektów w jednakowy sposób o zmianie stanu obiektu obserwowanego lub stanu podjętej operacji. Ze względu na ograniczenia wzorzec ten bywa przede wszystkim używany do informowania elementów systemu o wystąpieniu zdarzenia błędu czy też odpowiedzi sieciowej. 

## Implementacja
Obiekt obserwowany `ConcreteObservable` zawiera liste obserwatorów `Observer` oraz implementuje interfejs `Observable` za pomocą którego zapisuje, wypisuje i powiadamia obserwatorów o zmianie stanu. Obserwatorzy `Observer1`, `Observer2` implementują interfejs `Observer`. Opcjonalny obiekt przekazywany jako parametr metody aktualizującej może pełnić podobną rolę jak zdarzenia w programowaniu reaktywnym, tzn.: określa rodzaj zdarzenia wraz z zestawem danych (argumenty opakowane w event).

![Obserwator diagram](/assets/img/diagrams/observer.svg){: .center-image }

Poniższy listing przedstawia implementacje interfejsów obiektu obserwowanego `Observable` oraz jego obserwatorów `Observer`.

{% highlight java %}
public class ConcreteObservable implements Observable {

    private List<Observer> observers;

    //other fields

    public ConcreteObservable() {
        observers = new ArrayList();
    }
	
    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void unregisterObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers() {
        for(Observer observer : observers)
            observer.update(this, null);
    }

    @Override
    public void notifyObservers(Object object) {
        for(Observer observer : observers)
            observer.update(this, object);
    }

    public void method1() {
        //do some work
        notifyObservers();
    }

    public void method2() {
        //do some work
        Event event = new Event(); //args wrapped in event
        notifyObservers(event);
    }

    //others methods
}

public class Observer1 implements Observer {
	
    @Override
    public void update(Observable observable, Object object) {
        if(observable instanceof ConcreteObservable) {
            //do some work
        }
        else {
            //do some work
        }
    }

    //other methods
}

public class Observer2 implements Observer {
	
    @Override
    public void update(Observable observable, Object object) {
        if(observable instanceof ConcreteObservable) {
            if(object instanceof Event) {
                //do some work
            }
        }
    }

    //other methods
}

public interface Observable {

    public void registerObserver(Observer observer);
    public void unregisterObserver(Observer observer);
    public void notifyObservers(Object args);
    public void notifyObservers();
}

public interface Observer {

    public void update(Observable observable, Object args);
}
{% endhighlight %}

Obserwatorzy zapisują się do subskrypcji obiektu klasy `ConcreteObservable` co pokazuje poniższy listing.

{% highlight java %}
Observable observable = new ConcreteObservable();
Observer observer1 = new Observer1();
Observer observer2 = new Observer2();

observable.addObserver(observer1);
observable.addObserver(observer2);

//all Observers are notify
observable.method1();  //only observer1 do some update work 
observable.method2(); //only observer2 do some update work 
{% endhighlight %}

## Przykład
Aplikacja aukcyjna `AuctionHouse` służy do licytowania oraz śledzenia wybranych aukcji w trybie rzeczywistym. Interfejs graficzny użytkownika przedstawia listę śledzonych aukcji `Auctions`, a także szczegółowy widok wybranej aukcji wraz z listą ofert `Bids`. Gdy pojawi się nowa aukcja lub oferta wówczas główny komponent systemu powiadamia o tym fakcie pozostałe moduły, które dokonują aktualizacji. Poniższy listing przedstawia komponent `AuctionHouse`, którego zadaniem jest komunikacja przetwarzanie otrzymanych komunikatów sieciowych.

{% highlight java %}
public class AuctionHouse implements Observable {

    private List<Observer> observers;
    //other fields

    public AuctionHouse() {
        observers = new ArrayList();
    }

    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void unregisterObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers() {
        for(Observer observer : observers)
            observer.update(this, null);
    }

    @Override
    public void notifyObservers(Object args) {
        for(Observer observer : observers)
            observer.update(this, args);
    }

    public void newBid(NewBidEvent event) {
        //check event details and add if is OK
        Bid bid = event.getBid();
        notifyObservers(bid);
    }

    public void newAuction(NewAuctionEvent auction) {
        //check event details and add if is OK
        Auction auction = event.getAuction();
        notifyObservers(auction);
    }

    //others methods
}
{% endhighlight %}

Moduły Auctions i Bids obserwują stan komponentu AuctionHouse oczekując na zmianę stanu aukcji.

{% highlight java %}
public class Auctions implements Observer {

    private List<Auction> auctions;
    //other fields

    public Auction(List<Auction> auctions) {
    this.auctions = auctions;
    }

    @Override
    public void update(Observable observable, Object args) {
        if(observable instanceof AuctionManager) {
            if(args instanceof Auction) {
                Auction auction = (Auction) args;
                auctions.add(auction);
                //refresh UI list
            }
            else if(args == instanceof Bid) {
                Bid bid = (Bid) args;
                int auctionIndex = auctions.indexOf(bid.getAuction());
                if(auctionIndex != -1) {
                    auctions.get(auctionIndex).addBid(bid);
                    //refresh UI list
                }
            }
        }
    }

    //other methods
}

public class Bids implements Observer {

    private Auction auction;
    private List<Bid> bids;
    //other fields

    public Bids(Auction auction) {
        this.auction = auction;
        this.bids = auction.getBids();
    }

    @Override
    public void update(Observable observable, Object args) {
        if(observable instanceof AuctionManager) {
            if(args instanceof Bid) {
                Bid bid = (Bid) args;
                if(bid.getAuction() == auction) {
                    bids.add(bid);
                    //refresh UI list
                }
            }
        }
    }

    //other methods
}
{% endhighlight %}

Rejestracja modułów obserwatorów do głównego komponentu systemu oraz ich aktualizacja przedstawia się następująco.

{% highlight java %}
ActionHouse house = new ActionHouse();
Auctions auctions = new Auctions();
Bids bids = new Bids();

//register
house.registerObserver(auctions);
house.registerBids(bids);

//new action coming from server and runs ActionHouse
house.newBid(bidEvent); //update Bids and Auctions modules
house.newAuction(auctionEvent); //update Auctions module
{% endhighlight %}

## Biblioteki
Wzorzec `Obserwator` w implementacji klas `Observer` i `Observable` należał do standardowej biblioteki `Java`, jednakże ze względu na wspomniane zagrożenia oraz brak zapewnienia poprawnego działania w środowisku wielowątkowym, został uznany w `Java 9` jako `deprecated`. Zalecena jest korzystanie z klas `PropertyChangeListener` i `PropertyChangeEvent` lub `reaktywnych strumieni` z pakietu `Flow`. Popularnymi bibliotekami wzorca `Obserwator` w Androidzie są `szyny zdarzeń` `EventBus` i `Otto`, a także `RxAndroid` powstały na bazie `RxJava`, który jest implementacją podejścia do `programowania reaktywnego`.
