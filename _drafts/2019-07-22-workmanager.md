---
layout: post
title: "WorkManager"
date: 2019-07-22
categories: ["Background"]
image: background/workmanager
github: background/tree/master/workmanager
description: "Background threading"
keywords: "workmanager, workrequest, worker, jobscheduler, jobdispatcher, jobservice, job, jobinfo, constraints, alarmmanager, broadcastreceiver, async, task, background, threading, deferrable, chain, enqueue, schedule, android, programowanie, programming"
---

## Charakterystyka
`WorkManager` dostarcza prosty interfejs umożliwiający zarządzanie zadaniami wykonywanymi w tle, których realizacja może być odroczona w czasie i ograniczona wymaganiami stanu systemu. Gwarantuje ukończenie pracy niezależnie od cyklu życia aplikacji czy systemu dzięki lokalnej bazie danych, która zarządza informacjami i statusami zadań z kolejki. Automatycznie wykrywa spełnienie warunków wstępnych stanu systemu dokonując rozpoczęcia, wstrzymania lub wznowienia zadania co pozwala na optymalne zarządzanie użycia baterii czy transmisji danych. Realizuje prace w tle stosując najlepsze praktyki oraz zachowuje zgodność z ograniczeniami dla różnych wersji systemu. Ponadto umożliwia zaplanowanie nie tylko zadań jednorazowanych, ale także cyklicznych oraz złożonych łańcuchów zadań (sekwencyjncyh i asynchronicznych). `WorkManager` nie jest całkowicie nową implementacją lecz wykorzystuje funckjonalność samodzielnych mechanizmów takich jak m.in. `JobScheduler`, `JobDispatcher`, `AlarmManager` czy `Executor`, których użycie zależne jest od wersji systemu, stanu cyklu życia aplikacji i czasu wykonania.

## Implementacja
Zadania są kolejkowane na instancji `WorkManager` poprzez przekazanie do metody `enqueue` obiektu typu `WorkRequest` odpowiedzialnego za stworzenie kompletnego zadania. W skład `WorkRequest` wchodzą m.in. zadanie (`Worker`), ograniczenia (`Constraints`), dane wejściowe czy kryteria wznowienia, powtarzalności i opóznienia pracy. Klasa `Worker` w metodzie `doWork` definiuje wykonywaną pracę oraz zwraca rezultat zadania.

{% highlight kotlin %}
class WorkManagerActivity : AppCompatActivity() {

    private lateinit var workId : UUID //store id in case to manage work

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_workmanager)

        button.setOnClickListener {
            startSimpleWork()
            observeWorkStatus()
        }
    }

    private fun startSimpleWork() {
        val workRequest = createWorkRequest()
        WorkManager.getInstance().enqueue(workRequest)
        workId = workRequest.id
    }

    private fun createWorkRequest() : WorkRequest {
        //create input data
        val inputData = workDataOf(Pair("INPUT_KEY", "some_input"))

        //create constraint conditions
        val constraint = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .setRequiresBatteryNotLow(true)
            .setRequiresStorageNotLow(true)
            .setRequiresCharging(true)
            .build()

        //init WorkRequest using WorkRequest class or some Builder for single or periodic work
        val workRequest = OneTimeWorkRequestBuilder<CustomWorker>()
            .setInputData(inputData)
            .setConstraints(constraint)
            .setBackoffCriteria(BackoffPolicy.LINEAR, 10, TimeUnit.SECONDS) //retry work conditions
            .setInitialDelay(3, TimeUnit.SECONDS)
            .build()

        return workRequest
    }
	
	private fun observeWorkStatus() {
        WorkManager.getInstance().getWorkInfoByIdLiveData(workId)
            .observe(this, Observer { workInfo ->
                //do some action based on workInfo state
        })
    }
}

class CustomWorker(context: Context, params: WorkerParameters) : Worker(context, params) {

    //do some custom work, e.g. upload or sync something
    override fun doWork(): Result {
        val input = inputData.getString("INPUT_KEY") //get the input
        //do some work
        val result = "some_work_result" //retrieve the result
        val output = workDataOf(Pair("OUTPUT_KEY", result)) //create the output

        //return the success, failure or retry result based on situations
        return Result.success(output)
    }
}
{% endhighlight %}

