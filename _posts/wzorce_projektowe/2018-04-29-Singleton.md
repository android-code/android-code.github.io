---
layout: post
title: "Singleton"
date:  2018-04-29
categories: ["Wzorce projektowe"]
image: singleton
---

## Zastosowanie
`Singleton` (wzorzec konstrukcyjny) ma za zadanie przede wszystkim ograniczenie tworzenia obiektów danej klasy do jednej instancji oraz zapewnieniu do niego globalnego dostępu. Swoje pochodzenie zawdzięcza `C++`, gdzie pełnił rolę następnika zmiennej globalnej używanej przez preprocesor. Jest wzorcem, który wzbudza wiele kontrowersji - z pozoru prosty, łatwy w implementacji, jednakże źle wykorzystany stanowi popularny antywzorzec. Dzieje się tak, gdy jego rola jest sprowadzona do zmiennej globalnej bez uwzględnienia kontekstu wykorzystania. Singleton można zastosować tam, gdzie rzeczywiście istnieje pewność występienia tylko jednej współdzielonej instancji klasy.

## Ograniczenia
`Singleton` tworzony jest w oparciu o prywatny konstruktor, który uniemożliwia rozszerzenie klasy. Poza zarządzaniem swoim cyklem życia pełni rolę logiki biznesowej czym łamie zasadę `SRP` - pojedynczej odpowiedzialności. Co więcej łamie zasadę `OCP` - otwarte/zamknięte, która mówi, że klasy powinnyć być otwarte na rozszerzenie lecz zamknięte na modyfikacje. Singleton utrudnia testowanie ponieważ przed wywołaniem testów należy pamiętać, aby był on właściwie zainicjonowany. Ze względu na architekturę systemu `Android`, `Singleton` nie jest zalecanym wzorcem. `Singleton Mutable` może utracić swój stan, gdy system potrzebuje zwolnić zasoby pamięci. Natomiast `Singleton Immutable` może być często zastąpiony klasami typu `Utility` (z metodami statycznymi).

## Użycie
Wzorzec ten jest wykorzystywany w dostępie do `SharedPreferences`. Popularną praktyką jest również jego implementacja dla obiektów typu `Logger` czy API dostępu do danych np.: `Retrofit`.

## Implementacja
Istnieje wiele implementacji tego wzorca. Jedna z najprostszych opiera się na inicjalizowaniu obiektu poprzez publiczną metodę `getInstance` dopiero w momencie jego pierwszego wywołania. Konstruktor jest niewidoczna spoza klasy. 

![Singleton diagram](/assets/img/diagrams/singleton.svg){: .center-image }

Poniższy listing przedstawia najprostszą implementacje Singleton. Rodzin ona jednak problemy w środowisku wielowątkowym.

{% highlight java %}
public class Singleton {
 
    private static Singleton INSTANCE;
 
    private Singleton() {
    }
 
    public static Singleton getInstance() {
        if(INSTANCE == null) {
            INSTANCE = new Singleton();
        }
        return INSTANCE;
    }
}
{% endhighlight %}

Aby Singleton mógł zostać użyty w wielowątkowych aplikacjach należy skorzystać z podwójnego sprawdzania i blokowania.

{% highlight java %}
public class DoubleCheckSingleton {
 
    private static DoubleCheckSingleton INSTANCE;
 
    private DoubleCheckSingleton() {
    }
 
    public static DoubleCheckSingleton getInstance() {
        if (INSTANCE == null)
            synchronized (DoubleCheckSingleton.class) {
                if (INSTANCE == null)
                    INSTANCE = new DoubleCheckSingleton();
            }
        return INSTANCE;
    }
}
{% endhighlight %}

Optymalnym rozwiąaniem jest Singleton Holder, który zapewnia leniwe tworzenie instancji oraz nie wymaga synchronizacji. 

{% highlight java %}
public class SingletonHolder {
 
    private SingletonHolder() {
    }
 
    private static class Holder {
        private static final SingletonHolder INSTANCE = new SingletonHolder();
    }
 
    public static SingletonHolder getInstance() {
        return Holder.INSTANCE;
    }
}
{% endhighlight %}

## Przykład
SharedPreferences umożliwia przechowywanie ustawień aplikacji. Dostęp do odczytu i zapisu zapewnia instancja klasy, a operacje te są od niej niezależne. Zatem prawdopodobnie nie ma potrzeby przechowywania w pamięciu wielu instancji SharedPreferences. Dostęp do ustawień aplikacji może być wykorzystywany w wielu miejscach dlatego zastosowanie wzorca Singleton wydaje się być uzasadnione. SharedPreferences w sposób niejawny korzystają z wzorca Singleton. Poniższy listing przedstawia jego jawną implementacje.

###
{% highlight java %}
public class SharedPref {

    private static SharedPref instance;
    private static SharedPreferences sharedPreferences;
    private static SharedPreferences.Editor editor;

    private SharedPref() {
    }

    public static SharedPref getInstance(Context context) {
        if(instance == null) {
            synchronized (SharedPref.class) {
                if(instance == null) {
                    instance = new SharedPref();
                    sharedPreferences = context.getSharedPreferences(context.getPackageName(), Activity.MODE_PRIVATE);
                    editor = sharedPreferences.edit();
                }
            }
        }
        return instance;
    }

    public void writeString(String key, String value) {
        editor.putString(key, value);
        editor.commit();
    }

    public void writeInt(String key, int value) {
        editor.putInt(key, value);
        editor.commit();
    }

    public void writeBoolean(String key, boolean value) {
        editor.putBoolean(key, value);
        editor.commit();
    }

    public String readString(String key) {
        return sharedPreferences.getString(key, "");
    }

    public int readInt(String key) {
        return sharedPreferences.getInt(key, 0);
    }

    public boolean readBoolean(String key) {
        return sharedPreferences.getBoolean(key, false);
    }

    public void clearPreferences() {
        editor.clear();
        editor.commit();
    }
}
{% endhighlight %}

## Biblioteki
Właściwym wykorzystaniem `Singleton` jest zastosowanie frameworka `Dagger 2`, implementującego wzorzec `Dependency Injection` (Wstrzykiwanie Zależności).