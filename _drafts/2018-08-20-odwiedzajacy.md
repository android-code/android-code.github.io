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

Poniższy listing przedstawia wizytację obiektów typu `Visitor` dla klas typu `Element`.

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
        //do something specific for ConcreteVisitor2
        element.operation();
    }

    @Override
    public void visit(Element2 element) {
        //do something specific for ConcreteVisitor2
        element.action();
    }

    @Override
    public void visit(Element3 element) {
        //do something specific for ConcreteVisitor2
        element.work();
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
Księgarnia korzysta z oprogramowania `Bookstore`, które umożliwia zarządzanie aktualnym stanem zbiorów książek, filmów oraz gier komputerowych. Księgarnia chciałaby umożliwić czytelnikom sprawdzanie stanu swoich dostępnych zasobów przez internet. Ponadto dział techniczny zgłosił potrzebę dostępu do informacji o zasobach księgarni w formacie `json`. Ze względu na brak finansów oraz czasu niezbędnych do przetestowania potencjalnych zmian, dyrektor działu technicznego nie wyraził zgody na silną modyfikacje bieżącego kodu. Z myślą o niskim koszcie modyfikacji oraz rozszerzeniu generowanych plików o nowy format, programista podjął decyzję o wykorzystaniu wzorca Odwiedzający. Poniższy listing przedstawia zmiany dokonane w bieżącym kodzie poprzez implementacje interfejsu `Product` w klasach zasobów.

{% highlight java %}
public class Book implements Product {
	
    private String title;
    private int volume;
    private String description;
    private String author;
    private String isbn;
    private int pages;

    public Book(String title, int volume, String description, String author, String isbn, int pages) {
        this.title = title;
        this.volume = volume;
        this.description = description;
        this.author = author;
        this.isbn = isbn;
        this.pages = pages;
    }

    //the only change
    @Override
    public void generateFile(FileGenerator visitor) {
        visitor.createFile(this);
    }

    //getter methods
    //other methods
}

public class Film implements Product {
	
    private String title;
    private String description;
    private String director;
    private String scenario;
    private List<Actor> actors;
    private int year;

    public Film(String title, String description, String director, String scenario, List<Actor> actors, int year) {
        this.title = title;
        this.description = description;
        this.director = director;
        this.scenario = scenario;
        this.actors = actors;
        this.year = year;
    }

    //the only change
    @Override
    public void generateFile(FileGenerator visitor) {
        visitor.createFile(this);
    }

    //getter methods
    //other methods
}

public class Game implements Product {
	
    private String title;
    private String description;
    private String producer;
    private long budget;

    public Game(String title, String description, String producer, long budger) {
        this.title = title;
        this.description = description;
        this.producer = producer;
        this.budget = budget;
    }

    //the only change
    @Override
    public void generateFile(FileGenerator visitor) {
        visitor.createFile(this);
    }

    //getter methods
    //other methods
}

public interface Product {

    void accept(FileGenerator visitor);
}
{% endhighlight %}

Implementacja sposobu generowania plików została stworzona poza bieżącym kodem co prezentuje poniższy listing.

{% highlight java %}
public class HtmlGenerator implements FileGenerator {

    @Override
    public void createFile(Book element) {
        StringBuilder builder = new StringBuilder();
        builder.append("<html>").append("<head>");
        builder.append("<title>").append(element.getTitle()).append("</title>")
        builder.append("</head>").append("<body>");
        builder.append("<h1>").append(element.getTitle()).append("</h1>");
        builder.append("<h2>").append(element.getVolume()).append("</h2>");
        builder.append("<h3>").append(element.getAuthor()).append("</h3>");
        builder.append("<p>").append(element.getDescription()).append("</p>");
        builder.append("<p>Pages: ").append(element.getPages()).append("</p>");
        builder.append("<p>ISBN: ").append(element.getIsbn()).append("</p>");
        builder.append("</body>").append("</html>");
        String fileName = element.getTitle() + "_" + element.getVolume() + ".html";
        FileManager.write(fileName, builder.toString());
    }

    @Override
    public void createFile(Film element) {
        StringBuilder builder = new StringBuilder();
        builder.append("<html>").append("<head>");
        builder.append("<title>").append(element.getTitle()).append("</title>")
        builder.append("</head>").append("<body>");
        builder.append("<h1>").append(element.getTitle()).append("</h1>");
        builder.append("<h2>").append(element.getDirector())).append("</h2>");
        builder.append("<h3>").append(element.getScenario()).append("</h3>");
        for(Actor actor : element.getActors())
            builder.append("<h4>").append(actor.getName()).append("(").append(actor.getRole()).append(")");
        builder.append("<p>Year production: ").append(element.getYear()).append("</p>");
        builder.append("</body>").append("</html>");
        String fileName = element.getTitle() + ".html";
        FileManager.write(fileName, builder.toString());
    }

    @Override
    public void createFile(Game element) {
        StringBuilder builder = new StringBuilder();
        builder.append("<html>").append("<head>");
        builder.append("<title>").append(element.getTitle()).append("</title>")
        builder.append("</head>").append("<body>");
        builder.append("<h1>").append(element.getTitle()).append("</h1>");
        builder.append("<h2>").append(element.getProducer()).append("</h2>");
        builder.append("<p>").append(element.getDescription()).append("</p>");
        builder.append("<p>Budget: ").append(element.getBudget()).append("</p>");
        builder.append("</body>").append("</html>");
        String fileName = element.getTitle() + ".html";
        FileManager.write(fileName, builder.toString());
    }
}