Jeśli zadanie składa się z mniejszych procesów, które powinny zostać wykonane sekwencyjne lub asynchronicznie, a rezultat kolejnego zależy od poprzedniego wówczas należy wykorzystać mechanizm łańcucha zadań. Metoda budowniczego `beginWith` pozwala na asynchroniczne wywolanie grupy zadań początkowych natomiast `then` na sekwencyjne wywołanie kolejnych kroków. Łączenie argumentów wejściowych z grupy zadań poprzedzających możliwe jest dzięki `InputMerger`.

{% highlight kotlin %}
class WorkManagerChainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_workmanager_chain)

        button.setOnClickListener {
            startChainedWork()
        }
    }

    private fun startChainedWork() {
        WorkManager.getInstance()
            .beginWith(Arrays.asList(createPartOneWorkRequest(), createPartTwoWorkRequest()))
            .then(createFinalWorkRequest())
            .enqueue()
    }

    private fun createPartOneWorkRequest() : OneTimeWorkRequest {
        val inputData = workDataOf(Pair("SIZE", "small"))
        val constraints = Constraints.Builder().setRequiredNetworkType(NetworkType.CONNECTED).build()

        return OneTimeWorkRequestBuilder<PartWorker>()
            .setInputData(inputData)
            .setConstraints(constraints)
            .build()
    }

    private fun createPartTwoWorkRequest() : OneTimeWorkRequest {
        val inputData = workDataOf(Pair("COLOR", "red"))
        val constraints = Constraints.Builder().setRequiresStorageNotLow(true).build()

        return OneTimeWorkRequestBuilder<PartWorker>()
            .setInputData(inputData)
            .setConstraints(constraints)
            .build()
    }

    private fun createFinalWorkRequest() : OneTimeWorkRequest {
        return OneTimeWorkRequestBuilder<FinalWorker>()
            .setInputMerger(ArrayCreatingInputMerger::class.java) //merge all values from every keys
            .setConstraints(Constraints.Builder().setRequiresBatteryNotLow(true).build())
            .build()
    }
}

class PartWorker(context: Context, params: WorkerParameters) : Worker(context, params) {
    
    override fun doWork(): Result {
        val color = inputData.getString("COLOR")
        val size = inputData.getString("SIZE")
        
        //do some work
        
        if(color != null) {
            val output = workDataOf(Pair("RESULT", "colored to $color"))
            return Result.success(output)
        }
        else if(size != null) {
            val output = workDataOf(Pair("RESULT", "resize to $size"))
            return Result.success(output)
        }
        else {
            return Result.failure()
        }
    }
}

class FinalWorker(context: Context, params: WorkerParameters) : Worker(context, params) {

    override fun doWork(): Result {
        //get merged input from pre workers
        //it contains all values in this key from every pre Worker
        val input = inputData.getString("RESULT") //so it should be: colored to red, resize to small
        //do something with input and return result
        return Result.success()
    }
}
{% endhighlight %}

## JobScheduler
`JobScheduler` pozwala na planowanie zadań w tle gwarantując ich ukończenie w nieokreślonej przyszłości. Wykonanie zadania jest uzależnione od warunków stanu systemu. Może się zdarzyć, że rozpoczęcie pracy będzie natychmiastowe lub pozostanie w oczekiwaniu na korzystny stan systemu umożliwiający podjęcie działania zgodnie z wymaganiami wstępnymi. Optymalizuje użycie baterii i pamięci w stosunku do innych kosztownych rozwiązań realizacji pracy w tle takich jak np. `Service`, `AlarmManager` czy `BroadcastReceiver`. Usługa `JobScheduler` jest jednak dostępna od wersji systemu `Android L`. Alernatywnym rozwiązaniem może być wykorzystanie `FirebaseJobDispatcher` (działa także poniżej `Android L`). Obie usługi działają w oparciu o `JobService`, którego zadaniem jest obsługa zdarzeń rozpoczęcia i zatrzymania pracy. Warunki wstępne deklarowane są w obiektach `JobInfo` lub `Job`, a zaplanowanie zadania odbywa się przy pomocy metody `schedule`.

