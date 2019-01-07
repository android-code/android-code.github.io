---
layout: post
title: "Testy jednostkowe"
date:  2019-01-07
categories: ["Testowanie"]
image: testing/unit_test
description: "Testowanie"
keywords: "testowanie, testing, testy, jednostkowe, automatyczne, lokalne, integracyjne, zaślepka, atrapa, unit test, mock, stub, przypadki testowe, czarnoskrzynkowe, białoskrzynkowe, pokrycie kodu, pokrycie instrukcji, pokrycie gałęzi, pokrycie ścieżek, code coverage, statement coverage, path coverage, branch coverage, przypadki testowe, klasy równoważności, wartości brzegowe, tabele decyzyjne, przypadki użycia, junit, robolectric, mockito, android, programowanie, programming"
---

## Definicja
`Testy jednostkowe` (`unit test`) poddają weryfikacji sposób działania różnych elementów składowych kodu takich jak `metody`, `klasy` czy `moduły`. Testy powinny być pisane w sposób `niezależny`, tzn. testowana jednostka nie jest podatna na wpływy innych elementów, w przeciwnym razie wynik testu może być niewiarygodny. Testy jednostkowe należą do `testów automatycznych` za których tworzenie i wykonywanie powinien odpowiadać programista. Warto przeprowadzać je często i regularnie przy każdym nowym przyroście dzięki czemu na wczesnym etapie tworzenia oprogramowania możliwe jest wykrycie usterki. Pozwala to na przeciwdziałanie kumulowania się błędów (jeden błąd rodzi kolejne) co w rezultacie redukuje koszty w póżniejszych etapach cyklu życia aplikacji. Tworzenie testów jednostkowych polega przede wszystkim na pisaniu `asercji` czyli porównywanie uzyskanego wyniku z oczekiwanym. Do przeprowadzania testów jednostkowych można wykorzystać np. bibliotekę `JUnit`. Przeważnie jednostki testowe wymagają do poprawnego działania obiektów różnego typu. Aby przekazywane argumenty nie wpływały na wynik testu danej jednostki (niezależne testy) należy dostarczać tzw. `atrapę` obiektu czyli naiwną implementację zależności.

## Testy lokalne i instrumentalne
Testy (nie tylko jednostkowe), które mogą zostać wykonane na wirtualnej maszynie deweloperskiej nazywane są `testami lokalnymi`. Przeprowadzenie jednostkowych testów lokalnych charakteryzuje się niskim kosztem oraz szybkością w związku z czym powinny stanowić przeważającą część testów. Natomiast testy, które wymagają uruchomienia na fizycznym urządzeniu lub emulatorze nazywane są `testami instrumentalnymi`. Wynika to z faktu, że pewne fragmenty kodu są zależne od bibliotek `Android SDK` oraz cyklu życia komponentów, którymi zarządza system. Takie testy w stosunku do testów lokalnych są dosyć kosztowne i powolne. Biblioteka `Robolectrict` umożliwia realizacje lokalnych testów jednostkowych zależnych od Android SDK co może być w wielu przypadkach alternatywą dla kosztownych testów instrumentalnych.

## Atrapa i zaślepka
`Zaślepka` (`stub`) jest minimalną implementacją zależnego modułu używaną podczas testów innego modułu. Ma za zadanie zastępować wywoływany moduł poprzez zwracanie w prosty sposób wyniku bez dokonywania obliczeń w taki sposób, aby wykonywany test zawsze przeszedł pozytywnie. `Atrapa` (`mock`) dostarcza natomiast naiwną implementację zależności, która umożliwia rejestrowanie interakcji z implementowanym interfejsem (np. ilość wywołań i parametry) i w przeciwieństwie do zaślepki bierze udział w procesie testowania. Atrapa jest determinowana w momencie działania programu i reprezentuje instancje oczekiwanego obiektu. Posługując się fachową nomenklaturą można wyróżnić jeszcze szerszy podział obiektów zastępczych jednakże ważniejsze od teoretycznego podziału jest właściwa implementacja. Biblioteka `Mockito` ułatwia tworzenie i zarządzanie naiwnymi implementacjami.

