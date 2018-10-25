---
layout: post
title: "Dekorator"
date:  2018-05-14
categories: ["Wzorce projektowe"]
permalink: /blog/wzorce/:title/
image: patterns/decorator
github: design-patterns/tree/master/decorator
description: "Wzorce projektowe / strukturalny"
keywords: "dekorator, decorator, wzorzec, wzorce projektowe, wzorzec strukturalny, design patterns, android, java, decor, programowanie, programming"
---

## Zastosowanie
`Dekorator` (ang. `Decorator`) (wzorzec strukturalny) umożliwia rozszerzenie funkcjonalności lub modyfikacje stanu istniejącego obiektu bez zmiany jego struktury w trakcie działania programu. Jest alternatywą dla `dziedziczenia`, które w przeciwieństwie do `Dekoratora` zachodzi w trakcie kompilacji. Wzorzec ten jest swego rodzaju `Wrapperem` dla istniejącej klasy. Obiekt klasy podstawowej może być rozszerzony przez kilka dekoratorów umożliwiając tym samym tworzenie wielu wariantów zachowań bez ingerencji w strukturę rozszerzanej klasy. Proces ten odnosi się do jednego obiektu, a nie wszystkich instancji klasy. Dzięki temu struktura klas oraz dziedziczenia staje się przejrzysta, a projekt jest otwarty i elastyczny na przyszłe modyfikacje i nowe rozszerzenia.

## Ograniczenia
Zagrożenie płynące ze stosowania `Dekoratora` może być nadmiarowa ilość małych klas. Należałoby wówczas zastanowić się czy implementacja tego wzorca jest rzeczywiście najlepszym sposobem rozwiązania problemu. Dekorator jest trudny w zastosowaniu dla wielokrotnie 

## Użycie
`Dekorator` bywa często używany w implementacji kontrolek interfejsu - klas `View`. Można go spotkać w `strumieniach I/O w Javie`. Jest on implementowany tam, gdzie zachodzi potrzeba tworzenia wielu zachowań dla obiektów tego samego typu w zależności od miejsca ich wystąpienia, a użycie dziedziczenia jest zbyt kosztowne lub nie możliwe do przewidzenia. Takim przykładem może być proces składanie zamówienia danego produktu z opcją dodania do niego składników. Używając `Dekoratora` dla 5 rozszerzeń wystarczy utworzyć 5 Dekoratorów. Korzystając z `dziedziczenia`, aby zachować tą samą ilość typów należałoby stworzyć aż 31 klas!

## Implementacja
Podstawowe klasy komponentów `Component1`, `Component2` itd. oraz klasa bazowa dekoratora `Decorator` rozszerzają klasę abstrakcyjną lub implementują wspólny interfejs `Component`. `Decorator` przyjmuje jako parametr obiekt klasy `Component`. Klasy dekoratorów `Decorator1`, `Decorator2` itd. rozszerzają bazową klasę `Decorator`. Dekoratory implementują nowe funkcję oraz rozszerzają działanie metod klasy bazowej `Component`.

![Dekorator diagram](/assets/img/diagrams/patterns/decorator.svg){: .center-image }

Poniższe listing przedstawiają przykładową implementacje wzorca `Dekorator`.

{% highlight java %}
public abstract class Component {

    protected String name;

    public String getName() {
        return name;
    }

    public abstract int getValue();
}

public class Component1 extends Component {
    
    public Component1() {
        name = "Component1";
    }

    @Override
    public int getValue() {
        return 10;
    }
}

public abstract class Decorator extends Component {
    
    protected Component component;

    public Decorator(Component component) {
        this.component = component;
    }

    @Override
    public abstract String getName();
}

public class Decorator1 extends Decorator {
    
    public Decorator1(Component component) {
        super(component);
    }

    @Override
    public String getName() {
        return component.getName() + " + Decorator1";
    }

    @Override
    public int getValue() {
        return component.getValue() + 5;
    }
}

public class Decorator2 extends Decorator {
    
    public Decorator2(Component component) {
        super(component);
    }

    @Override
    public String getName() {
        return component.getName() + " + Decorator2";
    }

