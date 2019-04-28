---
layout: post
title: "Coroutines"
date:  2019-07-22
categories: ["Kotlin"]
image: kotlin/coroutines
github: kotlin/blob/master/coroutines
description: "Kotlin"
version: Kotlin v1.3
keywords: "kotlin, coroutines, coroutine, scope, context, dispatcher, suspend, suspendig, async, withcontext, coroutinescope, runBlocking, launch, channel, actor, produce, android, programowanie, programming"
---

## Problem
Tradycyjny model tworzenia kodu asynchronicznego opartego o metody zwrotne (`Callback`) narażony jest na występowanie różnych trudności. W przypadku realizacji współbieżnych zadań zależnych może pojawić się problem komunikacji między zadaniami, który wymusza znalezienie sposobu na synchronizowanie rezultatów zwiększając tym samym złożoność kodu. Innym problemem jest zagnieżdżenie metod zwrotnych co z kolei sprawia, że w rzeczywistości zadania wykonywane są synchronicznie przez co wydłuża się czas ich przetwarzania. `Coroutines` (współprogramy) znacznie ułatwiają realizację zadań współbieżnych poprzez zmianę stylu pisania kodu asynchronicznego w sposób sekwencyjny. Dzięki takiemu podejściu kod jest bardziej czytelny i zrozumiały, a zarządzanie zadaniami staję się łatwiejsze. Ponadto jeden wątek potrafi obsługiwać jednocześnie wiele coroutines co przekłada się na znaczny wzrost wydajności. O `coroutine` można myśleć jako sekwencji podzadań wykonywanych wg określonej kolejności. 

## Funkcje zawieszania
Zasada działania coroutine oparta jest o ideę funkcji zawieszania (`suspend function`). Funkcja ta potrafi zawiesić swoje działanie do późniejszego wykonania bez blokowania wątku (np. w oczekiwaniu na zakończenie innej funkcji), gdzie koszt zawieszenia w stosunku do blokowania wątku juz dużo niższy. Funkcje wstrzymania muszą być wywołane wewnątrz coroutine lub w innej funkcji zawieszenia i mogą być uruchamiane na tym samym lub różnych wątkach. Aby zadeklarować funkcje zawieszenia należy użyć słowa kluczowego `suspend`.

{% highlight kotlin %}
suspend fun suspendingFunction() : Int  {
    //some task
    return 0
}
{% endhighlight %}

## Kontekst
Kontekst współprogramu (`CoroutineContext`) jest zbiorem zasad i konfiguracji, która definiuje sposób w jaki coroutine będzie wykonywany (może być kombinacją różnych kontekstów `CombinedContext`). Jednym z możliwych sposobów dostarczenia kontekstu jest użycie klasy `Dispatchers` zawierającej zbiór implementacji różniących się wykorzystaniem wątków. `Dispatchers.Default` może działać na wielu wątkach i używany jest do kosztownych zadań o dużym zapotrzebowaniu mocy obliczeniowej jak np. algorytmy. `Dispatchers.Main` działa na głównym wątku co w przypadku Android pozwala na modyfikację interfejsu użytkownika. `Dispatchers.IO` jest używany przede wszystkim do prostych operacji typu wejście/wyjście jak np. zapytanie sieciowe, dostęp do bazy danych, plików czy sensorów. `Dispatchers.Unconfined` nie ogranicza się do żadnego wątku w wyniku czego jego zachowanie jest trudne do przewidzenia. Kontekst może być przekazany jawnie jako argument lub uzyskiwany niejawnie na podstawie zakresu w którym jest wykonywany.

{% highlight kotlin %}
//withContext is suspend functions itself so no need to declare in inside suspend function if used inside coroutine
suspend fun suspendingWithContext() =
    //allows to change contex in coroutine for given block
    withContext(Dispatchers.Main) { 
        //some work
    }
{% endhighlight %}

## Budowniczy
Tworzenie i wykonanie coroutine może odbywać się przez jedną z funkcji budowniczego (`coroutine builder`) do której należą m.in.: `runBlocking`, `launch`, `async`. Funkcję te nie są funkcjami zawieszenia w związku z czym po wywołaniu kontynuowane jest działanie kolejnych instrukcji. Coroutines budowane są i działają w ramach struktury hierarchii (`structured concurrency`), tzn. rodzic (`parent coroutine`) ma zdolność do oddziaływania na cykl życia dzieci (`child coroutine`), a ich wykonanie przywiązane jest do danego zakresu (`scope`).  

