---
layout: post
title: "Single Activity"
date: 2019-09-09
categories: ["Wzorce architektoniczne"]
permalink: /blog/wzorce/:title/
image: architecture/single_activity
github: architectural-patterns/tree/master/single_activity
description: "Wzorce projektowe / architektoniczny"
keywords: "wzorzec, pattern, architektura, single, activity, fragment, navigation, android, kotlin, programowanie, programming"
---

## Activity
`Activity` (`Aktywność`) jest jednym z podstawowych komponentów systemu odpowiedzialnym za reprezentacje całego ekranu interfejsu oraz interakcje z użytkownikiem. Jeśli aplikacja posiada graficzny interfejs wówczas ma przynajmniej jedną `Activity`. W tym miejscu następuje renderowanie widoku, przechwytywanie akcji użytkownika czy startowanie innych komponentów. Użycie `Activity` tak jak innych komponentów jest ścisle związane z jego cyklem życia. Jednakże cykl życia `Activity` może sprawić sporo problemów ponieważ jest on przeważnie powiązany z operacjami wykonywanymi zarówno na głównym i roboczym wątku, a także odpowiada za cykl życia innych obiektów. W wyniku nieprawidłowego zarządzania cyklem życia może dojść do wycieku pamięci i innych błędów. Szczególnymi sytuacjami jest zejście do tła i powrót z tła oraz nawigowanie między `Activity`. Może to prowadzić do wniosku, że ograniczenie nawigacji między `Activity` zmniejszy ilość potencjalnych problemów. 

## Fragment 
`Fragment` reprezentuje pewną porcje interfejsu graficznego czy też zachowania, która jest częścią `Activity` lub innego `Fragment`. Dzięki zastosowaniu `Fragment` część interfejsu graficznego i zachowania może być współdzielona w wielu miejscach co zmniejsza ilość nadmiarowego kodu oraz ułatwia tworzenie aplikacji z dedykowanymi widokami dla telefonów i tabletów. `Activity` może być budowana w oparciu o obiekty typu `Fragment` w relacji jeden do jeden lub jeden do wielu, tzn. jeden lub wiele obiektów `Fragment` może być widocznych i aktywnych w tym samym czasie. `Activity` zarządza stosem w którym przechowywane są instancje `Fragment` wykorzystując w tym celu `FragmentManager` dzięki czemu może nawigować między obiektami `Fragment` i zmieniać je w czasie działania co nie jest czynnością trywialną. Cykl życia obiektów `Fragment` jest zależny od cyklu życia rodzica `Activity`.

## Kompozycja
Aplikacje mogą być tworzonone w oparciu o różne warianty wykorzystania `Activity` i `Fragment`. Podstawowy sposób polega na użyciu samodzielnych obiektów typu `Activity`, które są tworzone dla każdego ekranu. Nawigacja zachodzi za pomocą metod `startActivity`, `startActivityForResult` natomiast wymiana danych odbywa się poprzez obiekt typu `Bundle` lub jest zapisywana na dysku. Usprawnieniem tego podejścia jest wykorzystanie obiektów `Fragment` dzięki czemu część interfejsu i logiki biznesowej może być wyłączona z odpowiedzialności `Activity` oraz ponownie użyta w innej. Wymaga to jednak dodania mechanizmu komunikacji między `Activity` i obiektami `Fragment` co może być rozwiązane na wiele różnych sposobów w tym m.in. callback, szyna zdarzeń, programowanie reaktywne. Rozwinięciem tego podejścia jest zasada jednego aktywnego `Fragment` dla `Activity`, tzn. w zależności od stanu, `Activity` wyświetla odpowiedni `Fragment` wypełniający stałą przestrzeń - przeważnie cały ekran (`Fragment` może zawierać inne obiekty `Fragment`). Dzięki temu redukowana jest ilość `Activity` lecz nie eliminuje to problemów związanych z nawigacją i komunikacją. Odpowiedzią na wspomniane problemy może być zastosowanie architektury `Single Activity`, której działanie jest oparte o jedną centralną `Activity` zarzadzającą wieloma obiektami `Fragment`.

## Zastosowanie
`Single Activity` w większości przypadków zakłada istnienie jednej `głównej Activity` matki. Wszelkie zmiany widoku oraz nawigacja przebiega poprzez manipulacje stosem instancji `Fragment`. Innymi słowy zmiana widoku nie powoduje pokazania nowego okna lecz wymianę aktywnego `Fragment`. Z uwagi na eliminacje procesu nawigacji między `Activity` redukuje się ilość wywołań metod cyklu życia `Activity` co przekłada się na zmniejszenie ilości potencjalnych problemów. Komunikacja między ekranami zostaje sprowadzona do procesu komunikacji między obiektami `Fragment` dzięki czemu nie ma już potrzeby konstruowania i przetwarzania podatnego na błędy obiektu `Intent` w `startActivity`, `startActivityForResult` i `onActivityResult` czy też zapisywania danych na dysku lub w obiekcie `Singleton`. `Activity` staje się centralnym punktem zarządzania i współdzielenia jednego stanu oraz danych, a także widoków nawigacyjnych i paska statusu. Ponadto zastosowanie `Single Activity` zmniejsza ilość klas i wpisów do `AndroidManifest` oraz dba o dostarczenie lepszych wrażeń `User Experience` poprzez szybsze przejścia i animacje.

## Ograniczenia
Głównym zagrożeniem zastosowania `Single Activity` jest sprowadzenie `Activity` do roli super obiektu nawet pomimo implementacji logiki biznesowej w obiektach `Fragment`. Jedyną odpowiedzialnością `Activity` powino być współdzielenie stanu i manipulowanie obiektami `Fragment` oraz ewentualnie uczestniczenie w procesie komunikacji. Jednakże zarządzanie stosem o dużej ilości instancji `Fragment` przez `Activity` nie należy do łatwych. Co więcej należy brać pod uwagę także cykl życia wszystkich `Fragment`, który dodatkowo komplikuje sytuacje. Rozważając wybór architektury `Single Activity` należy zastanowić się nad bilansem zysków i strat. Być może lepszym wyborem będzie rezygnacja ze standardowego `Single Activity` z jedną `Activity` na rzecz `Single Activity` opartego o kilka `Activity`, które tym samym dokonują podziału aplikacji na niezależne lub słabo powiązane ze sobą obszary. Czynnikiem przemawiającym za taką decyzją poza biznesową niezależnością obszarów mogą być także różnice w pasku statusu i nawigacji. Niezależnie od wyboru architektury warto dążyć do minimalizacji ilości `Activity` na rzecz `Fragment`.

## Navigation
Biblioteka `Navigation` znacząco ułatwia tworzenia aplikacji w architekturze `Single Activity`. Automatycznie przejmuje odpowiedzialność zarządzania transakcjami i stosem Fragment co praktycznie wyklucza ręczne wywołanie operacji na `FragmentManager`. Ponadto w łatwy sposób dostarcza animacje i przejścia oraz wspiera nawigacyjne wzorce interfejsu użytkownika poprzez współprace z kontrolkami widoku dzięki czemu eliminuje problem aktualizacji pasku statusu i nawigacji. Co więcej implementuje przechwytywanie `Deep Link`, usprawnia użycie `ViewModel` oraz umożliwia jednoznaczną wymianę argumentów przy pomocy `SafeArgs`. Składa się trzech kluczowy elementów: grafu nawigacyjnego, pustego kontenera hostującego `NavHost` oraz kontrolera `NavController`odpowiedzialnego za zarządzanie nawigacją.

## Graf nawigacyjny
TODO

## NavHost
TODO

## NavController
TODO

## NavigationUI
TODO