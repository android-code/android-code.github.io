---
layout: post
title: "Testy jednostkowe"
date:  2019-01-07
categories: ["Testowanie"]
image: testing/unit_test
description: "Testy jednostkowe"
keywords: "testowanie, testing, testy, jednostkowe, automatyczne, lokalne, integracyjne, zaślepka, atrapa, unit test, mock, stub, przypadki testowe, czarnoskrzynkowe, białoskrzynkowe, pokrycie kodu, pokrycie instrukcji, pokrycie gałęzi, pokrycie ścieżek, code coverage, statement coverage, path coverage, branch coverage, przypadki testowe, klasy równoważności, wartości brzegowe, tabele decyzyjne, przypadki użycia, junit, roboelectric, mockito, android, programowanie, programming"
---

## Definicja
`Testy jednostkowe` (`unit test`) poddają weryfikacji sposób działania różnych elementów składowych kodu takich jak `metody`, `klasy` czy `moduły`. Testy powinny być pisane w sposób `niezależny`, tzn. testowana jednostka nie jest podatna na wpływy innych elementów, w przeciwnym razie wynik testu może być niewiarygodny. Testy jednostkowe należą do `testów automatycznych` za których tworzenie i wykonywanie powinien odpowiadać programista. Warto przeprowadzać je często i regularnie przy każdym nowym przyroście dzięki czemu na wczesnym etapie tworzenia oprogramowania możliwe jest wykrycie usterki. Pozwala to na przeciwdziałanie kumulowania się błędów (jeden błąd rodzi kolejne) co w rezultacie redukuje koszty w póżniejszych etapach cyklu życia aplikacji. Tworzenie testów jednostkowych polega przede wszystkim na pisaniu `asercji` czyli porównywanie uzyskanego wyniku z oczekiwanym. Do przeprowadzania testów jednostkowych można wykorzystać np. bibliotekę `jUnit`. Przeważnie jednostki testowe wymagają do poprawnego działania obiektów różnego typu. Aby przekazywane argumenty nie wpływały na wynik testu danej jednostki (niezależne testy) należy dostarczać tzw. `atrapę` obiektu czyli naiwną implementację zależności.

## Testy lokalne i instrumentalne
Testy (nie tylko jednostkowe), które mogą zostać wykonane na wirtualnej maszynie deweloperskiej nazywane są `testami lokalnymi`. Przeprowadzenie jednostkowych testów lokalnych charakteryzuje się niskim kosztem oraz szybkością w związku z czym powinny stanowić przeważającą część testów. Natomiast testy, które wymagają uruchomienia na fizycznym urządzeniu lub emulatorze nazywane są `testami instrumentalnymi`. Wynika to z faktu, że pewne fragmenty kodu są zależne bibliotek `Android SDK` oraz cyklu życia komponentów, którymi zarządza system. Takie testy w stosunku do testów lokalnych są dosyć kosztowne i powolne. Przykładem biblioteki realizującej instrumentalne testy jednostkowe jest `Roboelectrict`.

## Atrapa i zaślepka
`Zaślepka` (`stub`) jest minimalną implementacją zależnego modułu używaną podczas testów innego modułu. Ma za zadanie zastępować wywoływany moduł poprzez zwracanie w prosty sposób wyniku bez dokonywania obliczeń w taki sposób, aby wykonywany test zawsze przeszedł pozytywnie. `Atrapa` (`mock`) dostarcza natomiast naiwną implementację zależności, która umożliwia rejestrowanie interakcji z implementowanym interfejsem (np. ilość wywołań i parametry) i w przeciwieństwie do zaślepki bierze udział w procesie testowania. Atrapa jest determinowana w momencie działania programu i reprezentuje instancje oczekiwanego obiektu. Posługując się fachową nomenklaturą można wyróżnić jeszcze szerszy podział obiektów zastępczych jednakże ważniejsze od teoretycznego podziału jest właściwa implementacja. Biblioteka `Mockito` ułatwia tworzenie i zarządzanie naiwnymi implementacjami.

## Przypadki testowe
`Przypadkiem testowym` (`test case`) nazywany jest zbiór danych wejściowych, warunków początkowych, oczekiwanych rezultatów i końcowych warunków wykonania opracowanych w celu pokrycia pewnych warunków testowych. Projektowany przypadek testowy może być ogólny lub szczegółowy. Formalny zapis szczegółowego przypadku testowego może wyglądać następująco.

ID: 1.0
Tytuł: Poprawne logowanie do aplikacji
Środowisko: Android 6.0 i wzwyż
Warunek wstępny: połączenie z internetem, użytkownik nie jest zalogowany
Kroki do wykonania:
- wpisz istniejący w bazie login: user
- wpisz przypisane do loginu android hasło: password
- wciśnij przycisk zaloguj
Oczekiwany rezultat: zalogowanie się do aplikacji jako "user" i przeniesienie do ekranu głównego
Warunki końcowe: użytkownik jest zalogowany

