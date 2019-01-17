---
layout: post
title: "Analytics"
date:  2019-02-25
categories: ["Firebase"]
image: firebase/analytics
github: firebase/tree/master/analytics
description: "Firebase"
version: Firebase-Core 16.0
keywords: "firebase, analityka, analytics, zdarzenie, event, właściwość, user properties, android, programowanie, programming"
---

## Cechy
`Google Analytics` dla `Firebase` jest narzędziem analitycznym przeznaczonym do pomiaru oraz rejestrowania wykorzystania aplikacji i zaangażowania użytkowników. Jest rdzeniem dla całej usługi Firebase i pozwala na integracje z różnymi innymi funkcjami. Dostarczane raporty pomagają w zrozumieniu zachowania użytkowników (w jaki sposób korzystają z aplikacji) co ułatwia podejmowanie przemyślanych, świadomych i opartych o trendy decyzje dotyczące marketingu aplikacji i optymalizacji wydajności. `SDK` rejestruje dwa podstawowe typy informacji: zdarzenia (`event`) oraz właściwości użytkownika (`user properties`), które mogą być personalizowane dla aplikacji lub pochodzić z grupy predefiniowanych. Informacje przechwytywane są w sposób automatyczny, a ich podgląd dostępny z poziomu pulpitu nawigacyjnego w `konsoli Firebase`. Na podstawie danych urządzenia, zdarzeń czy właściwości użytkownika możliwe jest tworzenie niestandardowej grupy odbiorców usług Firebase. Dodatkowo raportowane informacje mogą być przypisane do konkretnego użytkownika za pomocą jego identyfikatora przy zachowaniu polityki prywatności i regulaminu. Aby użyć Analytics do ręcznego raportowania dla konkretnego ekranu wystarczy pobrać instancje typu `FirebaseAnalytics` w `onCreate` Aktywności.

{% highlight kotlin %}
//define at the top of the activity
private lateinit var firebaseAnalytics: FirebaseAnalytics

//get instance in onCreate
firebaseAnalytics = FirebaseAnalytics.getInstance(this)
{% endhighlight %}

## Zdarzenia
`Zdarzenia` (`events`) informują o tym co się dzieje w aplikacji, jakie działania podjął użytkownik, jakie wystąpiły zdarzenia systemowe czy błędy. Rozróżniane są za pomocą klucza alfanumerycznego. Analytics automatycznie rejestruje niektóre zdarzenia (nie ma potrzeby dodawania żadnego kodu), które dotyczą interakcji m.in. z reklamami (`ad_click`), notyfikacjami (`notification_open`), wyświetleniem ekranu (`screen_view`), logowaniem (`login`) i rejestracją (`sign_up`), wyszukaniem (`search`) czy też aktualizowaniem (`app_update`), usuwaniem aplikacji (`app_remove`). Jeśli aplikacja wymaga zbierania dodatkowych danych należy je ręcznie raportować za pomocą metody `logEvent` podając klucz typu oraz obiekt `Bundle` zawierający oczekiwane wartości.

{% highlight kotlin %}
class MainActivity : AppCompatActivity() {

    private lateinit var firebaseAnalytics: FirebaseAnalytics

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        firebaseAnalytics = FirebaseAnalytics.getInstance(this)

        buttonAction.setOnClickListener {
            textView.setText(editText.text.toString())
            val bundle = Bundle()
            bundle.putString(FirebaseAnalytics.Param.ITEM_ID, textView.text.toString())
            bundle.putString(FirebaseAnalytics.Param.CONTENT_TYPE, "text")
            firebaseAnalytics.logEvent(FirebaseAnalytics.Event.SELECT_CONTENT, bundle)
        }

        buttonNavigate.setOnClickListener {
            firebaseAnalytics.logEvent("button_navigate_clicked", null)
            val intent = Intent(this, SecondActivity::class.java)
            startActivity(intent)
        }
    }
}
{% endhighlight %}

