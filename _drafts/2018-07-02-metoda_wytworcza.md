---
layout: post
title: "Metoda wytwórcza"
date:  2018-07-02
categories: ["Wzorce projektowe"]
image: factory_method
github: factory-method
description: "Wzorce projektowe / kreacyjny"
keywords: "metoda wytwórcza, factory method, fabryka, factory, wzorzec, wzorce projektowe, wzorzec kreacyjny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Metoda wytwórcza` (ang. `Factory method`) (wzorzec kreacyjny) umożliwia tworzenie obiektów o wspólnym typie poprzez dostarczenie interfejsu do tworzenia nieokreślonych instancji jednego typu. Zatem fabryki `Factory` `Metody wytwórczej` są ściśle związane z produktami jednego abstrakcyjnego typu `Product`. Na podstawie przekazanych argumentów bądź innych warunków zależnych od implementacji fabryki, tworzy ona egzemplarz konkretnego typu. Dzięki temu centralizuje oraz hermetyzuje się proces tworzenia obiektów co ułatwia wprowadzanie zmian w istniejącym kodzie. Dodatkowo niweluje zależnosci między implementacją, a zastosowaniem produktu. Klient nie musi znać dokładnego typu produktu oraz wywołania konstruktora, cała odpowiedzialność spada na `Metodę fabrykującą`. Spełnia zasadę `OCP` - otwarte/zamknięte, a także `DIP` - odwrócenia zależności.

## Ograniczenia
Wzorzec ten może wprowadzać zbyt duży poziom abstrakcji, tzn. klient nie wie z jakim obiektem współpracuje. Należy uważać, aby fabryka nie stała się super klasą, a jej rola została ograniczona do generowania obiektów. `Metoda wytwórcza` może być nadużywana, należy zatem umieścić ją tam gdzie rzeczywiście przyniesie ona korzyści, a nie wprowadzi tylko dodatkowy poziom skomplikowania.

## Użycie
`Metoda wytwórcza` jest stosowana tam gdzie pożądane jest ujednolicenie sposobu tworzenia obiektów danej rodziny oraz odcięcie klienta od szczegółów implementacji tworzenia obiektów. Ponadto dokładny typ obiektu nie musi być znany w trakcie tworzenia kodu, a decyzja o tym jaki obiekt zostanie wygenerowany zapada dynamicznie. Wzorzec ten warto użyć także, gdy klasa ma skomplikowany konstruktor.

## Implementacja
Konkretna fabryka `ProductFactory1` ma zadanie stworzyć obiekt pochodzący ze wspólnego typu. Realizuje to poprzez implementacje interfejsu fabryki danego typu produktów `ProductFactory`. Produkty `Product1`, `Product2,` itd muszą rozszerzać klasę bazową produktu dla której zostanie dedykowana fabryka `ProductFactory`.

![Metoda wytwórcza diagram](/assets/img/diagrams/factory_method.svg){: .center-image }

Poniższy listing przedstawia implementacja wzorca `Metoda wytwórcza` dla produktów klasy `Product`.

{% highlight java %}
public class ProductFactory1 implements ProductFactory {

	@Override
	public Product createProduct(Type type) {
		switch(type) {
			case TYPE1:
				return new Product1();
			case TYPE2:
				return new Product2();
			case TYPE3:
				return new Product3();
		}
		throw new IllegalArgumentException("Product type not recognized");
	}
}

public class Product1 extends Product {

	//other fields

	public Product1() {
		name = "Product1";
	}

	@Override
	public void action() {
		//do something
	}

	//other methods
}

//implement other Product class in similar way

public interface ProductFactory {

	Product createProduct(Type type);
}

public abstract class Product {

	String name;

	public String getName() {
		return name;
	}

	public abstract void action();
}
{% endhighlight %}

Klient może skorzystać wygenerować produkty za pomocą fabryki w następujący sposób.

{% highlight java %}
ProductFactory factory = new ProductFactory1();
Product product1 = factory.createProduct(TYPE1);
Product product2 = factory.createProduct(TYPE2);
{% endhighlight %}

