---
layout: post
title: "RxJava"
date: 2019-05-27
categories: ["Biblioteki"]
image: libraries/rxjava
github: libraries/tree/master/rxjava
description: "Biblioteki"
version: RxJava 2.2, RxAndroid 2.1
keywords: "reaktywne, reactive, rxandroid, rxjava, reactivex, obserwator, obserwowany, observer, observable, schedulers, operator, disposable, subscribe, async, thread, java, android, programowanie, programming"
---

## Programowanie reaktywne
`Programowanie reaktywne` (`reactive programming`) jest paradygmatem programowania asynchronicznego zorientowanym na przepływ danych w postaci strumieni i propagacje zmian w formie zdarzeń. Strumieniem danych może być prawie wszystko, np. zmiana wartości obiektu, zdarzenia interfejsu użytkownika, zapytania sieciowe, operacje na danych czy błędy, a każde zadanie może być wykonywane na własnym wątku. Sposób działania opiera się na `wzorcu Obserwator`, tzn. strumienie mogą emitować wartości i być obserwowane. Realizacją programowania reaktywnego dla `Android` jest moduł `RxAndroid` rozszerzający bibliotekę `RxJava` (implementacja `ReactiveX` dla `Java VM`) o harmonogramy (`Schedulers`) wspierające wielowątkowość w środowisku Android. `RxJava` upraszcza zarządzanie asynchronicznymi i współbieżnymi operacjami, ułatwia komunikacje zadań w tle z wątkiem głównym, pozwala na wczesne wykrycie błędów i zmniejsza zapotrzebowanie na zmienne stanu.

## Wzorzec
Zadaniem wzorca `Obserwator` jest realizacja mechanizmu komunikacji między obiektami zależnymi poprzez powiadamianie obiektów subskrybentów o zmianie stanu obiektu obserwowanego czy statusie operacji. Obiekt może być obserwowany (`Observable`) przez wielu obserwatorów (`Observer`) i sam może być obserwatorem innych obiektów. Obiekt obserwowany emituje zdarzenia, które trafiają do obserwatorów, a następnie są przez nich przetwarzane. 

## Obserwator
Obiekt obserwatora dokonuje subskrybcji obiektu obserwowanego za pomocą metody `subscribe` wywołanej na obiekcie obserwowanym oraz implementuje metody `onSubscribe`, `onNext`, `onError`, `onComplete` interfejsu `Observer` w celu podjęcia działania w odpowiedzi na stan i emisje obiektu obserwowanego.

{% highlight java %}
private Observer<String> createObserver() {
    Observer<String> observer = new Observer<String>() {
        @Override
        public void onSubscribe(Disposable d) {
            //Observer subscribe to Observable
        }

        @Override
        public void onNext(String s) {
            //Observer received some data stream
        }

        @Override
        public void onError(Throwable e) {
            //Observer received emitted error
        }

        @Override
        public void onComplete() {
            //Observer completed receiving data from Observable
        }
    };
    return observer;
}
{% endhighlight %}

## Obserwowany
Obiekt obserwowany w `RxJava` jest strumieniem danych wykonującym operacje i emitującym dane. Podstawowym typem obiektu obserwowanego jest `Observable` - emituje dowolną ilość elementów i kończy się sukcesem lub zdarzeniem błędu. Wyróżnia się również także: `Flowable` - rozszerza `Observable` o wsparcie dla `backpressure` (zarządzanie czasem emitowania i konsumowania zdarzenia), `Single` - emituje jedno zdarzenie lub błąd, `Maybe` - może emitować jedno zdarzenie lub błąd, `Completable` - nie emituje zdarzenia, kończy się sukcesem lub błędem. Obiekt obserwowany może być tworzony za pomocą kilku operatorów jak np.: `create`, `just`, `defer`, `from`, `interval`, `range`, `repeat`, `timer`, które różnią się sposobem definicji, czasem, częstotliwością i zwracanym typem danych emisji.

{% highlight java %}
private Observable<String> createObservableJust() {
    //up to 10 items
    Observable<String> observable = Observable.just("Catan", "Splendor", "7 Wonders", "Codenames", "Taboo");
    return observable;
}

private Observable<String> createObservableFrom() {
    //create Observable by one of: fromArray, fromCallable, fromFuture, fromIterable, fromPublisher
    Observable<String> observable = Observable.fromArray("Catan", "Splendor", "7 Wonders", "Codenames", "Taboo");
    return observable;
}

private Observable<String> createObservable() {
    final List<String> data = prepareData();
    Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(ObservableEmitter<String> emitter) throws Exception {
            //emit all data one by one
            for(String item : data) {
                if(!emitter.isDisposed()) {
                    emitter.onNext(item);
                }
            }
            //emit completed event
            if(!emitter.isDisposed()) {
                emitter.onComplete();
            }
        }
    });
    return observable;
}

private List<String> prepareData() {
    List<String> data = new ArrayList<>();
    data.add("Catan");
    data.add("Splendor");
    data.add("7 Wonders");
    data.add("Codenames");
    data.add("Taboo");
    return data;
}
{% endhighlight %}