    @Override
    public int getValue() {
        return component.getValue() + 3;
    }
}
{% endhighlight %}

Tak zaimplementowany wzorzec można wykorzystać w następujący sposób.

{% highlight java %}
//base version of 'product'
Component component;
component = new Component1();

//decorating 'product'
component = new Decorator1(component);
component = new Decorator2(component);

component.getName(); //Component1 + Decorator1 + Decorator2;
component.getValue(); //18
{% endhighlight %}

## Przykład
Aplikacja jest szkicem procesu budowania awatara (`Component`) postaci w grze komputerowej. Gracz rozwija swoją postać na różnych etapach gry poprzez zdobywanie punktów siły oraz nowych umiejętności (`Decorator`). Interakcje między postaciami jak np.: walka zależą od siły ich profili. Profil ten zmienia się w czasie, a dzięki różnorodności zdobywanych umiejętności świat postaci jest różnorodny. Poniższy listing przedstawia podstawowe komponenty wzorca.

{% highlight java %}
public abstract class Avatar {

    public abstract String getDescription();
    public abstract int getPower();
    public abstract void fight();
}

public class Druid extends Avatar {

    @Override
    public String getDescription() {
        return "Druid";
    }

    @Override
    public int getPower() {
        return 10;
    }

    @Override
    public void fight() {
        //use magic against opponent
    }
}

public class Knight extends Avatar {

    @Override
    public String getDescription() {
        return "Knight";
    }

    @Override
    public int getPower() {
        return 30;
    }

    @Override
    public void fight() {
        //use full of power against opponent
    }
}
{% endhighlight %}

Postać w grzę zyskuje dodatkowe punkty siły, moce, a także zmienia sposób walki za pomocą następujących dekoratorów (profili).

{% highlight java %}
public abstract class AvatarProfile extends Avatar {

    protected Avatar avatar;

    public AvatarProfile(Avatar avatar) {
        this.avatar = avatar;
    }
}

public class Defender extends AvatarProfile {

    public Defender(Avatar avatar) {
        super(avatar);
    }

    @Override
    public String getDescription() {
        return avatar.getDescription() + " Defender";
    }

    @Override
    public int getPower() {
        return avatar.getPower() + 15;
    }

    @Override
    public void fight() {
        //use shield to protect yourself
    }
}

public class Fighter extends AvatarProfile {

    public Fighter(Avatar avatar) {
        super(avatar);
    }

    @Override
    public String getDescription() {
        return avatar.getDescription() + " Fighter";
    }

    @Override
    public int getPower() {
        return avatar.getPower() + 25;
    }

    @Override
    public void fight() {
        //use sword to attack an opponent
    }
}

public class Magician extends AvatarProfile {

    public Magician(Avatar avatar) {
        super(avatar);
    }

    @Override
    public String getDescription() {
        return avatar.getDescription() + " Magician";
    }

    @Override
    public int getPower() {
        return avatar.getPower() + 5;
    }

    @Override
    public void fight() {
        //cast spells to be invisible
    }
}
{% endhighlight %}

Schemat zdobywania nowych umiejętności oraz interakcji między postaciami (rozłożony w czasie) mógłby wyglądać jak na poniższym listingu.

{% highlight java %}
Avatar john = new Knight();
Avatar katy = new Druid();

john = new Fighter(john);
john = new Defender(john);
katy = new Magician(katy);

john.getDescription(); //Knight Fighter Defender
katy.getDescription(); //Druid Magician

john.fight();
katy.fight();   
{% endhighlight %}

## Biblioteki
Z uwagi na prostote wzorca, jego specyfikę i niski koszt implementacji używanie bibliotek implementujących wzorzec `Dekorator` mija się z celem. Najczęstszym jego wykorzystaniem w `Androidzie` jest rozszerzanie widoków. `Decor` jest przykładem biblioteki, która ułatwia dodawanie dekoratorów do klas `View`. Przykładem implementacji wzorca w `Javie` są wszystkie podklasy `InputStream`, `OutputStream`, `Reader` czy też `Writer`.