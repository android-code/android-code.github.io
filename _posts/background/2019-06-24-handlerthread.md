---
layout: post
title: "HandlerThread"
date: 2019-06-24
categories: ["Background"]
image: background/handlerthread
github: background/tree/master/handlerthread
description: "Background threading"
keywords: "thread, handler, handlerthread, looper, threadpool, executor, runnable, callable, future, async, task, background, threading, android, programowanie, programming"
---

## Thread
Klasa `Thread` w `Java` reprezentuje wątek wykonania programu i jest podstawą wielu implementacji mechanizmów wielowątkowości. Pozwala na wykonanie pracy na innym wątku. W czystej implementacji bez wsparcia innych klas może służyć do wykonywania długich zadań bez komunikacji z wątkiem głównym. Wątek działa niezależnie od cyklu zycia aplikacji, dlatego potrafia przetrwać zmianę konfiguracji czy wyjście z aplikacji (trafia do póli wątków). Zatrzymanie działania następuje automatycznie po zakończeniu pracy lub w sposób manualny (metodą `interrupt`). Może być tylko raz wykonany zatem każde uruchomienie oczekiwanej pracy wymaga nowej instancji co sprawia, że nie nadaje sie do ponownego użytku. Komunikacja z wątkiem głównym może przebiegać przy użyciu klasy `Handler`.

{% highlight kotlin %}
class ThreadActivity : AppCompatActivity() {

    val runnable = Runnable {
        //some background job
        runOnUiThread { 
            //UI job if needed
        }
    }
    val thread = Thread(runnable) //pass runnable or create anonymous object

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_thread)

        button.setOnClickListener {
            //do not allow start thread if is started
            if(thread.state == Thread.State.NEW)
                thread.start()
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        if(thread.state != Thread.State.TERMINATED)
            thread.interrupt() //stop the thread if needed
    }
}
{% endhighlight %}

## Looper
Mówiąc o wątkach w Android nie sposób nie wspomnieć o klasie `Looper`, która używana jest do uruchamiania pętli wiadomości dla wątku. Wspiera wykonanie pracy wątku przez dostarczanie wybranych wiadomości i zadań z kolejki do odpowiednich obiektów klasy `Handler`. Każdy wątek `Thread` może mieć jedną uniknalną instancje `Looper`.

## Handler
`Handler` umożliwia komunikacje między wątkami poprzez wykorzystanie kanału wiadomości. Jest pośrednio powiązany z wątkiem `Thread` poprzez bezpośrednie przekazanie obiektu `Looper` danego wątku w konstruktorze (domyślnie jest to `Looper` wątku w którym `Handler` jest tworzony). Działa więc na wątku, który jest właścicielem dostarczonej instancji `Looper`. Wiadomość `Message` może być wysyłana z dowolnego miejsca przez `sendMessage` oraz zostać odebrana przez `handleMessage` na wątku obiektu `Handler`. Natomiast zadanie `Runnable` wywoływane jest metodą post, a jego wykonanie odbywa się na wątku obiektu `Handler`.

{% highlight kotlin %}
class HandlerActivity : AppCompatActivity() {

    val START = 1
    val FINISH = 2

    val runnable = Runnable {
        //some background job
        handler.sendMessage(createMessage(FINISH, "result")) //send message to handle action
        handler.post { 
            //do action on UI without message communication
        }
    }

    val thread: Thread = Thread(runnable)

    val handler = object: Handler(Looper.getMainLooper()) { //pass Looper of thread owner
        override fun handleMessage(msg: Message?) {
            super.handleMessage(msg)
            if(msg?.what == START && thread.state == Thread.State.NEW) {
                thread.start()
            }
            else if(msg?.what == FINISH) {
                //some action with msj.obj result
            }
        }
    }

    fun createMessage(what : Int, obj : Any) : Message {
        val message = Message()
        message.what = what
        message.obj = obj
        return message
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_handler)

