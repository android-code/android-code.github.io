---
layout: post
title: "Analytics"
date:  2019-02-25
categories: ["Firebase"]
image: firebase/analytics
github: firebase/tree/master/analytics
description: "Firebase"
version: Firebase-Core 16.0
keywords: "firebase, analytics, android, programowanie, programming"
---

## Możliwości
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

## Śledzenie ekranów
Analytics śledzi także przejścia między ekranami i dołącza do zdarzeń informacje o aktualnymi ekranie. Kiedy następuje zdarzenie wyświetlenia ekranu automatycznie dołącza parametr `firebase_screen_class` z informacją o nazwie klasy np. `MainActivity` oraz generuje `firebase_screen_id`. Śledzenie ekranów może być zgłaszane ręcznie poprzez metodę `setCurrentScreen` co może być przydatne jeśli aplikacja nie używa oddzielnego kontrolera `UIView` lub `Activity` na każdym śledzonym ekranie.

## Wyświetlanie wyników
//TODO debugView + debugowanie w konsoli
-mozna ustawic tryb debugowania co widac w konsoli