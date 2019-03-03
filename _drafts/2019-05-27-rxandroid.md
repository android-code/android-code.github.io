---
layout: post
title: "RxAndroid"
date: 2019-05-27
categories: ["Biblioteki"]
image: libraries/rxandroid
github: libraries/tree/master/rxandroid
description: "Biblioteki"
version: RxAndroid 2.1, RxJava 2.2
keywords: "reaktywne, reactive, rxandroid, rxjava, reactivex, obserwator, obserwowany, observer, observable, schedulers, operator, disposable, subscribe, async, thread, java, android, programowanie, programming"
---

## Programowanie reaktywne
`Programowanie reaktywne` (`reactive programming`) jest paradygmatem programowania asynchronicznego zorientowanym na przepływ danych w postaci strumieni i propagacje zmian w formie zdarzeń. Strumieniem danych może być prawie wszystko, np. zmiana wartości obiektu, zdarzenia interfejsu użytkownika, zapytania sieciowe, operacje na danych czy błędy, a każde zadanie może być wykonywane na własnym wątku. Sposób działania opiera się na `wzorcu Obserwator`, tzn. strumienie mogą emitować wartości i być obserwowane. Realizacją programowania reaktywnego dla `Android` jest moduł `RxAndroid` rozszerzający bibliotekę `RxJava` (implementacja `ReactiveX` dla `Java VM`) o harmonogramy (`Schedulers`) wspierające wielowątkowość w środowisku Android. `RxJava` upraszcza zarządzanie asynchronicznymi i współbieżnymi operacjami, ułatwia komunikacje zadań w tle z wątkiem głównym, pozwala na wczesne wykrycie błędów i zmniejsza zapotrzebowanie na zmienne stanu.

## Wzorzec
Zadaniem wzorca `Obserwator` jest realizacja mechanizmu komunikacji między obiektami zależnymi poprzez powiadamianie obiektów subskrybentów o zmianie stanu obiektu obserwowanego czy statusie operacji. Obiekt może być obserwowany (`Observable`) przez wielu obserwatorów (`Observer`) i sam może być obserwatorem innych obiektów. Obiekt obserwowany emituje zdarzenia, które trafiają do obserwatorów, a następnie są przez nich przetwarzane. 

## Harmonogramy
Wprowadzone w module `RxAndroid` harmonogramy decydują o tym na jakim wątku obiekt obserwowany będzie wykonywał zadanie oraz na jakim wątku obiekt obserwatora będzie odbierał i przetwarzał wyemitowane dane. Najpopularniejszymi harmonogrami są: `AndroidSchedulers.mainThread` - odpowiada za główny wątek aplikacji, `Schedulers.io` - wykorzystywany w nie wymagających operacjach i `Schedulers.computation` - przeznaczony do przetwarzania intensywnych zadań. Wyróżnia się także `Schedulers.newThread`, `Schedulers.single`, `Schedulers.immediate`, `Schedulers.trampoline`, `Schedulers.from`, które różnią się synchronicznością, kolejnością, czasem rozpoczęcia i ilością przetwarzanych zadań. Przypisanie harmonogramu dokonywane jest na obiekcie obserwowanym za pomocą metod `subscribeOn` i `observeOn`.

//TODO example with register schedulers on Observable

## Obserwator
Obiekt obserwatora dokonuje subskrybcji obiektu obserwowanego za pomocą metody `subscribe` wywołanej na obiekcie obserwowanym oraz implementuje metody `onSubscribe`, `onNext`, `onError`, `onComplete` interfejsu `Observer` w celu podjęcia działania w odpowiedzi na stan i emisje obiektu obserwowanego.

//TODO example with Observer interface implementation

## Obserwowany
Obiekt obserwowany w `RxJava` jest strumieniem danych wykonującym operacje i emitującym dane. Podstawowym typem obiektu obserwowanego jest `Observable` - emituje dowolną ilość elementów i kończy się sukcesem lub zdarzeniem błędu. Wyróżnia się również także: `Flowable` - rozszerza `Observable` o wsparcie dla `backpressure` (zarządzanie czasem emitowania i konsumowania zdarzenia), `Single` - emituje jedno zdarzenie lub błąd, `Maybe` - może emitować jedno zdarzenie lub błąd, `Completable` - nie emituje zdarzenia, kończy się sukcesem lub błędem. Obiekt obserwowany może być tworzony za pomocą kilku operatorów jak np.: `create`, `just`, `defer`, `from`, `interval`, `range`, `repeat`, `timer`, które różnią się sposobem definicji, momentem, częstotliwością i zwracanym typem danych emisji.

//TODO example with Observable created with: create (custom type), just, from

## Uchwyt
W celu uniknięcia wycieków pamięci związanych z niepożądaną dłużej subskrypcją obserwatora należy przypisać referencje `Disposable` z poziomu obserwatora oraz wypisać go z subskrypcji.

//TODO example

W przypadku wielu obserwatorów manualne niszczenie subskrypcji może być żmudne i podatne błędy dlatego w takiej sytuacji warto użyć `CompositeDisposable`, który utrzymuje listę subskrypcji obiektów `DisposableObserver` w puli i potrafi zlikwidować je wszystkie naraz.

//TODO example

## Operatory
Operatory pozwalają na manipulacje i modyfikacje emitowanych danych za pomocą zadań transformacji, filtrowania, łączenia, agregacji czy tworzenia. RxJava dostarcza szeroki zbiór operatorów podzielonych na kategorie w zależności od rodzaju operacji, a ich łączenie umożliwia uzyskanie dowolnego złożonego strumienia danych. Poza operatorami odpowiedzialnymi za tworzenie obiektów obserwowanych (`create`, `just`, `from` itp) do często wykorzystywanych należą m.in. `filter`, `map`, `skip`, `take`, `concat`.

//TODO example

## Przykład

//TODO example