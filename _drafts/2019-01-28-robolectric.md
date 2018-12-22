---
layout: post
title: "Robolectric"
date: 2019-01-28
categories: ["Testowanie"]
image: testing/robolectric
github: testing/tree/master/robolectric
description: "Robolectric"
keywords: "testowanie, testing, testy, jednostkowe, automatyczne, instrumentalne, zaślepka, atrapa, unit test, mock, stub, robolectric, android, programowanie, programming"
---

## Przeznaczenie
Uruchamianie jednostkowych testów instrumentalnych na urządzeniu lub emulatorze jest kosztowne i powolne. Jednakże w wielu sytuacjach nie sposób uciec od testowania logiki kodu powiązanego z komponentami Androida co sprawia, że testy instrumentalne są stosowane pomimo wynikających kosztów. Alternatywnym sposobem testowania kodu zależnego od `Android SDK` może być użycie framework `Robolectric`, który umożliwia przeprowadzanie `lokalnych testów` jednostkowych w środowisku wykonawyczm `JVM` na komponentach Android takich jak m.in. `Activity`, `Fragment`, `Intent`, `Service`, `ContentProvider`. W przeciwieństwie do testów instrumentalnych Robolectric wykonuje się piaskownicy systemowej (`sandbox`) co pozwala na konfiguracje środowiska Android dla wszystkich testów. Robolectric dostarcza również dublerów komponentów Android dla których nie potrafi dokonać tłumaczenia do testów jednostkowych (np. sensory hardware, usługi systemowe). Obsługuje inflację widoków, ładowanie zasobów i wiele innych aspektów zaimplementowanych w natywnym kodzie na urządzeniach co pozwala na wykonanie większości czynności jak na prawidzwym urządzeniu. Robolectric w łatwy sposób umożliwia zapewnienie własnej implementacji wybranych metod Android SDK dzięki czemu możliwa jest np. symulacja warunków czy zachowań sensorów. Wykorzystanie Robolectric jest bliższe podejściu testowania czarnoskrzynkowego (skupionego na zachowaniu) i nie wyklucza jednoczesnego stosowania innych bibliotek naiwnych implementacji takich jak np. `Mockito`.

## Uruchomienie
Aby uruchomić test w Robolectric należy opatrzeć klasę testową adnotacją `@RunWith(RobolectricTestRunner.class)` oraz uzyskać instancję żądanej klasy komponentu.

{% highlight kotlin %}
@RunWith(RobolectricTestRunner::class)
class SimpleTest {

    lateinit var activity: MainActivity

    @Before
    fun init() {
    	//this method provides that activity goes throw all create lifecycle
        activity = Robolectric.setupActivity(MainActivity::class.java)
    }

    @Test
    fun checkViewsValue() {
        assertEquals("title", activity.textViewTitle.text)
        assertEquals("action", activity.buttonAction.text)
    }

    @Test
    fun clickingTextViewChangeText() {
        activity.textViewTitle.performClick()
        assertEquals("clicked", activity.textViewTitle.text)
    }

    @Test
    fun clickingButtonNavigateToSecondActivity() {
        activity.buttonAction.performClick()
        val expectedIntent = Intent(activity, SecondActivity::class.java)
        val actual = shadowOf(getApplicationContext() as Application).nextStartedActivity
        assertNotNull(actual)
        assertEquals(expectedIntent.component, actual.component)
    }
}
{% endhighlight %}


## Shadow


## Konfiguracja
Aby zmienić domyślną konfiguracje testów należy oznaczyć klasy lub metody adnotacją `@Config` wraz z deklaracją konfiguracji. Metody z adnotacją `@Config` nadpisują zachowanie poziomu klasy. Konfiguracji mogą podlegać m.in. poziom `sdk`, plik `manifest`, klasy `shadows`, ścieżki do zasobów czy `kwalifikatory` (np. języka, regionu itp).

{% highlight kotlin %}
@RunWith(RobolectricTestRunner::class)
@Config(minSdk = LOLLIPOP)
class ConfigurationTest {

    lateinit var activity: MainActivity

    @Before
    fun init() {
    	//this method provides that activity goes throw all create lifecycle
        activity = Robolectric.setupActivity(MainActivity::class.java)
    }

    @Test
    fun checkViewsValueOnLollipopAndAbove() {
        assertEquals("title", activity.textViewTitle.text)
        assertEquals("action", activity.buttonAction.text)
    }

    @Config(minSdk = KITKAT, maxSdk = LOLLIPOP)
    @Test
    fun checkViewsValueOnMinKitkatMaxLollipop() {
        assertEquals("title", activity.textViewTitle.text)
        assertEquals("action", activity.buttonAction.text)
    }

    @Config(sdk = intArrayOf(O, O_MR1, P))
    @Test
    fun checkViewsValueOnlyOnOreoAndPie() {
        assertEquals("title", activity.textViewTitle.text)
        assertEquals("action", activity.buttonAction.text)
    }

    @Config(qualifiers = "pl")
    @Test
    fun checkViewsValueInPolish() {
        assertEquals("tytuł", activity.textViewTitle.text)
        assertEquals("akcja", activity.buttonAction.text)
    }

    @Config(qualifiers = "xhdpi")
    @Test
    fun checkViewsValueOnXhdpi() {
        assertEquals(32, activity.buttonAction.paddingTop)
    }
}
{% endhighlight %}

## Cykl życia
`ActivityController` odpowiada za tworzenie oraz zarządzanie cyklem życia tworzonej Aktywności. Metoda `buildActivity` tworzy instancję `ActivityController`, która umożliwia wywołanie wybranych metod cyklu życia oraz pobranie instancji Aktywności. Aby bezpośrednio uzyskać obiekt Aktywności z przebytym pełnym cyklem tworzenia należy wywołać metodę `setupActivity`.

{% highlight kotlin %}
@RunWith(RobolectricTestRunner::class)
class LifecycleTest {
    
    @Test
    fun checkMainActivityHasWorkingFinishLifecycle() {
        //this activity goes throw all create lifecycle
        val controller = Robolectric.buildActivity(MainActivity::class.java)
        val activity = controller.create().start().resume().visible().get()
        //equivalent of val activity = Robolectric.setupActivity(MainActivity::class.java)
        //visible methods assures that activity view's is attached

        assertFalse(activity.isFinishing)
        activity.finish()
        assertTrue(activity.isFinishing)
    }

    @Test
    fun checkMainActivityHasWorkingLifecycle() {
        val controller = Robolectric.buildActivity(MainActivity::class.java)
        val activity = controller.create().get()

        assertEquals(Lifecycle.State.CREATED, activity.lifecycle.currentState)
        //do more assertion for this state
        controller.start()
        assertEquals(Lifecycle.State.STARTED, activity.lifecycle.currentState)
        //do more assertion for this state
        controller.resume()
        assertEquals(Lifecycle.State.RESUMED, activity.lifecycle.currentState)
        //do more assertion for this state
        controller.pause().stop().destroy()
        assertEquals(Lifecycle.State.DESTROYED, activity.lifecycle.currentState)
    }
}
{% endhighlight %}