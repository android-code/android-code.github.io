---
layout: post
title: "Metoda szablonowa"
date:  2018-07-16
categories: ["Wzorce projektowe"]
image: template_method
github: design-patterns/tree/master/template-method
description: "Wzorce projektowe / behawioralny"
keywords: "metoda szablonowa, template method, szablon, template, wzorzec, wzorce projektowe, wzorzec behawioralny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Metoda szablonowa` (ang. `Template method`) (wzorzec behawioralny) ma za zadanie zdefiniować szkielet algorytmu, który jest otwarty na wprowadzanie zmian do wybranych jego części. Szablon ten składa się z operacji niezmiennych zdefiniowanych w klasie bazowej oraz z metod zmiennych tzw. `haczyków`, które przyjmują implementacje zależną od klas pochodnych. Dzięki temu zmniejsza się nadmiarowość kodu, a dokonanie w nim zmian staje się mniej kosztowne. Można powiedzieć, że metoda szablonowa dostarcza pewien standard realizacji tego samego zadania na różne sposoby. Spełnia zasadę `OCP` - otwarte/zamknięte, a także `DIP` - odwrócenia zależności.

## Ograniczenia
Wykorzystując wzorzec `Metoda szablonowa` należy zapewnić odpowiednią kontrolę dostępu do `metod haczyków` poprzez wymuszenie nadpisania (`metoda abstrakcyjna`) lub użycie chronionego modyfikatora dostępu. Metody niezmienne należy ukryć. Ze względu na konieczność przesłonięcia abstrakcyjnych metod haczyków w klasach pochodnych, warto dążyć do ich minimalizacji. Pozostałe haczyki mogą, ale nie muszą posiadać domyślnej implementacji. Decydując się na opcjonalne nadpisanie metod należy mieć pewność, że nie zaburzy to działania metody szablonowej.

## Użycie
`Metoda szablonowa` wykorzystywana jest w celu eliminacji powtórzeń kodu w metodach o tym samym przeznaczeniu, ale o różnej implementacji (w pewnych częściach) zależnej od podklas. Ponadto używa się jej tam, gdzie od metod o różnej implementacji oczekuje się podobnego sposobu działania.

## Implementacja
Klasa abstrakcyjna `AbstractClass` definiuje szkielet metody szablonowej `templateMethod`, dostarcza implementacje dla metod niezmiennych o prywatnym modyfikatorze dostępu oraz wymusza nadpisanie abstrakcyjnej metody zmiennej. Klasy rozszerzające dostarczają implementacji dla wszystkich metod zmiennych.

![Metoda szablonowa diagram](/assets/img/diagrams/template_method.svg){: .center-image }

Poniższy listing przedstawia implementację metody szablonowej w klasie `AbstractClass` oraz rozszerzenie implementacji w klasach pochodnych.

{% highlight java %}
public abstract class AbstractClass {

    public void templateMethod() {
        primitiveOperation1();
        primitiveOperation2();
        primitiveOperation3();
    }

    protected abstract void primitiveOperation2();

    private void primitiveOperation1() {
        //do something always
    }

    protected void primitiveOperation3() {
        //do something optional
    }

    //some others operations
}

public class ConcreteClass1 extends AbstractClass {

    @Override
    protected void primitiveOperation2() {
        //do custom work for ConcreteClass1
    }

    @Override
    protected void primitiveOperation3() {
        //do custom work for ConcreteClass2 by overriding default
    }
}

public class ConcreteClass2 extends AbstractClass {

    @Override
    protected void primitiveOperation2() {
        //do custom work for ConcreteClass2
    }
}
{% endhighlight %}

Klient tworzy obiekt żądanego typu i wywołuje na nim metodę szablonową, która wykonuje operacje w sposób dedykowany dla implementacji swojej klasy.

{% highlight java %}
AbstractClass object = new ConcreteClass1();
object.templateMethod();
{% endhighlight %}

