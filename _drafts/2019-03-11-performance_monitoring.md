---
layout: post
title: "Performance Monitoring"
date:  2019-03-11
categories: ["Firebase"]
image: firebase/performance_monitoring
github: firebase/tree/master/performance_monitoring
description: "Firebase"
version: Firebase-Perf 16.2
keywords: "firebase, wydajność, performance, monitoring, trasa, trace, metryka, metric, atrybut, attribute, network, android, programowanie, programming"
---

## Cechy
`Firebase Performance Monitoring` jest usługą pomagającą uzyskać wgląd do wydajności aplikacji na urządzeniach użytkowników dzięki czemu możliwe jest zlokalizowanie i rozwiązanie problemów związanych z wydajnością, płynnością i szybkością działania. Automatycznie dokonuje pomiarów dla czasu startu aplikacji, działania w tle (i na pierwszym planie), renderowania ekranów czy też większości zapytań sieciowych. Przesłane raporty zawierają informacje nt użycia procesora i pamięci, a także metadane urządzenia (m.in. model, wersja systemu, wersja aplikacji, kraj) i szczegółów trasy zgłoszenia. Performance Monitoring nie zbiera żadnych danych personalnych użytkownika. Jeśli automatycznie przesyłane próbki to za mało istnieje również możliwość tworzenia personalizowanych tras śledzenia wydajności. Konsola Firebase automatycznie dokonuje analizy otrzymanych próbek na podstawie których generuje podpowiedzi o potencjalnie niewydajnych miejscach aplikacji.

## Trasy personalizowane
Rejestrowanie wydajności aplikacji zachodzi na tzw. trasach (`traces`), które opisane są między dwoma miejscami: punktem startowym i końcowym. Automatyczna trasa dla działania aplikacji w tle (`App in background`) rozpoczyna się od momentu kiedy ostatnia Aktywność (`Activity`) będąca w `foreground` wywoła `onStop`, a zakończy się w momencie wywołania `onResume` przez pierwszą Aktywność przechodzącą z trybu `background` do `foreground`. Podobnie rzecz ma się z trasami własnymi, które dotyczą wybranego przez programistę przedziału cyklu życia.

{% highlight kotlin %}
class MainActivity : AppCompatActivity() {

    lateinit var trace: Trace

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        trace = FirebasePerformance.getInstance().newTrace("activity_creation")
        trace.start()

        //do some work
    }

    //more lifecycle methods

    override fun onResume() {
        super.onResume()
        trace.stop()
    }
}
{% endhighlight %}

Trasy personalizowane mogą być dodatkowo opisane metrykami, których zadaniem jest liczenie wystąpień pewnych zdarzeń oraz atrybutami informacyjnymi.

{% highlight kotlin %}
//TextView mock, use some RecyclerView or ListView instead
private fun addItem(item: String) {
    //start trace
    val addItemTrace = FirebasePerformance.getInstance().newTrace("add_item_trace")
    addItemTrace.start()

    //set metrics and attributes
    addItemTrace.incrementMetric("text_length", item.length.toLong())
    addIteaTrace.putAttribute("item", item) //avoid personal info

    //update UI
    val text = textViewItems.text
    textViewItems.text = "$text \n $item"

    //stop trace
    addItemTrace.stop()
}
{% endhighlight %}

Dodanie adnotacji `@AddTrace` do metody wraz z jej nazwą powoduje rejestrowanie wydajności podczas wykonywania funkcji (trasa rozpoczyna się na początku metody, a kończy kiedy zostanie wykonana). Jednakże tak stworzone trasy nie mają możliwości dodawania metryk.

{% highlight kotlin %}
//use instead of addItem method if extra info no needed
@AddTrace(name = "add_item_trace")
private fun addItemWithoutMetrics(item: String) {
    val text = textViewItems.text
    textViewItems.text = "$text \n $item"
}
{% endhighlight %}

## Zapytania sieciowe
Wsyłane żądania sieciowe `HTTP/S` są w większośći automatycznie przechwytywane i przetwarzane do postaci raportów zawierających informacje takie jak: czas odpowiedzi, rozmiar wysłanych żądań i otrzymanych odpowiedzi, wskaźnik sukcesu oraz metadane urządzenia. Ze względu na zachowanie polityki prywatności Performance Monitoring pomija parametry `URL` w procesie budowania anonimowych wzorców adresów wyświetlanych w `konsoli Firebase`. Rejestrowanie zdarzeń sieciowych wspierane jest tylko dla żądań stworzony przy użyciu `OkHttp3`. W przypadku personalizacji raportowania zapytań sieciowych lub ich ręcznego wywołania można wykorzystać kod na poniższym listingu.

{% highlight kotlin %}
private fun download(link: String) {
    //run on background thread
    val url = URL(link)
    val metric = FirebasePerformance.getInstance().newHttpMetric(link, FirebasePerformance.HttpMethod.GET)
    metric.start()

    //set connections
    val conn = url.openConnection() as HttpURLConnection
    conn.doOutput = true
    conn.setRequestProperty("Content-Type", "application/json")
    val content = convertStreamToString(conn.outputStream) //do something with content
    
    metric.setHttpResponseCode(conn.responseCode)
    conn.disconnect()
    metric.stop()
}

fun convertStreamToString(outputStream: OutputStream): String {
    return "webpage content" //mock
}
{% endhighlight %}

## Debugowanie
Tryb debugowania pozwala już na wczesnym etapie deweloperskim sprawdzić poprawność konfiguracji oraz otrzymywane rezultaty. Aby włączyć debugowanie należy dodać odpowiedni wpis `meta-data` do `AndroidManifest` oraz uruchomić właściwe polecenie w `adb`.

{% highlight xml %}
<meta-data
  android:name="firebase_performance_logcat_enabled"
  android:value="true" />
{% endhighlight %}

{% highlight console %}
adb logcat -s FirebasePerformance
{% endhighlight %}

## Automatyczne raportowanie
Zbieranie i wysyłanie raportów jest domyślnie włączone dla wszystkich użytkowników i wersji aplikacji. W przypadku, gdy istnieje potrzeba opcjonalnego wyłączenia raportowania (np. dla wersji deweloperskich) należy dodać odpowiedni wpis `meta-data` do `AndroidManifest` lub zmienić wartość właściwości `firebasePerformanceInstrumentationEnabled` w `gradle.properties`.

{% highlight xml %}
<!-- disable at build time but allow to enable at runtime -->
<meta-data
  android:name="firebase_performance_logcat_enabled"
  android:value="true" />

<!-- disable at build time and don't allow to enable at runtime -->
<meta-data
  android:name="firebase_performance_collection_deactivated"
  android:value="true" />
{% endhighlight %}

Aby dokonywać zmian (włączenia lub wyłączenia raportowania) w trakcie działania programu ze względu na ustawienia użytkownika lub zdalną konfigurację wystarczy zmienić wartość właściwości `isPerformanceCollectionEnabled`.

{% highlight kotlin %}
FirebasePerformance.getInstance().isPerformanceCollectionEnabled = true //get value from pref or remote config
{% endhighlight %}
