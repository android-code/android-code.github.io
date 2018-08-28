---
layout: post
title: "Strategia"
date:  2018-07-23
categories: ["Wzorce projektowe"]
image: strategy
github: strategy
description: "Wzorce projektowe / behawioralny"
keywords: "strategia, strategy, wzorzec, wzorce projektowe, wzorzec behawioralny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Strategia` (ang. `Strategy`) (wzorzec behawioralny) definiuje rodzinę wymiennych algorytmów (w postaci klas), które służą do rozwiązywania tego samego problemu na kilka różnych sposobów. Klient może dynamicznie wybrać strategie dla obiektu i dowolnym momencie zmienić ją na inną. Szczegóły implementacji konkretnych strategii są ukryte dla klienta, a ewentualna modyfikacja wiążę się zmianą kodu w klasie konkretnej strategii. Ponadto zwiększa możliwość niezależnego testowania strategii i klienta. W celu usprawnienia procesu wyboru strategii wzorzec ten może być łączony ze wzorcem `Metoda wytwórcza`. Spełnia zasadę `OCP` - otwarte/zamknięte.

## Ograniczenia
Ze względu na podobieństwa w strukturze, wzorzec ten może być mylony ze wzorcami `Adapter` i `Most`, których zastosowanie wynika z realizacji innych celów. Należy zatem się upewnić, że wybór wzorca `Strategia` jest zasadny. W sytuacji, gdy obiekt kontekstu nie wymaga przekazania strategii w konstruktorze, należy zadbać o jej domyślną implementacje. Dodatkowo wzrasta koszt komunikacji między obiektami, a w przypadku wielu implementacji algorytmu powstaje wiele klas.

## Użycie
Wzorzec ten podobnie jak `Adapter` wykorzystuje mechanizm delegowania operacji do innego obiektu. Ponadto używa podobnej struktury jak `Most`, jednakże celem wzorca `Most` jest zaprojektowanie odpowiedniej struktury klas projektu, natomiast `Strategia` skupia się zmienności zachowania obiektu. `Strategia` ma zastosowanie w sytuacjach, gdzie dane zadanie może zostać zrealizowane na wiele różnych sposobów, a decyzja o wyborze implementacji zapada dynamicznie.

## Implementacja
Klasa kontekstu `Context` zawiera referencję do obiektu klasy strategii `AbstractStrategy`. Obiekt strategii może zostać wstrzyknięty przez konstruktor bądź metodę dostępową. Metoda klasy kontekstu wykorzystuje obiekt strategii w celu finalizacji operacji. Klasy strategii `ConcreteStrategy1`, `ConcreteStrategy2` itd. implementują metody interfejsu `AbstractStrategy`.

![Strategia diagram](/assets/img/diagrams/strategy.svg){: .center-image }

Poniższy listing przedstawia implementację wzorca `Strategia` wykorzystywaną w obiektach klasy `Context` w oparciu o dwa warianty `ConcreteStrategy1` oraz `ConcreteStrategy2`.

{% highlight java %}
public class Context {

    private Strategy strategy;

    public Context(Strategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }

    public void run(Object args) {
        strategy.action(args);
    }
}

public class Strategy1 implements Strategy {

    @Override
    public void action(Object args) {
        //do action specific for Strategy1
    }
}

public class Strategy2 implements Strategy {

    @Override
    public void action(Object args) {
        //do action specific for Strategy2
    }
}

interface Strategy {

    void action(Object args);
}
{% endhighlight %}

Klient przed wywołaniem metody docelowej dokonuje wyboru wstrzykniętej strategii w zależności od stanu i spełnionych warunków.

{% highlight java %}
Context context = new Context(new Strategy1());
String arg = "args passed to strategy";
context.run(arg);

//conditions have changed
context.setStrategy(new Strategy2());
arg = "new args for strategy";
context.run(arg);
{% endhighlight %}