## Przypadki testowe
`Przypadkiem testowym` (`test case`) nazywany jest zbiór danych wejściowych, warunków początkowych, oczekiwanych rezultatów i końcowych warunków wykonania opracowanych w celu pokrycia pewnych warunków testowych. Projektowany przypadek testowy może być ogólny lub szczegółowy. Formalny zapis szczegółowego przypadku testowego może wyglądać następująco.

>**ID:** 1.0  
>**Tytuł:** Poprawne logowanie do aplikacji  
>**Środowisko:** Android 6.0 i wzwyż  
>**Warunek wstępny:** połączenie z internetem, użytkownik nie jest zalogowany  
>**Kroki do wykonania:**  
>- wpisz istniejący w bazie login: user
>- wpisz przypisane do loginu android hasło: password
>- wciśnij przycisk zaloguj  
>
>**Oczekiwany rezultat:** zalogowanie się do aplikacji jako "user" i przeniesienie do ekranu głównego  
>**Warunki końcowe:** użytkownik jest zalogowany  

Przypadki testowe są przydatne w automatyzacji testów ponieważ dokładnie opisują kroki, które powinny zostać zautomatyzowane. Dodatkowo pomagają `programiście` oraz `QA` dokładnie zrozumieć proces działania aplikacji. Odpowiedni dobór przypadków testowych pozwala dobrze i komplementarnie przetestować kod, dlatego tak ważna jest znajomość technik, które ułatwiają projektowanie właściwych testów. Techniki projektowania przypadków testowych można podzielić na `białoskrzynkowe`, `czarnoskrzynkowe` oraz bazujące na doświadczeniu (`testy eksploracyjne`). Techniki białoskrzynkowe oparte są o analizę wewnętrznej struktury kodu i są to przede wszystkim testy pokrycia kodu. Natomiast techniki czarnoskrzynkowe bazują na specyfikacji i definiują warunki odwołując się do analizy dokumentów opisujących funkcjonalność systemu.

## Pokrycie kodu
Jedną z głównych miar mówiącą w jakim stopniu program został sprawdzony przez testy jest `pokrycie kodu` (`code coverage`). Białoskrzynkowa (`white box`) metoda analityczna wskazująca, które części programu zostały pokryte zestawem testowym, a które nie. Wynik może być zwracany procentowo w odniesieniu do całego kodu. Metrykę pokrycia kodu nie należy traktować jako cel sam w sobie, ponieważ większy wynik nie zawsze musi oznaczać poprawę jakości kodu i co więcej może rodzić złudne przekonanie o bezbłędności danego fragmentu. Metryka ta doskonale wskazuje obszary niepokryte testami oraz wymusza dodatkową analizę kodu. Można wyróżnić m.in. następujące kryteria pokrycia: `instrukcji`, `gałęzi` i `ścieżek`.

>**Przykład**  
>Na podstawie funkcji `calculate` zostaną omówione typy wspomnianych pokryć wraz z ich wartościami dla zadanych argumentów.

{% highlight kotlin %}
fun calculate(a: Boolean, b: Boolean, c: Boolean) {
    print("some action")
    if(a)
        print("A")
    if(b) print("B")
  
    if(c)
        print("C")
    else {
        print("not C")
    }
}
{% endhighlight %}

`Pokrycie instrukcji` (`statement coverage`) zwane również `pokryciem linii` obejmuje tylko rzeczywiste warunki, gdzie każda linia kodu jest analizowana. Sprawdza przepływ ścieżek oraz weryfikuje poprawność realizacji (czy robi to co jest oczekiwane). Nie weryfikuje natomiast fałszywych wyników, jest zależna od struktury kodu, pomija operatory logiczne i nie zgłasza warunku zakończenia pętli. Wynikiem pokrycia instrukcji jest iloczyn ilości linii kodu uruchomionych na skutek testów do wszystkich linii (z pominięciem instrukcji niewykonywalnych).

