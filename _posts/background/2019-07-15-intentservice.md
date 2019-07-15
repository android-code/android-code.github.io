---
layout: post
title: "IntentService"
date: 2019-07-15
categories: ["Background"]
image: background/intentservice
github: background/tree/master/intentservice
description: "Background threading"
keywords: "intentservice, jobintentservice, broadcastreceiver, async, task, background, onhandleintent, threading, android, programowanie, programming"
---

## Wstęp
`IntentService` jest rozszerzeniem klasy `Service` o obsługę zadań przetwarzanych na w tle na dedykowanym wątku roboczym przy pomocy klasy `Handler`. Umożliwia wykonanie pracy w sposób asynchroniczny niezależnie od wątku głównego, jednakże obsługuje tylko jedno zadanie w danej chwili. Wysyłane żądania w postaci obiektu `Intent` zostają natychmiast obsłużone lub trafiają do kolejki oczekujących. Gdy cała kolejka zadań zostanie zrealizowana wówczas usługa automatycznie kończy pracę. Większość zdarzeń cyklu życia interfejsu użytkownika nie ma wpływu na działanie `IntentService` (w przeciwieństwie do `AsyncTask`). Przeznaczony jest przede wszystkim do prostych operacji w tle, które niekoniecznie wymagają reakcji interfejsu użytkownika na otrzymany rezultat.

## Implementacja
Aby stworzyć komponent `IntentService` należy zdefiniować klasę rozszerzającą klasę `IntentService`, nadpisać metodę `onHandleIntent` odpowiedzialną za przechwytywanie żądań i podobnie jak przy klasycznej usłudze (`Service`) dodać do pliku manifestu (`AndroidManifest`).

{% highlight kotlin %}
class CustomIntentService : IntentService("name") {

    override fun onHandleIntent(intent: Intent?) {
        //retrieve data
        val action = intent?.action
        val data = intent?.dataString
        //do some work based on action
    }

    //avoid to override Service's callback like onStartCommand
    //they are automatically invoked by IntentService
}

class CustomActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_custom)
        button.setOnClickListener { startIntentService() }
    }

	//just call startService to run IntentService
    private fun startIntentService() {
        val intent = Intent(this, CustomIntentService::class.java)
        intent.action = "action"
        intent.putExtra("extra", "value")
        startService(intent)
    }
}
{% endhighlight %}

## Komunikacja
`IntentService` nie dostarcza mechanizmu odpowiedzi zwrotnej do klientów. W celu implementacji komunikacji do interfejsu użytkownika lub innych fragmentów kodu aplikacji można posłużyć się m.in. `BroadcastReceiver` lub szyną zdarzeń.

{% highlight kotlin %}
class IntentServiceCommunication : IntentService("name") {

    override fun onHandleIntent(intent: Intent?) {
        //retrieve data
        val action = intent?.action
        val data = intent?.getStringExtra("extra")
        //do some work based on action

        sendBroadcast()
    }

    private fun sendBroadcast() {
        val broadcastIntent = Intent("FILTER_ACTION")
        broadcastIntent.putExtra("message", "result")
        sendBroadcast(broadcastIntent)
        Log.d("IntentService", "sendBroadcast")
    }
}

class CommunicationActivity : AppCompatActivity() {

    private val receiver = CustomBroadcastReceiver()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_communication)
        button.setOnClickListener { startIntentService() }

        registerReceiver(receiver, IntentFilter("FILTER_ACTION"))
    }

    override fun onDestroy() {
        //if service is running and don't need anymore callback then stop IntentService
        unregisterReceiver(receiver)
        super.onDestroy()
    }

    fun startIntentService() {
        val intent = Intent(this, CustomIntentService::class.java)
        intent.action = "action"
        intent.putExtra("extra", "value")
        startService(intent)
    }

    //implement as nested class if only need to interact with this activity
    class CustomBroadcastReceiver : BroadcastReceiver() {

        override fun onReceive(context: Context?, intent: Intent?) {
            val message = intent?.getStringExtra("message")
            //do something like update UI
        }
    }
}
{% endhighlight %}

## Ograniczenia
Poza koniecznością zapewnienia mechanizmu odpowiedzi zwrotnej do klienta oraz ograniczeniem przepływu pracy do jednego wątku należy uwzględnić także restrykcje procesów w tle dla `Service` (w tym `IntentService`) w wyniku których usługa musi działać w trybie `foreground` (inaczej zostanie po pewnym czasie zatrzymana). Rozwiązaniem tego problemu może być użycie `JobIntentService` lub wystartowanie usługi w trybie foreground (`startForegroundService`) i wyświetlenie notyfikacji z poziomu usługi (`startForeground`).

## JobIntentService
`JobIntentService` w zależności od wersji systemu dobiera optymalną strategię wykonywania kolejki zadań w tle. W przypadku wersji systemu od `Android Oreo` zadania zostaną wykonane przy użyciu `JobScheduler`, natomiast dla starszych wersji usługa zostanie wywołana w standardowy sposób przy pomocy metody `startService`. Klasa rozszerzająca `JobIntentService` powinna nadpisać metodę `onHandleWork` w której będzie wykonywana praca (podobnie jak w `onHandleIntent` w `IntentService`). Klient dodaje zadanie do kolejki wywołując `enqueueWork`.

{% highlight kotlin %}
class CustomJobIntentService : JobIntentService() {

    companion object {
        const val ID = 100

        //method for enqueuing work to this service, just call from client to start
        fun enqueueWork(context : Context, work : Intent) {
            enqueueWork(context, CustomJobIntentService::class.java, ID, work)
        }
    }

    override fun onHandleWork(intent: Intent) {
        //do some job here
        //use BroadcastReceiver or something else to post the result
    }
}
{% endhighlight %}

{% highlight xml %}
<!-- remember to add Service to AndroidManifest in the same way like base JobService 
//with required BINB_JOB_SERVICE permission -->
<service
    android:name=".CustomJobIntentService"
    android:permission="android.permission.BIND_JOB_SERVICE"/>
{% endhighlight %}