`runBlocking` blokuje bieżący wątek dopóki wszystkie zadania w coroutine nie zostaną wykonane. Jest to przydatne w pisaniu testów wymagających wstrzymania.

{% highlight kotlin %}
//wait to finish this function to go further
fun testSuspendingWork() = runBlocking {
    val result = suspendingFunction()
    //wait here for result
    assertEquals(0, result)
}
{% endhighlight %}
  
`launch` jest często wykorzystywanym budowniczym, który w przeciwieństwie do `runBlocking` nie blokuje bieżącego wątku. Zwraca obiekt typu `Job` dzięki któremu możliwe jest manualne zarządzanie stanem zadań. Metoda `join` blokuje powiązany coroutine tak długo dopóki wszystkie jego zadania nie zostaną wykonane, natomiast `cancel` anuluje wszystkie zadania. Wykorzystywany do zadań typu `fire and forget` w których nie oczekuje się zwrócenia rezultatu.

{% highlight kotlin %}
//coroutines must be called on some scope so just use main scope called GlobalScope
suspend fun launchJobAndJoin() {
    //Job extends CoroutineContext to it's context itself
    val job = GlobalScope.launch(Dispatchers.Main) {
        //some work
        val result1 = suspendingWork1()
        //wait for result1
        val result2 = suspendingWork2()
        //wait for result2
        //process results
    }
	
    //wait here until suspendingWork tasks finished
    job.join() //suspending function itself
}

fun launchJobAndCancel() {
    val job = GlobalScope.launch(Dispatchers.Main) {
        //some work
        val result1 = suspendingWork1()
        val result2 = suspendingWork2()
        //process results
    }
	
    //cancel all the jobs, so if suspendingWork is running then cancel it and pending tasks
    job.cancel() //regular function so no need to run inside coroutine or suspend function
}
{% endhighlight %}
  
`async` podobnie jak `launch` pozwala na równoległe wykonanie zadań w tle, jednakże zwraca obiekt typu `Deferred`, który jest obietnicą przyszłego rezultatu. Metoda `await` w przypadku braku wyniku zawiesza dalsze wykonywanie instrukcji do momentu otrzymania rezultatu.

{% highlight kotlin %}
fun launchAsync() {
    val job = GlobalScope.launch(Dispatchers.Main) {
        val result = suspendingWork()
        val deferred1 = async {
            //some work
            return@async "result2"
        }
        val deferred2 = async {
            //some work
            return@async "result3"
        }

        //if in this place deferred1 and deferred2 not finished, wait for it
        val finalResult = "$result ${deferred1.await()} ${deferred2.await()}"
    }
    //run job.cancel() to cancel parent and all childs
}
{% endhighlight %}

## Zakres
Coroutines działają w ramach zakresu (`scope`), który stanowi dla nich przestrzeń wykonawczą i jest realizacją struktury hierarchii. Odwołanie się do zakresu wpływa na wszystkie znajdujące się w nim coroutines. Ich wykorzystanie eliminuje problem manualnego zarządzania stanem zadań, których praca i oczekiwanie na rezultat nierzadko ma sens tylko dla bieżącego ekranu (np. ładowanie danych do wyświetlenia). Zamiast ręcznego anulowania wszystkich coroutines wystarczy odwołać je poprzez zakres. Dowolna klasa implementująca `CoroutineScope` oraz nadpisująca właściwość `coroutineContext` może stać się zakresem. Warto zauważyć, że funkcje budowniczego są funkcjami rozszerzającymi `CoroutineScope`. Przykładem zakresu może być `GlobalScope` stanowiący ogólny zakres aplikacji.

{% highlight kotlin %}
class ScopeActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        //calling just launch { } is not possible
        //launch must be called on some CoroutineScope

        CoroutineScope(Dispatchers.Main).launch {
            //work
        }
		
        val job = GlobalScope.launch(Dispatchers.Main) { 
            //work
            async {
                //coroutine nested in coroutine
            }
        }
        //job.cancel() will cancel parent and childs
    }
}

//extend CoroutineScope
class ScopeClassActivity : AppCompatActivity(), CoroutineScope {
    
