---
layout: post
title: "Pełnomocnik"
date:  2018-07-30
categories: ["Wzorce projektowe"]
image: patterns/proxy
github: design-patterns/tree/master/proxy
description: "Wzorce projektowe / strukturalny"
keywords: "pełnomocnik, proxy, wzorzec, wzorce projektowe, wzorzec strukturalny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Pełnomocnik` (ang. `Proxy`) (wzorzec strukturalny) tworzy obiekt, którego zadaniem jest reprezentowanie (pełnienie pełnomocnictwa) innego obiektu. Na podstawie stanu systemu i spełnienia warunków obiekt klasy pełnomocnika `Proxy`, decyduje o utworzeniu bądź nie instancji klasy obiektu docelowego i wywołaniu na nim odpowiednich metod. Takie podejście zapewnia kontrolowane tworzenie kosztownych obiektów oraz kontrolę dostępu. W przypadku nie spełnienia warunków (np.: błędne hasło) pomijany jest proces tworzenia kosztownego obiektu i wywołania na nim metody, która nie może zostać wykonana zgodnie z oczekiwaniami systemu czy użytkownika. Wzorzec ten umożliwia realizowanie dodatkowych zadań w ramach emulowanego obiektu.

## Ograniczenia
Wykorzystując wzorzec `Pełnomocnik`, należy mieć na uwadze, że poza oczywistymi korzyściami wynikającymi z późnego inicjalizowania i dodatkowego sprawdzania warunków można odnotować opóźnienie w odpowiedzi czy wykonaniu zadania. Wzorzec ten jest podobny w strukturze do `Dekoratora`, jednakże `Pełnomocnik` zarządza cyklem życia obiektów emulowanych, natomiast struktura obiektu klasy implementującego wzorzec `Dekorator` jest zarządzana przez użytkownika. Ponadto ze względu na opakowanie inicjalizowania obiektu i wywołania jego metod może przypominać wariacje wzorców `Fasada` oraz `Adapter`. Ze względu na podobieństwa do innych wzorców decyzja o wyborze wzorca `Pełnomocnik` może nie być trywialna.

## Użycie
`Pełnomocnik` jest wykorzystywany przede wszystkim w sytuacjach, gdzie należy zapewnić kontrolę dostępu do obiektu (np. proces autoryzacji). Znajduje również zastosowanie w procesie tworzenia kosztownych obiektów (np. dostęp do bazy danych) oraz emulacji zdalnego obiektu (np. przesyłanie danych).

## Implementacja
Klasa pełnomocnika `Proxy` oraz klasa `RealSubject`, której obiekt udziela pełnomocnictwa implementują interfejs `Subject`. Klasa `Proxy` zawiera prywatne pole typu `Subject`, które w zależności od ustalonych warunków jest inicjalizowane instancją klasy `RealSubject`. Klasa `Proxy` nadpisuje metody interfejsu w taki sposób, że wywołują one odpowiadające im metody obiektu klasy `RealSubject`.

![Pełnomocnik diagram](/assets/img/diagrams/patterns/proxy.svg){: .center-image }

Poniższy listing przedstawia implementacje wzorca `Pełnomocnik` dla klasy `RealSubject` w oparciu o proces autoryzacji.

{% highlight java %}
public class Proxy implements Subject {

    private Subject subject;
    private String password;

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public void action() {
    	if(password != null && password.equals("password")) {
    		if(subject == null)
    		    subject = new RealSubject();
            subject.action();
        }
        else {
        	//show message or do something else
        }
    }
}

public class RealSubject implements Subject {

    @Override
    public void action() {
        //do something
    }
}

interface Subject {

    void action();
}
{% endhighlight %}

Klient inicjalizuje obiekt typu `Subject` instancją klasy `Proxy`, a następnie wywołuje na nim żądane metody.

{% highlight java %}
Subject subject = new RealSubject();
subject.action(); //always execute

Subject proxy = new Proxy();
proxy.action(); //will not effect
proxy.setPassword("password");
proxy.action(); //do action
{% endhighlight %}