## Przykład
Aplikacja `Navigation` poza swoją podstawową funkcjonalnością nawigowania oraz wyszukiwania trasy, umożliwia personalizowanie zapytania dla wybranych środków transportu `RouteFinder` (samochód `CarRoute`, ciężarówka, pieszo, transport publiczny `PublicTransportRoute` itp). Pomimo zachodzących różnic w sposobie znajdywania trasy, każdy proces wyszukiwania powinien przebiegać zgodnie ze stałym schematem (przetwarzanie zapytania, logowanie, zapis do historii, wyszukiwanie). Ponadto niektóre metody (np.: logowanie czy zapis) posiadają domyślną implementacje, jednakże ich zachowania mogą zostać nadpisane lub całkowicie pominięte. Dostawca rozkładów jazdy dla publicznych środków transportu wymaga przesłania informacji o pełnym zapytaniu użytkownika. Ze względu na powtarzalność kodu procesu wyszukiwania trasy, spełnienia wymogu spójnego zachowania tych procesów oraz możliwości edycji niektórych zachowań, problem ten może zostać zrealizowany przy użyciu wzorca `Metoda szablonowa` co przedstawia poniższy listing.

{% highlight java %}
public abstract class RouteFinder {

    private Route route = new Route();

    public Route find(Coordinates start, Coordinates destination) {
        route = new Route();
        Marker startPoint = Map.getMarker(start);
        Marker endPoint = Map.getMarker(destination);

        logSearching("Search trace: " + start.toString() + " - " + destination.toString());

        saveSearch(startPoint);
        saveSearch(destinationPoint);

        List<Marker> trackPoints = getTrackPoints(startPoint, destinationPoint);
        if(trackPoints.size() > 2) {
            List<Tip> tips = getRouteTips(trackPoints);
            route.setTrackPoints(trackPoints);  
            route.setTips(tips);
            return true;
        }
        else return false;
    }

    public Route getRoute() {
        return route;
    }

    protected abstract List<Marker> getTrackPoints(Marker start, Marker destination);
    protected abstract List<Tip> getRouteTips(List<Marker> trackPoints);

    protected void saveSearch(Marker point) {
        //add search to local memory to improve search suggestions
    }

    protected void logSearching(String text) {
    	Logger.log(text);
    }

    //some others operations
}

public class PublicTransportRoute extends RouteFinder {

    @Override
    protected List<Marker> getTrackPoints(Marker start, Marker destination) {
        List<Marker> markers = new ArrayList();
        //find the nereast station to start Marker and the nearest station to destination
        //connect with public transport providers to get possible routes between stations
        //add walk or car track points between start Marker and start station
        //add stations to track points
        //add walk or car track points between end station and destination Marker
        //add trackpoints from start to nerest station
        return markers;
    }

    @Override
    protected List<Tip> getRouteTips(List<Marker> trackPoints) {
        List<Tip> tips = new ArrayList();
        //connect with public transport providers to get schedule
        //add tips for every station
        return tips;
    }

    @Override
    protected void logSearching(String text) {
        super.logSearching(text);
        //and send analytics data to public transport providers
    }

    @Override
    protected void saveSearch(Marker point) {
        //do not save public transport searching
    }
}

public class CarRoute extends RouteFinder {

    @Override
    protected List<Marker> getTrackPoints(Marker start, Marker destination) {
        List<Marker> markers = new ArrayList();
        //find possible roads
        //add track all track points from the road
        return markers;
    }

    @Override
    protected List<Tip> getRouteTips(List<Marker> trackPoints) {
        List<Tip> tips = new ArrayList();
        //add tips for every crossing
        return tips;
    }
}
{% endhighlight %}

W oknie mapy użytkownik personalizuje zapytanie wyszukania trasy. Wybiera środek transportu oraz definiuje miejsce początkowe i docelowe.

{% highlight java %}
//check the user choice and set proper RouteFinder
RouteFinder routeFinder = new CarRoute();

//get coordinates of choosen start and destination points
Coordinates start = new Coordinates(52.409538, 16.931993); //Poznań
Coordinates destination = new Coordinates(52.237049, 21.017532); //Warsaw

//find route
//show progress
boolean success = routeFinder.find(start, destination);
//hide progress
if(success) {
    Route route = routeFinder.getRoute();
    //show route on map with tips
}
else {
    //show proper message
}
{% endhighlight %}

## Biblioteki
Wzorzec `Metody szablonowej` wykorzystywany jest m.in. w bibliotekach wspomagających automatyzację testów jednostkowych np.: `jUnit`. Ponadto idea wzorca znajduje zastosowanie tam, gdzie zachodzi pewien stały cykl życia obiektu. Cykl życia `Aktywności` w `Androidzie` może być rozpatrywany w kontekście idei `Metody szablonowej`.