>Dla podanych argumentów pokrycie instrukcji wynosi:  
>- **a**=true, **b**=true, **c**=true -> **statement coverage**=6/7
>- **a**=false, **b**=true, **c**=false -> **statement coverage**=5/7
>- **a**=true, **b**=false, **c**=false -> **statement coverage**=6/7
>
>Aby uzyskać całkowite pokrycie instrukcji należy przygotować 2 testy jednostkowe.  

`Pokrycie gałęzi` (`branch coverage`) zwane także `pokryciem decyzji` sprawdza wyliczoną w trakcie działania programu wartość logiczną warunku oraz weryfikuje ilość pokrytych decyzji (ścieżek) wynikających z warunków logicznych. Weryfikuje nieoczekiwane działanie programu oraz likwiduje problemy znane z pokrycia instrukcji. Ignoruje sposób wyliczenia wartości logicznej oraz wymaga analizy struktury kodu. Przypadki testowane dobierane są w taki sposób, aby uzyskać określone rezultaty decyzyjne.

>Dla podanych argumentów pokrycie gałęzi wynosi:  
>- **a**=true, **b**=true, **c**=true -> **branch coverage**=3/4
>- **a**=false, **b**=true, **c**=false -> **branch coverage**=2/4
>- **a**=true, **b**=false, **c**=false -> **branch coverage**=2/4
>
>Aby uzyskać całkowite pokrycie gałęzi należy przygotować 2 testy jednostkowe.   

`Pokrycie ścieżek` (`path coverage`) mierzy stosunek pokrycia przebytych ścieżek do wszystkich możliwych ścieżek wykonania funkcji. Struktura drzewa binarnego może posłużyć za diagram ścieżek. Żeby uzyskać całkowite pokrycie należy zatem sporządzić tyle testów jednostkowych co możliwych ścieżek.

>Dla podanych argumentów pokrycie ścieżek wynosi:    
>- **a**=true, **b**=true, **c**=true -> **path coverage**=1/8
>- **a**=false, **b**=true, **c**=false -> **path coverage**=1/8
>- **a**=true, **b**=false, **c**=false -> **path coverage**=1/8
>
>Aby uzyskać całkowite pokrycie gałęzi należy przygotować 8 testów jednostkowych.

## Techniki czarnoskrzynkowe
Czarnoskrzynkowa (`black box`) technika projektowania przypadków testowych w odróżnieniu do białoskrzynkowej techniki nie zajmuje się analizą struktury kodu lecz jako podstawę bierze specyfikacje dokumentową. Testowana jednostka traktowana jest jak czarna skrzynka w samolocie - nie wiadomo co dokładnie znajduje się w środku. Techniki czarnoskrzynkowe są spojrzeniem od zewnątrz na testowany obiekt. Do najpopularniejszych technik czarnoskrzynkowych można zaliczyć m.in. `klasy równoważności`, `testowanie wartości brzegowych`, `przejść między stanami`, `tabeli decyzyjnych` czy w oparciu o `przypadki użycia`.

>**Przykład**  
>Aplikacja do zamawiania jedzenia umożliwia zalogowanym użytkownikom dodawanie dań z jednej resturacji w ramach jednego zamówienia. Określa również minimalną kwotę zamówienia, aby mogło zostać zrealizowane z dowozem. Na podstawie przedstawionego procesu zamawiania i wyliczania kosztów dostawy zostaną przedstawione wybrane techniki czarnoskrzynkowe.

Projektowanie przypadków testowych oparciu o `klasy równoważności` polega na pokryciu każdej klasy równoważności przynajmniej raz, gdzie klasa równoważności jest podzbiorem dziedziny danych wejściowych lub wyjściowych, które na podstawie specyfikacji powinny zachowywać się tak samo.

>Klasy równoważności kosztów dostawy w stosunku do kwoty zamówienia mogą przedstawiać się następująco:
>
>**Kwota 0-9:** brak dostawy  
>**Kwota 10-49:** dostawa płatna  
>**Kwota >49:** dostawa darmowa  