Przypadki testowe są przydatne w automatyzacji testów ponieważ dokładnie opisują kroki, które powinny zostać zautomatyzowane. Dodatkowo pomagają `programiście` oraz `QA` dokładnie zrozumieć proces działania aplikacji. Odpowiedni dobór przypadków testowych pozwala dobrze i komplementarnie przetestować kod, dlatego tak ważna jest znajomość technik, które ułatwiają projektowanie właściwych testóœ. Techniki projektowania przypadków testowych można podzielić na `białoskrzynkowe`, `czarnoskrzynkowe` oraz bazujące na doświadczeniu (`testy eksploracyjne`). Techniki białoskrzynkowe oparte są o analizę wewnętrznej struktury kodu i są to przede wszystkim testy pokrycia kodu. Natomiast techniki czarnoskrzynkowe bazują na specyfikacji i definiują warunki odwołując się do analizy dokumentów opisujących funkcjonalność systemu.

## Pokrycie kodu
Jedną z głównych miar mówiącą w jakim stopniu program został sprawdzony przez testy jest `pokrycie kodu` (`code coverage`). Białoskrzynkowa (`white box`) metoda analityczna wskazującą, które części programu zostały pokryte zestawem testowym, a które nie. Wynik może być zwracany procentowo w odniesieniu do całego kodu. Metrykę pokrycia kodu nie należy traktować jako cel sam w sobie, ponieważ większy wynik nie zawsze musi oznaczać poprawę jakości kodu i co więcej może rodzić złudne przekonanie o bezbłędności danego fragmentu. Metryka ta doskonale wskazuje obszary kodu niepokryte testami oraz wymusza dodatkową analizę kodu. Można wyróżnić m.in. następujące kryteria pokrycia: `instrukcji`, `gałęzi` i `ścieżek`.

//TODO example

`Pokrycie instrukcji` (`statement coverage`) obejmuje tylko rzeczywiste warunki, a każda linia kodu musi być analizowana. Sprawdza przepływ ścieżek oraz weryfikuje poprawność realizacji (czy robi to co jest oczekiwane). Nie weryfikuje natomiast fałszywych wyników, jest zależna od struktury kodu, pomija operatory logiczne i nie zgłasza warunku zakończenia pętli. Wynikiem pokrycia instrukcji jest iloczyn ilości linii kodu uruchomionych na skutek testów do wszystkich linii (z pominięciem instrukcji niewykonywalnych).

//TODO example

`Pokrycie gałęzi` (`branch coverage`) sprawdza wyliczoną w trakcie działania programu wartość logiczną warunku oraz weryfikuje ilość pokrytych decyzji (ścieżek) warunku logicznego. Weryfikuje nieoczekiwane działanie programu oraz likwiduje problemy znane z pokrycia instrukcji. Ignoruje sposób wyliczenia wartości logicznej oraz wymaga analizy struktury kodu. Przypadki testowane dobierane są w taki sposób, aby uzyskać określone rezultaty decyzyjne.

//TODO example

`Pokrycie ścieżek` (`path coverage`) wykonuje każdą ścieżke co najmniej raz wraz ze wszystkimi wariacjami warunków. Mierzy stosunek pokrycia przebytych ścieżek do wszystkich możliwych ścieżek wykonania funkcji.

//TODO example

## Techniki czarnoskrzynkowe
Czarnoskrzynkowa (`black box`) technika projektowania przypadków testowych w odróżnieniu do białoskrzynkowej techniki nie zajmuje się analizą struktury kodu lecz jako podstawę bierze specyfikacje dokumentową. Testowana jednostka traktowana jest jak czarna skrzynka w samolocie - nie wiadomo co dokładnie znajduje się w środku. Techniki czarnoskrzynkowe są spojrzeniem od zewnątrz na testowany obiekt. Do najpopularniejszych technik czarnoskrzynkowych można zaliczyć m.in. `klasy równoważności`, `testowanie wartości brzegowych`, `przejść między stanami`, `tabeli decyzyjnych` czy w oparciu o `przypadki użycia`.

Projektowanie przypadków testowych oparciu o `klasy równoważności` polega na pokryciu każdej klasy równoważności przynajmniej raz, gdzie klasa równoważności jest podzbiorem dziedziny danych wejściowych lub wyjściowych, które na podstawie specyfikacji powinny zachowywać się tak samo.

//TODO example

`Wartość brzegowa` jest wartością wejścia lub wyjścia i znajduje się na granicy poprawnej klasy równoważności (lub w jej najbliższym sąsiedztwie w niepoprawnej klasy równoważności). W przypadku klas jednowymiarowej są to wartości minimalne i maksymalne.

//TODO example

Testowanie `przejść między stanami` analizuje dozwolone i niedozwolone przejścia między stanami aplikacji za pomocą `automatu skończenie stanowego`. Analizując diagram lub tabelę stanową można zaprojektować testy w taki sposób by pokryły wszystkie przejście, konkretny ciąg czy testowały przejścia zabronione.

Testowanie z użyciem `tabeli decyzyjnych` polega na sprawdzeniu działania jednostki testowanej w odniesieniu do kombinacji warunków wejściowych znajdujących się w tabeli.

//TODO example

Przypadki testowe tworzone w oparciu o `przypadki użycia` są projektowane w taki sposób, aby wykonane były scenariusze użytkownika co w odniesieniu do biznesowego charakteru przypadków użycia sprawia, że pozwalają wykryć usterki w przepływach proesów w czasie rzeczywistym. Stosowane są głównie w testach akceptacyjnych.

//TODO example

## Dobre praktyki