{% highlight kotlin %}
class JobScheduler : AppCompatActivity() {

    lateinit var jobScheduler: JobScheduler //works min for API 21
    lateinit var jobDispatcher: FirebaseJobDispatcher //alernative for lower API

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_jobscheduler)

        jobScheduler = getSystemService(Context.JOB_SCHEDULER_SERVICE) as JobScheduler
        jobDispatcher = FirebaseJobDispatcher(GooglePlayDriver(this))

        button.setOnClickListener {
            //call schedule method from JobScheduler or JobDispatcher
            val resultCode = jobScheduler.schedule(createJobInfo())
            if(resultCode == JobScheduler.RESULT_SUCCESS) {
                //job is scheduled
            }
        }
		
        //call cancel method from JobScheduler or JobDispatcher to cancel pending Job on some event
    }

    //customized and schedule this JobInfo to JobScheduler
    private fun createJobInfo() : JobInfo {
        val componentName = ComponentName(this, CustomJobService::class.java)
        return JobInfo.Builder(1, componentName)
            .setMinimumLatency(1000)
            .setOverrideDeadline(3000)
            .setRequiresCharging(true)
            .build()
    }
	
	//customized and schedule this Job for JobDispatcher
	private fun createJob() : Job {
        return dispatcher.newJobBuilder()
            .setService(CustomJobService::class.java)
            .setConstraints(Constraint.DEVICE_CHARGING)
            .setTrigger(Trigger.executionWindow(1, 3))
            .setTag("TAG")
            .build()
    }
}

class CustomJobService : JobService() {

    //note that JobService is running on main thread like standard Service
    private val uiHandler = Handler(Looper.getMainLooper())
    private val executor = Executors.newSingleThreadExecutor()

    override fun onStartJob(params: JobParameters?): Boolean {
        //reached when job starts running
        executor.execute {
            //do some background job
            jobFinished(params, false) //call to inform that job is finished
            //post some result on UI if needed
        }
        return false //is work still in progress?
    }

    override fun onStopJob(params: JobParameters?): Boolean {
        //reached when job has stopped - manual or auto
        jobFinished(params, false)
        return false //should work be retried?
    }
}
{% endhighlight %}

## AlarmManager
Zadaniem `AlarmManager` jest zapewnienie dostępu do usług alarmowych systemu, dzięki czemu możliwe jest zaplanowanie uruchomienia zadania w wybranym momencie w przyszłości. Gdy alarm się włącza wówczas przechwytywana jest zarejestrowana intencja (`PendintIntent`) i delegowana do miejsca docelowego, gdzie jest wykonywana (np. przez BroadcastReceiver). Emisja alarmu może być jednorazowa lub cykliczna i działa niezależnie od cyklu życia aplikacji.

{% highlight kotlin %}
class AlarmManagerActivity : AppCompatActivity() {

    val alarmManager : AlarmManager by lazy { getSystemService(ALARM_SERVICE) as AlarmManager }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_alarmmanager)

        button.setOnClickListener {
            startAlarm()
        }
    }

    private fun startAlarm() {
        //prepare Intent and PendingIntent (e.g. send alarm to BroadcastReceiver)
        val intent = Intent(this, AlarmBroadcastReceiver::class.java)
        val pendingIntent = PendingIntent.getBroadcast(this, 100, intent, 0)

        //schedule alarm with specific trigger time and PendingIntent
        //use specific method based on OS version in addition 
        val time = System.currentTimeMillis() + 3000
        val type = RTC_WAKEUP
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M)
            alarmManager.setExactAndAllowWhileIdle(type, time, pendingIntent)
        else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT)
            alarmManager.setExact(type, time, pendingIntent)
        else
            alarmManager.set(type, time, pendingIntent)
    }
}

class AlarmBroadcastReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        //do some action when Broadcast receive Intent from AlarmManager
    }
}
{% endhighlight %}