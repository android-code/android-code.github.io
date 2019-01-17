---
layout: post
title: "Performance Monitoring"
date:  2019-03-11
categories: ["Firebase"]
image: firebase/performance_monitoring
github: firebase/tree/master/performance_monitoring
description: "Firebase"
version: Firebase-Perf 16.2
keywords: "firebase, wydajność, performance, monitoring, trasa, trace, network, android, programowanie, programming"
---

## Cechy
`Firebase Performance Monitoring` jest usługą pomagającą uzyskać wgląd do wydajności aplikacji na urządzeniach użytkowników dzięki czemu możliwe jest zlokalizowanie i rozwiązanie problemów związanych z wydajnością, płynnością i szybkością działania. Automatycznie dokonuje pomiarów dla czasu startu aplikacji, działania w tle (i na pierwszym planie), renderowania ekranów czy też większości zapytań sieciowych. Przesłane raporty zawierają informacje nt użycia procesora i pamięci, a także metadane urządzenia (m.in. model, wersja systemu, wersja aplikacji, kraj) i szczegółów trasy zgłoszenia. Performance Monitoring nie zbiera żadnych danych personalnych użytkownika. Jeśli automatycznie przesyłane próbki to za mało istnieje również możliwość tworzenia personalizowanych tras śledzenia wydajności. Konsola Firebase automatycznie dokonuje analizy otrzymanych próbek na podstawie których generuje podpowiedzi o potencjalnie niewydajnych miejscach aplikacji.

## Trasy personalizowane
Rejestrowanie wydajności aplikacji zachodzi na tzw. trasach (`traces`), które opisane są między dwoma miejscami: punktem startowym i końcowym. Automatyczna trasa dla działania aplikacji w tle (`App in background`) rozpoczyna się od momentu kiedy ostatnia Aktywność (`Activity`) będąca w `foreground` wywoła `onStop`, a zakończy się w momencie wywołania `onResume` przez pierwszą Aktywność przechodzącą z trybu `background` do `foreground`. Podobnie rzecz ma się z trasami własnymi, które dotyczą wybranego przez programistę przedziału cyklu życia.

//TODO
{% highlight kotlin %}
val myTrace = FirebasePerformance.getInstance().newTrace("test_trace")
myTrace.start()
myTrace.stop()
{% endhighlight %}

Trasy personalizowane mogą być dodatkowo opisane metrykami, których zadaniem jest liczenie wystąpień pewnych zdarzeń.

//TODO
{% highlight kotlin %}
val item = cache.fetch("item")
if (item != null) {
    myTrace.incrementMetric("item_cache_hit", 1)
} else {
    myTrace.incrementMetric("item_cache_miss", 1)
}
{% endhighlight %}

Dodanie adnotacji `@AddTrace` do metody wraz z jej nazwą powoduje rejestrowanie wydajności podczas wykonywania funkcji (trasa rozpoczyna się na początku metody, a kończy kiedy zostanie wykonana). Jednakże tak stworzone trasy nie mają możliwości dodawania metryk.

//TODO
{% highlight kotlin %}
@AddTrace(name = "onCreateTrace", enabled = true /* optional */)
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
}
{% endhighlight %}

## Zapytania sieciowe
Wysyłane żądania sieciowe `HTTP/S` są przechwytywane i przetwarzane do postaci raportów zawierających informacje takie jak: czas odpowiedzi, rozmiar wysłanych żądań i otrzymanych odpowiedzi, wskaźnik sukcesu oraz metadane urządzenia. Ze względu na zachowanie polityki prywatności Performance Monitoring pomija parametry `URL` w procesie budowania anonimowych wzorców adresów wyświetlanych w `konsoli Firebase`. Rejestrowanie zdarzeń sieciowych wspierane jest tylko dla żądań stworzony przy użyciu `OkHttp3`.

//TODO
{% highlight kotlin %}
val metric = FirebasePerformance.getInstance().newHttpMetric("https://www.google.com",
        FirebasePerformance.HttpMethod.GET)
val url = URL("https://www.google.com")
metric.start()
val conn = url.openConnection() as HttpURLConnection
conn.doOutput = true
conn.setRequestProperty("Content-Type", "application/json")
try {
    val outputStream = DataOutputStream(conn.outputStream)
    outputStream.write(data)
} catch (ignored: IOException) {
}

metric.setRequestPayloadSize(data.size.toLong())
metric.setHttpResponseCode(conn.responseCode)
printStreamContent(conn.inputStream)

conn.disconnect()
metric.stop()
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