    //can be combined with another context like contatenation with Job
    override val coroutineContext: CoroutineContext
        get() = Dispatchers.Main

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        //now calling launch { } is possible because Activity is CoroutineScope itself
        launch { 
            //work
        }
    }
	
    override fun onDestroy() {
        super.onDestroy()
        cancel() //cancel on scope so all coroutines inside scope are cancelling
    }
}

//or provide CoroutineScope by delegate
class ScopeDelegateActivity : AppCompatActivity(), CoroutineScope by MainScope() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }
	
    //this class is CoroutineScope itself
}
{% endhighlight %}

## Kanały
Kanały są mechanizmem (podobnym do kolejki) pozwalającymi na przesyłanie i odbieranie potoku strumienia wartości między coroutines. W celu zbudowania kanału należy stworzyć instancję klasy `Channel`, która implementuje interfejsy zachowania zarówno nadawcy (`SendChannel`) jak i odbiorcy (`ReceiveChannel`). Opcjonalny parametr przekazany do metody wytwórczej odpowiada za wielkość bufora. Funkcja `send` i `receive` są funkcjami zawieszenia (zawieszają się w przypadku braku odbiorcy lub braku emisji) i umożliwiają odpowiednio emisję i odbiór wartości. Przetwarzanie wartości może także odbywać się poprzez iteracje kanału lub funkcje `consume`, `consumeEach`. 

{% highlight kotlin %}
fun runChannel() = runBlocking {
    //create channel using factory method
    //use one of RENDEZVOUS, UNLIMITED, CONFLATED optional param to specify buffer
    val channel = Channel<Int>(RENDEZVOUS)
	
    launch { 
        repeat(3) {
            //send some items from this coroutine
            channel.send(it)
            //suspend if no receivers
        } 
		
        //close channel to stop emission, this guarantees to send pending items
        channel.close()
    }
	
    //receive emitted values by receive function
    val value = channel.receive()

    //as alternative use channel's loop or consume by extension function of ReceiveChannel
    channel.consumeEach { 
        //do something with received values: 1, 2, 3
    }
	
    //emitted items can be consumed single time, so trying to receive again will no result
}
{% endhighlight %}

Kanały typu `Channel` ograniczone są do jednorazowego przepływu informacji, tzn. kanał może tylko raz odebrać wiadomości (w jednym miejscu). W sytuacji, gdy występuje wielu potencjalnych odbiorców należy użyć kanału typu `BroadcastChannel` lub `ConflatedBroadcastChannel`. Różnica między nimi polega na tym, że ten drugi informuje tylko o ostatniej wartości.

{% highlight kotlin %}
fun runBroadcastChannel() = runBlocking {
    val channel = BroadcastChannel<Int>(UNLIMITED)
	launch {
        repeat(3) { channel.send(it) } //doesn't supsend if no receivers
        channel.close()
    }
	
    //items can be consumed by multiple receivers
    channel.consumeEach {}
    channel.consumeEach {}	
}
{% endhighlight %}

Możliwe jest także tworzenie coroutine z automatycznie załączonym kanałem nastawionym na emisje lub odbiór za pomocą funkcji budowniczych producenta i aktora. `produce` uruchamia coroutine, który emituje strumień danych do kanału (jeden nadawca, wielu odbiorców). Zwraca `ReceiveChannel` oraz należy do zakresu `ProducerScope`.

{% highlight kotlin %}
//use it as some variable in real world
fun runProducer() = GlobalScope.launch {
    //produce is extension function of CoroutineScope
    val producer : ReceiveChannel<Int> = produce {
        repeat(3) { send(it) }
    }

    //use instance of ReceiveChannel to receive values emitted by produce
    producer.consumeEach {
        //do something
    }	
}
{% endhighlight %}

`actor` uruchamia coroutine, który odbiera wiadomości z kanału (jeden odbiorca, wielu nadawców). Zwraca `SendChannel` i należy do zakresu `ActorScope`.

{% highlight kotlin %}
//use it as some variable in real world
fun runActor() = GlobalScope.launch {	
    //actor is extension function of CoroutineScope
    val actor : SendChannel<Int> = actor {
        for(item in channel) {
            //do something
        }
    }

    repeat(3) { actor.send(it) }
}
{% endhighlight %}