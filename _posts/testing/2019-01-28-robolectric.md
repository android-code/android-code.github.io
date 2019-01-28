---
layout: post
title: "Robolectric"
date: 2019-01-28
categories: ["Testowanie"]
image: testing/robolectric
github: testing/tree/master/robolectric
description: "Testowanie"
version: Robolectric v4.12
keywords: "testowanie, testing, testy, jednostkowe, automatyczne, lokalne, instrumentalne, zaślepka, atrapa, unit test, mock, stub, robolectric, robolectrictestrunner, config, shadow, build, setup, controller, activitycontroller, activity, fragment, service, intent, contenrprovider, android, programowanie, programming"
---

## Przeznaczenie
Uruchamianie jednostkowych testów instrumentalnych na urządzeniu lub emulatorze jest kosztowne i powolne. Jednakże w wielu sytuacjach nie sposób uciec od testowania logiki kodu powiązanego z komponentami Androida co sprawia, że testy instrumentalne są stosowane pomimo wynikających kosztów. Alternatywnym sposobem testowania kodu zależnego od `Android SDK` może być użycie framework `Robolectric`, który umożliwia przeprowadzanie `lokalnych testów` jednostkowych w środowisku wykonawyczm `JVM` na komponentach Android takich jak m.in. `Activity`, `Fragment`, `Intent`, `Service` czy `ContentProvider`. W przeciwieństwie do testów instrumentalnych Robolectric wykonuje się piaskownicy systemowej (`sandbox`) co pozwala na konfiguracje środowiska Android dla wszystkich testów. Robolectric dostarcza również `dublerów` komponentów Android dla których nie potrafi dokonać tłumaczenia do testów jednostkowych (np. sensory hardware, usługi systemowe). Obsługuje inflację widoków, ładowanie zasobów i wiele innych aspektów zaimplementowanych w natywnym kodzie na urządzeniach co pozwala na wykonanie większości czynności tak jak na prawidzwym urządzeniu. Robolectric w łatwy sposób pozwala na zapewnienie własnej implementacji wybranych metod Android SDK dzięki czemu możliwa jest np. symulacja warunków czy zachowań sensorów. Wykorzystanie Robolectric jest bliższe podejściu testowania czarnoskrzynkowego (skupionego na zachowaniu) i nie wyklucza jednoczesnego stosowania innych bibliotek naiwnych implementacji takich jak np. `Mockito`.

>**Przykład**  
>Na podstawie Aktywności `MainActivity` zostaną przedstawione możliwości implementacji testów jednostkowych w Robolectric.

{% highlight kotlin %}
class MainActivity : AppCompatActivity() {

    companion object {
        const val TITLE_KEY = "TITLE"
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        buttonAction.setOnClickListener {
            val intent = Intent(this, SecondActivity::class.java)
            startActivity(intent)
        }

        textViewTitle.setOnClickListener { textViewTitle.setText(R.string.clicked) }
    }

    override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
        super.onRestoreInstanceState(savedInstanceState)
        savedInstanceState?.let {
            if(it.containsKey(TITLE_KEY))
                textViewTitle.text = it.getString(TITLE_KEY)
        }
    }
}
{% endhighlight %}

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

## Konfiguracja
Aby zmienić domyślną konfiguracje testów należy oznaczyć klasy lub metody adnotacją `@Config` wraz z deklaracją konfiguracji. Metody z adnotacją `@Config` nadpisują zachowanie z poziomu klasy. Konfiguracji mogą podlegać m.in. poziom `sdk`, plik `manifest`, klasy `shadows`, ścieżki do zasobów czy `kwalifikatory` (np. języka, regionu itp).

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
`ActivityController` odpowiada za tworzenie oraz zarządzanie cyklem życia tworzonej Aktywności. Metoda `buildActivity` tworzy instancję `ActivityController`, która umożliwia wywołanie wybranych metod cyklu życia (`create`, `start`, `resume`, `pause`, `stop`, `destroy`) oraz pobranie instancji Aktywności dla bieżącego stanu. Aby bezpośrednio uzyskać obiekt Aktywności z przebytym pełnym cyklem tworzenia należy wywołać metodę `setupActivity`. 

