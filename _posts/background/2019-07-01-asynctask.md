---
layout: post
title: "AsyncTask"
date: 2019-07-01
categories: ["Background"]
image: background/asynctask
github: background/tree/master/asynctask
description: "Background threading"
keywords: "asynctask, async, task, background, threading, doinbackground, onpreexecute, onprogressupdate, onpostexecuted, execute, weakreference, android, programowanie, programming"
---

## Wstęp
`AsyncTask` umożliwia wykonanie zadania w tle i publikowanie stanu pracy w interfejsie użytkownika bez konieczności manipulowania wątkami. Jest zaprojektowany jako klasa pomocnicza wokół klasy `Thread` i `Handler`. Wykorzystywany przede wszystkim do krótkich jednorazowych operacji powiązanych z bieżącym ekranem, które wymagają przetworzenia rezultatu na wątku głównym aplikacji. Dostarcza prosty w obsłudze interfejs co zwalnia programistę z obowiązku zajmowania się obsługą wątków (roboczego i głównego).

## Implementacja
Aby stworzyć zadanie asynchroniczne należy rozszerzyć klasę `AsyncTask` oraz nadpisać przynajmniej metodę `doInBackground` odpowiedzialną za wykonywanie zadania na wątku roboczym. Pozostałe metody `onPreExecute`, `onProgressUpdate`, `onPostExecute`, `onCancelled` są wykonywane na głównym wątku dla różnych stanów zadania. Częstą praktyką jest implementacja klasy `AsyncTask` jako klasy wewnętrznej (`inner class`) w kontekście wywołania. Rozpoczęcie wykonywania zadania odbywa się przy pomocy metody `execute`.

{% highlight kotlin %}
//first of generic types are params, second progress and third is final result
private class CustomAsyncTask : AsyncTask<String, Int, String>() {

    override fun onPreExecute() {
        //prepare before running background job like show progress dialog
    }

    override fun doInBackground(vararg params: String): String {
        //some background job with passed params like URLs
        var total = ""
        for((counter, url) in params.withIndex()) {
            //download info from url
            val result = "result from : $url\n"
            total = total.plus(url)
            publishProgress(counter) //inform that some part of full request if completed
        }
        return total
    }

    override fun onProgressUpdate(vararg values: Int?) {
        //show some progress updates based on Int from publishProgress
    }

    override fun onPostExecute(result: String) {
        //do something on main thread when background job finished like update view
    }

    override fun onCancelled() {
        //stop and clear or show some message
    }
}

//to start just call
//CustomAsyncTask().execute("url1", "url2", "url3")
{% endhighlight %}

## Ograniczenia
Implementacja klasy `AsyncTask` jako klasy wewnętrznej może powodować wycieki pamięci (`leak memmory`). Instancja takiej klasy przechowuje referencje do klasy zewnętrznej (np. `Activity`, `Fragment`) niezależnie od stanu cyklu życia obiektu klasy zewnętrznej co powstrzymuje `Garbage Collector` przed usunięciem referencji. Aby temu zapobiec należy stworzyć klasę jako statyczną wewnętrzną (`static inner`), zagnieżdżoną (`nested`) lub na poziomie `top-level`. W przypadku wymaganej referencji do `Activity` warto posłużyć się słabą referencją `WeakReference` lub przekazać obiekt `Listener` implementujący oczekiwane zachowanie interfejsu użytkownika.

{% highlight kotlin %}
class MainActivity : AppCompatActivity(), Listener {
    
    //reference to task allows to manage it's lifecycle
    private var asyncTask : SaferAsyncTask? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
		
        button.setOnClickListener {
            //AsyncTask is single shot, so create new task every time
            if(asyncTask == null || asyncTask?.status != AsyncTask.Status.RUNNING) {
                asyncTask = SaferAsyncTask(this)
                asyncTask?.execute("url1", "url2", "url3")
            }
        }
    }

    override fun onDestroy() {
        //stop the task when exit from Activity
        if(asyncTask?.status == AsyncTask.Status.RUNNING)
            asyncTask?.cancel(true)
        super.onDestroy()
    }

    override fun onStarting() {
        //create progress dialog
    }

    override fun onProgress(progress: Int) {
        //update progress dialog
    }

    override fun onFinished(result: String) {
        //close progress dialog
        //show the result
    }

    override fun onCancel() {
        //close progress dialog
        //show error message
    }

    private class SaferAsyncTask(listener : Listener) : AsyncTask<String, Int, String>() {

        private val reference = WeakReference<Listener>(listener)

        override fun onPreExecute() {
            reference.get()?.onStarting()
        }

        override fun doInBackground(vararg params: String): String {
            var total = ""
            for((counter, url) in params.withIndex()) {
                val result = "result from : $url\n"
                total = total.plus(result)
                publishProgress(counter) //inform that some part of full request if completed
            }
            return total
        }

        override fun onProgressUpdate(vararg values: Int?) {
            values[0]?.let { reference.get()?.onProgress(it) }
        }

        override fun onPostExecute(result: String) {
            reference.get()?.onFinished(result)
        }

        override fun onCancelled() {
            reference.get()?.onCancel()
        }
    }
}

private interface Listener {
    fun onStarting()
    fun onProgress(progress : Int)
    fun onFinished(result : String)
    fun onCancel()
}
{% endhighlight %}

Ponadto należy również zwrócić uwagę na fakt iż tak skonstruowane zadanie nie przetrwa zmiany konfiguracji co będzie skutkowało utratą aktualnego progresu. Aby temu zapobiec można umieścić obiekt `AsyncTask` w obiekcie `Fragment` z ustawionym `setRetainInstance(true)` lub w `ViewModel`. Co więcej, `AsyncTask` jest zadaniem jednorazowym i każde jego ponowne wywołanie wymaga nowej instancji w związku z czym warto zadbać o odpowiednią obsługę stanu zadania w zależności od cyklu życia. Ogranicza również komunikację z innymi zadaniami współbieżnymi wykonywanymi na innych wątkach.