## Przykład
Aplikacja sklepu internetowego znajdującego się na terenie Europy obsługuje dostawę swoich produktów do różnych rejonów świata. W zależności od rejonu lub kraju wysyłki naliczane jest odpowiednia opłata celna oraz wzrasta koszt wysyłki `EuropeShopping`, `AmericaShopping`. Niektóre produkty ze względu na gabaryt mogą nie być dostępne do wysyłki w dane miejsce. Aplikacja automatycznie wykrywa kraj klienta i dobiera odpowiednią strategie (`Shopping`) naliczania ceny całkowitej w walucie klienta (podstawową walutą jest euro). Poniższy listing prezentuje sposób realizacji doboru strategii naliczania ceny końcowej.

{% highlight java %}
public class Cart {

    private Shopping shopping;
    private List<Product> products;

    public Context(Shopping shopping) {
        this.shopping = shopping;
        products = new ArrayList();
    }

    public void setShoppingRegion(Shopping shopping) {
        this.shopping = shopping;
    }

    public double getTotalPrice() {
        return shopping.calculatePrice(products);
    }

    public boolean addProduct(Product product) {
        if(shopping.checkAvailability(product)) {
          products.add(product);
          return true;
        }
        else return false;
    }

    public boolean removeProduct(Product product) {
        return products.remove(product);
    }
}

public class EuropeShopping implements Shopping {

    @Override
    public double calculatePrice(List<Product> products) {
        double totalPrice = 0;
        double deliveryCost = 5;
        for(Product product : products) {
            totalPrice += product.getPrice();
            if(product.getSize() == Size.BIG)
                deliveryCost += 2;
        }
        return totalPrice;
    }

    @Override
    public boolean checkAvailability(Product product) {
        return true;
    }

    @Override
    public Currency getCurrency {
        return Currency.EUR;
    }
}

public class AmericaShopping implements Shopping {

    private final double USD_RATE = 0.75;
    private final double CUSTOMS_DUTY = 1.1;
    private final double CUSTOMS_DUTY_NORMAL = 1.5;

    @Override
    public double calculatePrice(List<Product> products) {
        double totalPrice = 0;
        double deliveryCost = 10;
        for(Product product : products) {
            if(product.getSize == Size.NORMAL) {
                totalPrice += product.getPrice() * CUSTOMS_DUTY_NORMAL;
                deliveryCost += 5;
            }
            else {
                totalPrice += product.getPrice() * CUSTOMS_DUTY;
                deliveryCost += 2;
            }
        }
        return (totalPrice + deliveryCost) * USD_RATE;
    }

    @Override
    public boolean checkAvailability(Product product) {
        if(product.getSize == Size.BIG)
            return false;
        else
            return true;
    }

    @Override
    public Currency getCurrency {
        return Currency.USD;
    }
}

interface Shopping {

    double calculatePrice(List<Product> products);
    boolean checkAvailability(Product product);
    Currency getCurrency();
}
{% endhighlight %}

Użytkownik dodaje produkty do swojego koszyka, a w przypadku braku możliwości wysłania produktu do danego regionu otrzymuje stosowany komunikat. Przed przystąpieniem do potwierdzenia zamówienia następuje proces wyliczania całkowitej ceny realizacji zamówienia.

{% highlight java %}
//check world region
Shopping shopping;
if(Region.EUROPE)
    shopping = new EuropeShopping();
else if(Region.AMERICA)
    shopping = new AmericaShopping();
else {
    //stop shopping or choose another strategy if is possible
}

//if delivery not available for products show message  
Cart cart = new Cart(shopping);
cart.addProduct(new Product(1, "Ball", 5, Size.NORMAL));  
cart.addProduct(new Product(2, "Notebook", 100, Size.NORMAL));
cart.addProduct(new Product(3, "Smartphone", 80, Size.SMALL));
cart.addProduct(new Product(4, "Desk", 50, Size.LARGE)); //in America this product is not available to deliver

//check total price at the end
double totalPrice = cart.getTotalPrice();
Currency currency = cart.getCurrency();
//Europe: 192EUR, America: 193,125USD (without Desk)
{% endhighlight %}

## Biblioteki
Przykładem biblioteki realizującej implementacje wzorca `Strategia` może być metoda `sort` klasy `Collections` (należącej do standardowego pakietu `Java`) do której wstrzykiwany jest `Comparator`. Wybór odpowiednich zasobów aplikacji (język, widoki, wielkości itp) w `Androidzie` w swojej idei przypomina wzorzec `Strategia`.