{% highlight kotlin %}
@RunWith(RobolectricTestRunner::class)
class LifecycleTest {

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
    fun checkMainActivityHasWorkingRestoringState() {
        val savedInstanceState = Bundle()
        savedInstanceState.putString(TITLE_KEY, "value restored")
        val activity = Robolectric.buildActivity(MainActivity::class.java)
            .create().restoreInstanceState(savedInstanceState).get()

        assertEquals("value restored", activity.textViewTitle.text)
    }
}
{% endhighlight %}

W analogiczny sposób można zarządzać innymi kompontentami Androida i ich cyklem życia poprzez operowanie na dedykowanym kontrolerze (`ServiceController`, `FragmentController`, `ContentProviderController` itp.) oraz wykonanie metod odpowiadających ich cyklowi życia.

{% highlight kotlin %}
@RunWith(RobolectricTestRunner::class)
class ServiceTest {

    @Test
    fun checkSomeServiceProperlyBindAndUnbind() {
        val service = Robolectric.buildService(SomeService::class.java).bind().get()
        assertTrue(service.bound)
        service.onUnbind(null)
        assertFalse(service.bound)
    }
}

//this is only dummy implementation
class SomeService : Service() {

    var bound = false

    override fun onBind(intent: Intent?): IBinder? {
        bound = true
        return null
    }

    override fun onUnbind(intent: Intent?): Boolean {
        bound = false
        return super.onUnbind(intent)
    }

    //more methods
}
{% endhighlight %}

## Shadow
Robolectric tworzy środowisko wykonawcze zawierające prawdziwy kod Android SDK co zwiększa realizm testów tak jakby były przeprowadzane na fizycznym urządzeniu. Wszystkie klasy Android są zastąpione tzw. `obiektami cienia` (`Shadows`). Każdy obiekt cienia może modyfikować lub rozszerzać zachowanie odpowiadającej mu klasy z Android SDK. Aby stworzyć klasę `Shadow` należy oznaczyć ją adnotacją `@Implements` wraz z nazwą odpowiadającej klasy Android, opcjonalnie rozszerzyć superklasę Shadow oraz dostarczyć publiczny bez argumentowy konstruktor. Metody rozszerzene muszą mieć adnotację `@Implementation` (obiekt cienia implementuje metody z tą samą sygnaturą dla odpowiadającej klasy Android niezależnie od modyfikatora dostępu) i przeważnie modyfikator `protected`. Metoda `directlyOn` pozwala na wywołanie akcji na faktycznym obiekcie.

{% highlight kotlin %}
@Implements(TextView::class)
class CustomShadowTextView : ShadowView() {

    @Implementation
    fun setEnabled(enable: Boolean) {
        directlyOn(realView, View::class.java).setEnabled(enable)
        if(enable)
            directlyOn(realView, View::class.java).setAlpha(1.0f)
        else
            directlyOn(realView, View::class.java).setAlpha(0.5f)
    }

    fun getAlpha() : Float {
        return realView.alpha
    }
}
{% endhighlight %}

Klasa testowa lub metoda wykorzystująca własną implementacje klas Shadows musi być oznaczona adnotacją `@Config(shadows=arrayOf(CustomShadowClass::class))` (lub zawierać odpowiedni wpis w pliku ustawień) co umożliwia rozpoznanie i skojarzenie klas Shadow z klasami przykrywanymi. Metoda `shadowOf` jest przeznaczona dla dostarczonych przez Robolectric klas Shadow, natomiast `extract` dla autorskich klas Shadow.

{% highlight kotlin %}
@RunWith(RobolectricTestRunner::class)
@Config(shadows=arrayOf(CustomShadowTextView::class))
class ShadowTest {

    @Test
    fun shadowCustomClassImplicit() {
        val activity = Robolectric.setupActivity(MainActivity::class.java)
        val view = activity.textViewTitle //TextView instance
        view.setEnabled(false) 
        assertEquals(0.5f, view.getAlpha())
    }

    @Test
    fun shadowCustomClassExplicit() {
        val activity = Robolectric.setupActivity(MainActivity::class.java)
        val view = extract(activity.textViewTitle) as CustomShadowTextView
        view.setEnabled(false)
        assertEquals(0.5f, view.getAlpha())
    }
}
{% endhighlight %} 