## Harmonogramy
Harmonogramy decydują o tym na jakim wątku obiekt obserwowany będzie wykonywał zadanie oraz na jakim wątku obiekt obserwatora będzie odbierał i przetwarzał wyemitowane dane. Najpopularniejszymi harmonogrami są: `AndroidSchedulers.mainThread` - odpowiada za główny wątek aplikacji (wprowadzony w module `RxAndroid`), `Schedulers.io` - wykorzystywany w nie wymagających operacjach i `Schedulers.computation` - przeznaczony do przetwarzania intensywnych zadań. Wyróżnia się także `Schedulers.newThread`, `Schedulers.single`, `Schedulers.immediate`, `Schedulers.trampoline`, `Schedulers.from`, które różnią się synchronicznością, kolejnością, czasem rozpoczęcia i ilością przetwarzanych zadań. Przypisanie harmonogramu dokonywane jest na obiekcie obserwowanym za pomocą metod `subscribeOn` i `observeOn`.

{% highlight java %}
private void startWorkAndSubscribe() {
    //create Observable and Observer
    Observable<String> observable = createObservable();
    Observer<String> observer = createObserver();

    //subscribe on choosen schedulers
    observable
        .observeOn(Schedulers.io()) //emit data on background thread
        .subscribeOn(AndroidSchedulers.mainThread()) //receive data on UI thread
        .subscribe(observer); //subscribe observer to observable
}
{% endhighlight %}		

## Operatory
Operatory pozwalają na manipulacje i modyfikacje emitowanych danych za pomocą zadań transformacji, filtrowania, łączenia, agregacji czy tworzenia. RxJava dostarcza szeroki zbiór operatorów podzielonych na kategorie w zależności od rodzaju operacji, a ich łączenie umożliwia uzyskanie dowolnego złożonego strumienia danych. Poza operatorami odpowiedzialnymi za tworzenie obiektów obserwowanych (`create`, `just`, `from` itp) do często wykorzystywanych należą m.in. `filter`, `map`, `skip`, `take`, `concat`.

{% highlight java %}
private void startWorkAndSubscribeWithOperators() {
    Observable<String> observable = createObservable();
    Observer<Object> observer = createObserver();

    observable
        .observeOn(Schedulers.io())
        .subscribeOn(AndroidSchedulers.mainThread())
        .filter(new Predicate<String>() { //emit only some elements
            @Override
            public boolean test(String s) throws Exception {
                return s.toLowerCase().startsWith("c");
            }
        })
        .map(new Function<String, Object>() { //modify data
            @Override
            public Object apply(String s) {
                return s.toUpperCase();
            }
        })
        .subscribe(observer);
}
{% endhighlight %}

## Uchwyt
W celu uniknięcia wycieków pamięci związanych z niepożądaną dłużej subskrypcją obserwatora należy przypisać referencje `Disposable` z poziomu obserwatora oraz wypisać go z subskrypcji.

{% highlight java %}
public class SingleObserverActivity extends AppCompatActivity {
    
    private Disposable disposable;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_single_observer);

        Observable<String> observable = createObservableWithDispose();
        Observer<String> observer = createObserver();

        observable
            .observeOn(Schedulers.io())
            .subscribeOn(AndroidSchedulers.mainThread())
            .subscribe(observer);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        disposable.dispose(); //don't send event anymore
    }


    private Observer<String> createObserverWithDispose() {
        Observer<String> observer = new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                //assign subscription to dispoable
                disposable = d;
            }
            @Override
            public void onNext(String s) { }
            @Override
            public void onError(Throwable e) { }
            @Override
            public void onComplete() { }
        };
        return observer;
    }
	
    private Observable<String> createObservableJust() {
        Observable<String> observable = Observable.just("Catan", "Splendor", "7 Wonders", "Codenames", "Taboo");
        return observable;
    }
}
{% endhighlight %}	

W przypadku wielu obserwatorów manualne niszczenie subskrypcji może być żmudne i podatne błędy dlatego w takiej sytuacji warto użyć `CompositeDisposable`, który utrzymuje listę subskrypcji obiektów `DisposableObserver` w puli i potrafi zlikwidować je wszystkie naraz.

{% highlight java %}
public class MultipleObserversActivity extends AppCompatActivity {

    private CompositeDisposable compositeDisposable = new CompositeDisposable();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_multiple_observers);
		
        Observable<String> observable = createObservable();
        DisposableObserver<String> observerGames = createDisposableObserver();
        DisposableObserver<String> observerFilteredGames = createDisposableObserver();

        compositeDisposable.add(observable
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribeWith(observerGames));

        compositeDisposable.add(observable
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .filter(new Predicate<String>() {
                @Override
                public boolean test(String s) throws Exception {
                    return s.toLowerCase().startsWith("c");
                }
            })
            .subscribeWith(observerFilteredGames));
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        compositeDisposable.clear(); //clear all disposable
    }
	
    private Observable<String> createObservableJust() {
        Observable<String> observable = Observable.just("Catan", "Splendor", "7 Wonders", "Codenames", "Taboo");
        return observable;
    }

    private DisposableObserver<String> createDisposableObserver() {
        DisposableObserver<String> observer = new DisposableObserver<String>() {
            @Override
            public void onNext(String s) { }
            @Override
            public void onError(Throwable e) { }
            @Override
            public void onComplete() { }
        };
        return observer;
    }	
}
{% endhighlight %}	