## Przykład
Aplikacja `Maps` umożliwia przeglądanie na mapie listę punktów POI. Każdy z rodzajów POI jest przedstawiany na mapie jako ikona charakterystyczna dla danego typu. Ze względu na różnorodność typów POI, każdy z nich ma swój dedykowany widok szczegołów, który różni się szatą graficzną oraz przedstawianymi informacjami. Poniższy listing przedstawia implementacje generowania punktów POI za pomocą `Metody wytwórczej`.

{% highlight java %}
public class PoiMap implements PoiFactory {

    //some fields

  	@Override
  	public Poi createPoi(PoiInfo poiInfo) {
		switch(poiInfo.getType()) {
  			case MUSEUM:
  				return new Museum(poiInfo);
  			case SHOPPING:
  				return new Shopping(poiInfo);
  			case RESTAURANT:
  				return new Restaurant(poiInfo);
        	default:
            	return new DefaultPoi(poiInfo);
		}
  	}

    //some methods
}

public class Museum extends Poi {

    @Override
    public Icon getIcon() {
        return new Icon("Museum.svg");
    }

    @Override
    public void showCardInfo() {
        //generate card info for Museum based on PoiInfo
    }
}

public class Shopping extends Poi {

    @Override
    public Icon getIcon() {
        return new Icon("Shopping.svg");
    }

    @Override
    public void showCardInfo() {
        //generate card info for Shopping based on PoiInfo
    }
}

public class Restaurant extends Poi {

    @Override
    public Icon getIcon() {
        return new Icon("Restaurant.svg");
    }

    @Override
    public void showCardInfo() {
        //generate card info for Restaurant based on PoiInfo
    }
}

public class DefaultPoi extends Poi {

    @Override
    public Icon getIcon() {
        return new Icon("DefaultPoi.svg");
    }

    @Override
    public void showCardInfo() {
        //generate card info for Default based on PoiInfo
    }
}

public interface PoiFactory {

    Poi createPoi(Type type);
}

public abstract class Poi {

    PoiInfo poiInfo;

    public Poi(PoiInfo poiInfo) {
        this.poiInfo = poiInfo;
    }

    public String getName() {
        return poiInfo.getName();
    }

    public Coordinates getCoordinates() {
        return poiInfo.getCoordinates();
    }

    public abstract Icon getIcon();
    public abstract void showCardInfo();
}
{% endhighlight java %}

Jedną z funckjonalności nowej wersji mapy jest wybór wyświetlanego trybu dziennego / nocnego, co wpływa na zmianę kolorystyki mapy oraz punktów POI. Wystarczy zatem dodać nowe typy `Poi` oraz nową implementację `PoiFactory`.

{% highlight java %}
public class PoiNightMap implements PoiFactory {

	//some fields

  	@Override
  	public Poi createPoi(PoiInfo poiInfo) {
		switch(poiInfo.getType()) {
  			case MUSEUM:
  				 return new MuseumDark(poiInfo);
  			case SHOPPING:
  				 return new ShoppingDark(poiInfo);
  			case RESTAURANT:
  				 return new RestaurantDark(poiInfo);
        	default:
            	return new DefaultPoi(poiInfo);
		}
  	}

	//some methods
}

//implement Poi types in the same way as for day mode
{% endhighlight java %}

Komponent mapy aplikacji wykorzystuje `Metodę wytwórczą` do uzyskania listy punktów `Poi` w następujący sposób.

{% highlight java %}
//get pois info list from server like List<PoiInfo> poisInfo;
List<Poi> pois = new ArrayList();
PoiFactory poiFactory = new PoiMap(); //check if is day/night mode
for(PoiInfo poiInfo : poisInfo) {
    pois.add(poiFactory.createPoi(poiInfo));
    //show poi icon on map
}

//when user click on some poi
itemId = 0; //get itemId from click
pois.get(itemId).showCardInfo();

//if user click navigate on the card
pois.get(itemId).getCoordinates();
{% endhighlight java %}

## Biblioteki
Ze względu na specyfikację wzorca, implementacja `Metody wytwórczej` spoczywa na barkach programisty.
