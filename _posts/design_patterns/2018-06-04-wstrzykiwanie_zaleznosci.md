---
layout: post
title: "Wstrzykiwanie zależności"
date:  2018-06-04
categories: ["Wzorce projektowe"]
image: dependency_injection
github: dependency-injection
description: "Wzorce projektowe / Odwrócenie Sterowania"
keywords: "wstrzykiwanie zależności, dependency injection, di, wstrzykiwanie, zależności, odwrócenie sterowania, inversion of control, loose coupling, constructor injection, setter injection, interface injection, field injection, kontener di, wzorzec, wzorce projektowe, design patterns, android, java, programowanie, programming, dagger 2, butter knife"
---

## Zastosowanie
`Wstrzykiwanie zależności` (ang. `Dependency Injection`) jest wzorcem, który pozwala na eliminacje bezpośrednich zależności między elementami systemu. Wzorzec ten jest realizacją paradygmatu `Odwrócenie Sterowania` (ang. `Inversion of Control`), który polega na zamianie odpowiedzialności kodu, tzn. to kod frameworka wywołuje kod programisty. `Wstrzykiwanie zależności` spełnia zasadę pojedynczej odpowiedzialności `SRP`, otwarte/zamknięte `OCP` oraz odwrócenia zależności `DIP`. Dzięki takiemu podejściu testowanie jest ułatwione ze względu na modularność i możliwe wstrzykiwanie zaślepek (`mock`) do testowanego kodu. Realizacja wzorca następuje poprzez przekazanie zainicjalizowanych obiektów do klas, które z nich korzystają, dzięki czemu komponenty są ze sobą luźno powiązane (`loose coupling`) - nie przejmują odpowiedzialności innych obiektów. `Wstrzykiwanie zależności` może odbywać się m.in. poprzez konstruktor (`Constructor Injection`, metodę set (`Setter Injection`), interfejs (`Interface Injection`) czy też refleksje (`Field Injection`). Ponadto można wykorzystać tzw. `Kontener DI` (`DI Container`), który przejmuje odpowiedzialność inicjalizowania i wstrzykiwania obiektów w odpowiednie miejsca w odpowiednim czasie pod warunkiem zdefiniowania reguł powiązań.

## Ograniczenia
Korzystając ze `Wstrzykiwanie zależności` należy mieć na uwadze, że zastosowany w sposób nieumiejętny może doprowadzić do tworzenia cyklicznych zależności co jest sygnałem, że jakieś klasy nie spełniają zasady pojedynczej odpowiedzialności. Istnieje także ryzyko stworzenia super klasy poprzez wstrzyknięcie zbyt dużej ilości zależności. Z uwagi na możliwe zagrożenia refleksja nie jest zalecena do realizacji `Wstrzykiwania zależności`.

## Użycie
Ze względu na swoją charakterystykę i spełnienie trzech zasad `SOLID` użycie `Wstrzykiwania zależności` wydaje się naturalnym sposobem tworzenia większości aplikacji. Znacząco ułatwia testowanie, utrzymanie i rozwój kodu. Niektóre architektury realizujące paradygmat `Odwrócenia terowania` wymuszają użycie `Wstrzykiwania zależności` (np. `Spring`).

## Implementacja
Instancje klas, które mają zostać przekazane jako zależności do innego obiektu, są tworzone na zewnątrz poza jego ciałem. Wstrzyknięcie utworzonych zależności następuje poprzez konstruktor docelowego obiektu oraz jego metody. 

![Wstrzykiwanie zależności diagram](/assets/img/diagrams/dependency_injection.svg){: .center-image }

Poniższy listing przedstawia realizacja wzorca na trzy sposoby.

{% highlight java %}
public class ConstructorInjection {

    private Dependency1 dependency1;
    private Dependency2 dependency2;

    public ConstructorInjection(Dependency1 dependency1, Dependency2 dependency2) {
        this.dependency1 = dependency1;
        this.dependency2 = dependency2;
    }
}

public class SetterInjection {

    private Dependency1 dependency1;
    private Dependency2 dependency2;

    public void setDependency1(Dependency1 dependency1) {
        this.dependency1 = dependency1;
    }

    public void setDependency2(Dependency2 dependency2) {
    	this.dependency2 = dependency2;
    }
}

public class InterfaceInjection implements IInjection {

    private Dependency1 dependency1;
    private Dependency2 dependency2;

    @Override
    public void injectDependency1(Dependency1 dependency1) {
    	this.dependency1 = dependency1;
    }

    @Override
    public void injectDependency2(Dependency2 dependency2) {
    	this.dependency2 = dependency2;
    }
}

public interface IInjection {

    void injectDependency1(Dependency1 dependency1);
    void injectDependency2(Dependency2 dependency2);
}
{% endhighlight %}

Tak stworzone klasy mogą zostać zainicjalizowane różnymi obiektami typu `Dependency1` i `Dependency2`, co przedstawia poniższy listing.

{% highlight java %}
Dependency1 dependency1 = new Dependency1("A");
Dependency2 dependency2 = new Dependency2(10);
ConstructorInjection constructorInjection = new ConstructorInjection(dependency1, dependency2);

SetterInjection setterInjection = new SetterInjection();
setterInjection.setDependency1(new Dependency1(20), new Dependency2("B"));

InterfaceInjection interfaceInjection = new InterfaceInjection();
interfaceInjection.injectDependency1(new Dependency1(30), new Dependency2("C"))
{% endhighlight %}

Poniższy listing przedstawia implementacje klasy o podobnych właściwościach bez użycia wzorca.

{% highlight java %}
public class WithoutInjection {

    private Dependency1 dependency1;
    private Dependency2 dependency2;

    public WithoutInjection() {
    	this.dependency1 = new Dependency1(10);
    	this.dependency2 = new Dependency2("A");
    }
}
{% endhighlight %}

Jak łatwo zauważyć obiekty klas `Dependency1` oraz `Dependency2` przyjmują jeden stan zgodny z wewnętrzną implementacją klasy `WithoutInjection` co mocno wiąże zależności tej klasy.

## Przykład
W wielu miejscach aplikacji zachodzi potrzeba logowania przepływu operacji, interakcji użytkownika oraz napotkanych błędów. W tym celu używane są obiekty typu `Logger`. Jeśli użytkownik jest podłączony do internetu to logi wysyłane są bezpośrednio na serwery. W przeciwnym razie logi zapisywane są do pliku w pamięci urządzenia. Działanie aplikacji jest poddawane testom, zgodnie z którymi proces logowania ma zostać wyjęty z testów komponentów systemu. W tym celu tworzona jest zaślepka. Poniższy listing przedstawia implementacje komponentu `ComponentWithLogs` wykorzystującego `Wstrzykiwania zależności` przez konstruktor oraz logerów aplikacji.

{% highlight java %}
public class ComponentWithLogs {

    private Logger logger;

    public ComponentWithLogs(Logger logger) {
    	this.logger = logger;
    }

    public void operation1() {
    	logger.logClickEvent(getClickEvent());
    	try {
    		//do stuff
    		logger.logState(getState());
    	}
    	catch (Exception exception) {
    		logger.logError(getError(exception));
    	}
    }

    //other methods
}

public class NetworkLogger implements Logger {

    @Override
    public void logState(AppState appState) {
    	NetworkManager.send(appState);
    }

    @Override
    public void logClickEvent(ClickEvent clickEvent) {
    	NetworkManager.send(clickEvent);
    }

    @Override
    public void logError(Error error) {
    	NetworkManager.send(error);
    }
}

public class FileLogger implements Logger {

    @Override
    public void logState(AppState appState) {
    	FileManager.write(appState);
    }

    @Override
    public void logClickEvent(ClickEvent clickEvent) {
    	FileManager.write(clickEvent);
    }

    @Override
    public void logError(Error error) {
    	FileManager.write(error);
    }
}

public class MockLogger implements Logger {

    @Override
    public void logState(AppState appState) {
    	Console.log(appState.toString());
    }

    @Override
    public void logClickEvent(ClickEvent clickEvent) {
    	Console.log(clickEvent.toString());
    }

    @Override
    public void logError(Error error) {
    	Console.log(error.toString());
    }
}

public interface Logger {

    void logState(AppState appState);
    void logClickEvent(ClickEvent clickEvent);
    void logError(Error error);
}
{% endhighlight %}

Użycie `Wstrzykiwania zależności` logerów dla komponentu `ComponentWithLogs` aplikacji może odbywać się w nasępujący sposób.

{% highlight java %}
ComponentWithLogs component;
if(isInternetConnection)
    component = new ComponentWithLogs(new NetworkLogger());
else
    component = new ComponentWithLogs(new FileLogger());

//test class
ComponentWithLogs componentTest = new ComponentWithLogs(new MockLogger());

//do stuff
{% endhighlight %}

## Biblioteki
Popularnym frameworkiem dla `Androida` realizujący wstrzykiwanie zależności jest `Dagger 2`. Jest to `Kontener DI`, który ułatwia wstrzykiwanie zależności w całej aplikacji. Innym przykładem jest `Butter Knife`, którego zadaniem jest wiązanie widoków do obiektów.