public class JsonGenerator implements FileGenerator {

    @Override
    public void createFile(Book element) {
        StringBuilder builder = new StringBuilder();
        builder.append("{"));
        builder.append("title:").append("\"").append(element.getTitle()).append("\",");
        builder.append("volume:").append(element.getVolume()).append(",");
        builder.append("author:").append("\"").append(element.getAuthor()).append("\",");
        builder.append("description:").append("\"").append(element.getDescription()).append("\",");
        builder.append("pages:").append(element.getPages()).append(",");
        builder.append("isbn:").append("\"").append(element.getIsbn()).append("\"};");
        String fileName = element.getTitle() + "_" + element.getVolume() + ".json";
        FileManager.write(fileName, builder.toString());
    }

    @Override
    public void createFile(Film element) {
        StringBuilder builder = new StringBuilder();
        builder.append("{"));
        builder.append("title:").append("\"").append(element.getTitle()).append("\",");
        builder.append("director:").append("\"").append(element.getDirector()).append("\",");
        builder.append("scenario:").append("\"").append(element.getScenario()).append("\",");
        builder.append("actors:").append("[");
        for(int i=0; i<=element.getActors().size()-1; i++) {
            builder.append("\"").append(actor.getName() + "(" + actor.getRole() + ")").append("\"")
            if(i != element.getActors().size()-1)
                builder.append(",");
        }
        builder.append("],")
        builder.append("year:").append(element.getYear()).append("};");
        String fileName = element.getTitle() + ".json";
        FileManager.write(fileName, builder.toString());
    }

    @Override
    public void createFile(Game element) {
        StringBuilder builder = new StringBuilder();
        builder.append("{"));
        builder.append("title:").append("\"").append(element.getTitle()).append("\",");
        builder.append("producer:").append("\"").append(element.getProducer()).append("\",");
        builder.append("description:").append("\"").append(element.getDescription()).append("\",");
        builder.append("budget:").append(element.getBudget()).append("};");
        String fileName = element.getTitle() + "_" + element.getVolume() + ".json";
        FileManager.write(fileName, builder.toString());
    }
}

public interface Visitor {

    void createFile(Book product);
    void createFile(Film product);
    void createFile(Game product);
}
{% endhighlight %}

Dział techniczny korzystając z modyfikacji oprogramowania generuje pliki `html` oraz `json` dla wybranych zasobów.

{% highlight java %}
//get resources from database
List<Product> products = new ArrayList();

//add books
product.add(new Book("Hobbit", 1, "Description", "J.R.R. Tolkien", "15515377", 320));

//add films
List<Actor> actors = new ArrayList();
actors.add(new Actor("Russell Crowe", "Maximus"));
actors.add(new Actor("Joaquin Phoenix", "Kommodus"))
product.add(new Film("Gladiator", "Description", "Ridley Scott", "David Franzoni", actors, 2000));

//add games
product.add(new Game("Heroes of Might and Magic III", "Description", "Ubisoft", 1000000));

//generate html files for customers
FileGenerator visitor = new HtmlGenerator();
for(Product element : products)
    element.generateFile(visitor);

//generate json files for IT
visitor = new JsonGenerator();
for(Product element : products)
    element.generateFile(visitor);
{% endhighlight %}

## Biblioteki
Przykładem biblioteki implementującej wzorzec `Odwiedzający` są klasy `ElementVisitor` oraz `Element` (ze standardowego pakietu `Java`). Ze względu na ich generyczność, mogą być z powodzeniem zastosowane w wielu przypadkach bez konieczności tworzenia własnych interfejsów.