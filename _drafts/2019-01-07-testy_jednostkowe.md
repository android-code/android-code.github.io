---
layout: post
title: "Testy jednostkowe"
date:  2019-01-07
categories: ["Testowanie"]
image: testing/unit_test
description: "Testy jednostkowe"
keywords: "testowanie, testing, testy, jednostkowe, automatyczne, lokalne, integracyjne, zaślepka, atrapa, unit test, mock, stub, pokrycie kodu, pokrycie instrukcji, pokrycie gałęzi, pokrycie ścieżek, code coverage, statement coverage, path coverage, branch coverage, przypadki testowe, klasy równoważności, wartości brzegowe, tabele decyzyjne, przypadki użycia, junit, roboelectric, mockito, android, programowanie, programming"
---

## Definicja
`Testy jednostkowe` (`unit test`) poddają weryfikacji sposób działania różnych elementów składowych kodu takich jak `metody`, `klasy` czy `moduły`. Testy powinny być pisane w sposób `niezależny`, tzn. testowana jednostka nie jest podatna na wpływy innych elementów, w przeciwnym razie wynik testu może być niewiarygodny. Są to `testy automatyczne` za których tworzenie i wykonywanie powinien odpowiadać programista. Testy jednostkowe warto przeprowadzać często i regularnie przy każdym nowym przyroście dzięki czemu na wczesnym etapie tworzenia oprogramowania możliwe jest wykrycie usterki. Pozwala to na przeciwdziałanie kumulowania się błędów (jeden błąd rodzi kolejne) co w rezultacie redukuje koszty w póżniejszych etapach cyklu życia aplikacji. Tworzenie testów jednostkowych polega przede wszystkim na pisaniu `asercji` czyli porównywanie uzyskanego wyniku z oczekiwanym. Do przeprowadzania testów jednostkowych można wykorzystać np. bibliotekę `jUnit`. Przeważnie jednostki testowe wymagają do poprawnego działania obiektów różnego typu. Aby przekazywane argumenty nie wpływały na wynik testu danej jednostki (niezależne testy) należy dostarczać tzw. `atrapę` obiektu czyli naiwną implementację zależności.

## Testy lokalne i instrumentalne
Testy (nie tylko jednostkowe), które mogą zostać wykonane na wirtualnej maszynie deweloperskiej nazywane są `testami lokalnymi`. Przeprowadzenie jednostkowych testów lokalnych charakteryzuje się niskim kosztem oraz szybkością w związku z czym powinny stanowić przeważającą część testów. Natomiast testy, które wymagają uruchomienia na fizycznym urządzeniu lub emulatorze nazywane są `testami instrumentalnymi`. Wynika to z faktu, że pewne fragmenty kodu są zależne bibliotek `Android SDK` oraz cyklu życia komponentów, którymi zarządza system. Takie testy w stosunku do testów lokalnych są dosyć kosztowne i powolne. Przykładem biblioteki realizującej instrumentalne testy jednostkowe jest `Roboelectrict`. 

## Atrapa i zaślepka
`Zaślepka` (`stub`) jest minimalną implementacją zależnego modułu używaną podczas testów innego modułu. Ma za zadanie zastępować wywoływany moduł poprzez zwracanie w prosty sposób wyniku bez dokonywania obliczeń w taki sposób, aby wykonywany test zawsze przeszedł pozytywnie. `Atrapa` (`mock`) dostarcza natomiast naiwną implementację zależności, która umożliwia rejestrowanie interakcji z implementowanym interfejsem (np. ilość wywołań i parametry) i w przeciwieństwie do zaślepki bierze udział w procesie testowania. Atrapa jest determinowana w momencie działania programu i reprezentuje instancje oczekiwanego obiektu. Posługując się fachową nomenklaturą można wyróżnić jeszcze szerszy podział obiektów zastępczych jednakże ważniejsze od teoretycznego podziału jest właściwa implementacja. Biblioteka `Mockito` ułatwia tworzenie i zarządzanie naiwnymi implementacjami.

## Pokrycie kodu
Jedną z głównych miar mówiącą w jakim stopniu program został sprawdzony przez testy jest `pokrycie kodu` (`code coverage`). Metoda analityczną wskazującą, które części programu zostały pokryte zestawem testowym, a które nie. Wynik może być zwracany procentowo w odniesieniu do całego kodu. Metrykę pokrycia kodu nie należy traktować jako cel sam w sobie, ponieważ większy wynik nie zawsze musi oznaczać poprawę jakości kodu i co więcej może rodzić złudne przekonanie o bezbłędności danego fragmentu. Metryka ta doskonale wskazuje obszary kodu niepokryte testami oraz wymusza dodatkową analizę kodu. Można wyróżnić m.in. następujące kryteria pokrycia: instrukcji, gałęzi i ścieżek. 

//TODO example

`Pokrycie instrukcji` (`statement coverage`) obejmuje tylko rzeczywiste warunki, a każda linia kodu musi być sprawdzona. Sprawdza przepływ ścieżek oraz weryfikuje poprawność realizacji (czy robi to co jest oczekiwane). Nie weryfikuje natomiast fałszywych wyników, jest zależna od struktury kodu, pomija operatory logiczne i nie zgłasza warunku zakończenia pętli. Wynikiem pokrycia instrukcji jest iloczyn ilości linii kodu uruchomionych na skutek testów do wszystkich linii (z pominięciem instrukcji niewykonywalnych). 

`Pokrycie gałęzi` (`branch coverage`) sprawdza wyliczoną w trakcie działania programu wartość logiczną warunku oraz weryfikuje ilość pokrytych decyzji (ścieżek) warunku logicznego. Weryfikuje nieoczekiwane działanie programu oraz likwiduje problemy znane z pokrycia instrukcji. Ignoruje sposób wyliczenia wartości logicznej oraz wymaga analizy struktury kodu. Przypadki testowane dobierane są w taki sposób, aby uzyskać określone rezultaty decyzyjne. 

`Pokrycie ścieżek` (`path coverage`) wykonuje każdą ścieżke co najmniej raz wraz ze wszystkimi wariacjami warunków. Mierzy stosunek pokrycia przebytych ścieżek do wszystkich możliwych ścieżek wykonania funkcji.

## Projektowanie przypadków testowych 

## Dobre praktyki 