## Przykład
Aplikacja `Tasker` umożliwia przechowywanie i zarządzanie listą zadań w trybie online i offline oraz współdzielenia zasobów w obrębie zespołu. Każdy użytkownik zespołu może przeglądać listę zadań, jednak tylko autoryzowani użytkownicy mogą dodawać lub usuwać zadania. Gdy system nie ma połączenia z internetem wówczas ładowane są dane z pamięci podręcznej aplikacji. W celu optymalizacji zużycia kosztownych zasobów (tworzenie obiektów) oraz pakietu danych, aplikacja wykorzystuje wzorzec `Pełnomocnik` do sprawdzenia stanu połączenia internetowego oraz autoryzacji użytkownika, co przedstawia poniższy listing.

{% highlight java %}
public class TaskManagerProxy implements TaskManager {

    @Override
    public List<Task> getTasks() {
    	if(isNetworkActive()) {
            init();
            return taskManager.getTasks();
    	}
    	else {
            //try to get the last saved task list from cache
            List<Task> tasks = getTaskFromCache();
            if(tasks.isEmpty()) {
                //show message about any tasks available
            }
            else {
                //show message about no internet connection, so tasks are not up to date
            }
            return tasks;
        }
    }

    @Override
    public void addTask(Task task) {
        if(isNetworkActive()) {
            if(isAccessProvided()) {
                init();
                taskManager.addTask(task);
            }
            else {
            	//show message about user authorization
            	addTaskToCache(task);
            }
        }
        else {
            //show message about no internet connection
            addTaskToCache(task);
        }
    }

    @Override
    public void deleteTask(Task task) {
        if(isNetworkActive()) {
            if(isAccessProvided()) {
                init();
                taskManager.deleteTask(task);
            }
            else {
                //show message about user authorization
                deleteTaskFromCache(task);
            }
        }
        else {
            //show message about no internet connection
            deleteTaskFromCache(task);
        }
    }

    private boolean isNetworkActive() {
        //check network connection state
        return true; //mock
    }

    private boolean isAccessProvided() {
    	//check is user authorized
    	return true; //mock
    }

    private void init() {
    	if(taskManager == null)
    	    taskManager = new TaskManager();
    }

    private List<Task> getTaskFromCache() {
    	List<Task> tasks = new ArrayList();
    	//get tasks from Cache
    	return tasks;
    }

    private void addTaskToCache(Task task) {
    	//add task to Cache
    	//add task to "resend list" in Cache
    }

    private void deleteTaskFromCache(Task task) {
    	//delete task from Cache
    	//add task to "deleted list" in Cache
    }
}

public class NetworkTaskManager implements TaskManager {

    private Network network;

    public NetworkTaskManager() {
        network = new Network();
        //set network server, port, etc
    }

    @Override
    public List<Task> getTasks() {
        //prepare and send request
        network.setOperation("GET");
        return network.getTasks();
    }

    @Override
    public void addTask(Task task) {
        //prepare and send request with
        network.setOperation("POST");
        network.send(task.toJson());
    }

    @Override
    public void deleteTask(Task task) {
        //prepare and send request
        network.setOperation("DELETE");
        network.send(task.toJson());
    }
}

interface TaskManager {

    List<Task> getTasks();
    void addTask(Task task);
    void deleteTask(Task task);
}
{% endhighlight %}

Sposób wykorzystania `Pełnomocnika` dla `TaskManager` może przebiegać następująco.

{% highlight java %}
//login to app as authorized user
TaskManager taskManager = new TaskManagerProxy();
List<Task> tasks = taskManager.getTasks();
//show tasks in UI

//delete some task
taskManager.deleteTask(tasks.get(0));

//prepare task
Task task = new Task("name", "description", Priority.HIGH);
//internet connection lost
taskManager.addTask(task); //task will be add only to cache
{% endhighlight %}

## Biblioteki
Ze względu na prostotę wzorca opartą o implementacje wspólnego interfejsu, biblioteki implementujące wzorzec `Pełnomocnik` nie mają sensu. Przykładem realizacji wzorca jest klasa `Proxy` wraz z `InvocationHandler` z pakietu `java.lang.reflect`.