`Wartość brzegowa` jest wartością wejścia lub wyjścia i znajduje się na granicy poprawnej klasy równoważności (lub w jej najbliższym sąsiedztwie w niepoprawnej klasie równoważności). W przypadku klas jednowymiarowej są to wartości minimalne i maksymalne.

>Niech:  
>**x<sub>1</sub>** - poprawna minimalna wartość graniczna  
>**x<sub>2</sub>** - poprawna maksymalna wartość graniczna  
>**y<sub>1</sub>** - niepoprawna minimalna wartość graniczna  
>**y<sub>2</sub>** - niepoprawna maksymalna wartość graniczna  

>Wówczas dla zadanego problemu dostawy płatnej wartości brzegowe przyjmują odpowiednio wartości:  
>**x<sub>1</sub>** = 10  
>**x<sub>2</sub>** = 49  
>**y<sub>1</sub>** = 9  
>**y<sub>2</sub>** = 50  

Testowanie `przejść między stanami` analizuje dozwolone i niedozwolone przejścia między stanami aplikacji za pomocą `automatu skończenie stanowego`. Analizując diagram lub tabelę stanową można zaprojektować testy w taki sposób by pokryły wszystkie przejścia, konkretny ciąg czy testowały przejścia zabronione.

>Diagram stanów wraz z możliwymi przejściami dla kosztów dostawy ukazany jest poniżej.
![Diagram stanów](/assets/img/diagrams/testing/unit_test_flow_states.svg){: .center-image }

Testowanie z użyciem `tabeli decyzyjnych` polega na sprawdzeniu działania jednostki testowanej w odniesieniu do kombinacji warunków wejściowych znajdujących się w tabeli.

>Przedstawione warunki składania zamówienia w poniższej tabeli implikują akcję lub możliwość jej podjęcia.

|:-----------:|:----------------------------:|:-:|:-:|:-:|:-:|---|
| **Warunek** |     Użytkownik zalogowany    | F | T | T | T | T |
|             |      Restauracja wybrana     |   | F | T | T | T |
|             |     Wartość koszyka >9zł     |   |   | F | T | T |
|             |     Wartość koszyka >49zł    |   |   |   | F | T |
| **Akcja**   |    Wyświetl stronę główną    | X |   |   |   |   |
|             |  Wyświetl panel użytkownika  |   | X |   |   |   |
|             |  Przejdź do listy produktów  |   |   | X | X | X |
|             |  Powiadom o braku dostawy    |   |   | X |   |   |
|             |  Powiadom o płatnej dostawie |   |   |   | X |   |
|             | Powiadom o darmowej dostawie |   |   |   |   | X |
|             |        Złóż zamówienie       |   |   |   | X | X |

Przypadki testowe tworzone w oparciu o `przypadki użycia` są projektowane w taki sposób, aby wykonane były scenariusze użytkownika co w odniesieniu do biznesowego charakteru przypadków użycia sprawia, że pozwalają wykryć usterki w przepływach proesów w czasie rzeczywistym. Stosowane są głównie w testach akceptacyjnych.

>Przypadek testowy dla aktualizacji kosztów dostawy w oparciu o przypadki użycia może przedstawiać się następująco:
>
>**UC01:** Aktualizacja kosztów dostawy  
>**Aktorzy:** Tester  
>**Scenariusz główny:**  
>1. Tester poprawnie loguje się do aplikacji
>2. System wyświetla panel użytkownika
>3. Tester wybiera restauracje
>4. System wyświetla dostępne dania
>5. Tester kompletuje zamówienie, aby jego wartość mieściła się w kwocie 10-49zł
>6. System informuje o możliwości płatnej dostawy
>7. Tester kompletuje zamówienie, aby jego wartość wynosiła przynajmniej 50zł
>8. System informuje o możliwości darmowej dostawy  
>
>**Rozszerzenie:**  
>7.A Tester kompletuje zamówienie, aby jego wartość wynosiła maksymalnie 9zł  
>7.A 1 System informuje o braku możliwości dostawy  