Przegląd danych zdarzeń w konsoli Firebase może prezentować się jak poniżej. Warto zauważyć, że niecałe 30% użytkowników przechodzi z pierwszego do drugiego ekranu i niewiele więcej wypełnia pole edycji w pierwszym ekranie.

![Statystyki zdarzeń](/assets/img/diagrams/firebase/analytics_events.png){: .center-image }

## Właściwości użytkownika
`Właściwości użytkownika` (`user attributes`) są atrybutami opisującymi segmenty bazy użytkowników dzięki którym przesyłane dane mogą być analizowane i filtrowane pod kątem wskazanej grupy docelowej. Podobnie jak w przypadku zdarzeń niektóre właściwości rejestrowane są w sposób auotomatyczny i są to np. wiek (`Age`), kraj (`Country`), marka urządzenia (`Device brand`), płeć (`Gender`), wersja systemu (`OS Version`) czy język (`Language`). W przypadku przesyłania dodatkowych informacji należy zarejestrować właściwość w konsoli Firebase oraz ustawić właściwość w kodzie za pomocą metody `setUserProperty` przekazując klucz oraz parametr opisujący.

{% highlight kotlin %}
class SecondActivity : AppCompatActivity() {

    private lateinit var firebaseAnalytics: FirebaseAnalytics

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)
        firebaseAnalytics = FirebaseAnalytics.getInstance(this)

        val city = "Poznań" //get property from some place
        firebaseAnalytics.setUserProperty("city", city)
    }
}
{% endhighlight %}

Konsola Firebase dostarcza także informacji nt właściwości użytkownika. Co ciekawe z aplikacji korzystają tylko urządzenia z Android 9, a użytkownicy pochodzą z Polski.

![Statystyki właściwości użytkownika](/assets/img/diagrams/firebase/analytics_user_properties.png){: .center-image }

## Śledzenie ekranów
Analytics śledzi także przejścia między ekranami i dołącza do zdarzeń informacje o aktualnymi ekranie. Kiedy następuje zdarzenie wyświetlenia ekranu automatycznie dołącza parametr `firebase_screen_class` z informacją o nazwie klasy np. `MainActivity` oraz generuje `firebase_screen_id`. Śledzenie ekranów może być zgłaszane ręcznie poprzez metodę `setCurrentScreen` co może być przydatne jeśli aplikacja nie używa oddzielnego kontrolera `UIView` lub `Activity` na każdym śledzonym ekranie.

## Debugowanie
Aby dokonać weryfikacji poprawności konfiguracji Analytics z aplikacją można włączyć debugowanie i sprawdzić na maszynie deweloperskiej (w konsoli `logcat`) czy i jakie informacje są przesyłane.

{% highlight console %}
adb shell setprop log.tag.FA VERBOSE
adb shell setprop log.tag.FA-SVC VERBOSE
adb logcat -v time -s FA FA-SVC
{% endhighlight %}

Dodatkowo konsola Firebase oferuje możliwość wykorzystania trybu `DebugView`, który pozwala na walidację przesyłanych informacji w trybie rzeczywistym. W przeciwieństwie do standardowego produkcyjnego trybu `StreamView` w którym informacje wysyłane są grupowe co jakiś czas, logowanie odbywa się dla każdego zdarzenia osobno z minimalnym opoźnieniem. Aby włączyć tryb `DebugView` na urządzeniu należy wykonać poniższe polecenie.

{% highlight console %}
adb shell setprop debug.firebase.analytics.app <package_name> //enable
adb shell setprop debug.firebase.analytics.app .none. //disable
{% endhighlight %}

Tryb DebugView umożliwia podgląd informacji zarówno w ogólnym trybie minutowym jak w szczegółowym trybie sekundowym.

![Tryb DebugView](/assets/img/diagrams/firebase/analytics_debug_view.png){: .center-image }