        button.setOnClickListener {
            //instead of direct thread.start() it can be done by sending message
            handler.sendEmptyMessage(START)
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        handler.removeCallbacksAndMessages(null) //remove pending work from MessageQueue
        thread.interrupt()
    }	
}
{% endhighlight %}

## HandlerThread
`HandlerThread` jest rozbudowaną klasą `Thread`, która posiada własną instancję `Looper` i `Handler` dzięki czemu potrafi kolejkować zadania i może być użyty wielokrotnie. Zadania wykonywane są synchronicznie w sposób sekwencyjny. Obiekt `HandlerThread` podobnie jak Thread jest aktywny dopóki nie zostanie zniszczony automatycznie przez system lub ręcznie.

{% highlight kotlin %}
class HandlerThreadActivity : AppCompatActivity() {

    //work to do by requestHandler
    val runnable = Runnable {
        //some background job
        responseHandler.sendMessage(Message()) //send message or post the job
        responseHandler.post { 
            //some UI action
        }
    }

    //handler on the main thread, override handleMessage if needed
    val responseHandler = object: Handler(Looper.getMainLooper()) {
        override fun handleMessage(msg: Message?) {
            super.handleMessage(msg)
            //some action on UI
        }
    }

    //handler on the background thread of handlerThread object
    lateinit var requestHandler: Handler

    //instead of Thread use reusable HandlerThread
    //this can be also done by extend HandlerThread class and putting there Handler object
    val handlerThread = HandlerThread("HandlerThread")
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_handler_thread)

        handlerThread.start() //start the thread
        requestHandler = Handler(handlerThread.looper) //now Looper of handlerThread can be passed

        button.setOnClickListener {
            requestHandler.post(runnable) //post the job to MessageQueue
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        //remove tasks and messages from handlers
        requestHandler.removeCallbacksAndMessages(null)
        responseHandler.removeCallbacksAndMessages(null)
        handlerThread.quitSafely() //quit HandlerThread's Looper
        handlerThread.interrupt() //stop current job
    }
}
{% endhighlight %}

## Pula wątków
Pula wątków (`ThreadPoolExecutor`) jest pojedynczą kolejką typu `FIFO` składającą się z grupy wątków roboczych, które w momencie osiągnięcia stanu dostępności pobierają zadanie z kolejki. Umożliwia asynchroniczne wykonanie zadań przy zachowaniu optymalnej strategii przetwarzania wielowątkowoego, dostarcza mechanizm zarządzania wątkami (dodawanie, anulowanie, priorytety) oraz prostą statystykę. Pula wątków może być użyta przez jedną z implementacji interfejsu `Executor`, `Callable` czy `Future`. Przeważnie instancje uzyskuje się przy pomocy metody wytwórczej z klasy `Executors` oferującej kilka rodzajów typu puli watków.

{% highlight kotlin %}
class ThreadPoolActivity : AppCompatActivity() {

    //this ExecutorService is wrapped ThreadPoolExecutor (this class extends ExecutorService)
    //equivalent of ThreadPoolExecutor with min and max only 1 thread in pool and LinkedBlockingQueue
    val executor : ExecutorService = Executors.newSingleThreadExecutor()

    val runnable = Runnable {
        //some background job
        uiHandler.post {
            //some UI job
        }
    }

    val callable = Callable {
        //some background job and return result
        return@Callable "result"
    }

    val uiHandler = Handler()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_thread_pool)

        button1.setOnClickListener {
            //use executor with Runnable        
            executor.execute(runnable)
        }

        button2.setOnClickListener {
            //use executor with Callable        
            val future : Future<String> = executor.submit(callable)
            val result = future.get()
            //this can be canceled by run future.cancel()
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        //remove tasks and messages from handlers
        uiHandler.removeCallbacksAndMessages(null)
        executor.shutdownNow() //kill all threads
    }
}
{% endhighlight %}