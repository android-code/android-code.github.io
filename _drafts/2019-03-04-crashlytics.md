---
layout: post
title: "Crashlytics"
date:  2019-03-04
categories: ["Firebase"]
image: firebase/crashlytics
github: firebase/tree/master/crashlytics
description: "Firebase"
version: Crashlytics 2.9
keywords: "firebase, crashlytics, crash, android, programowanie, programming"
---

## Możliwości
`Firebase Crashlytics` jest narzędziem raportującym awarie w aplikacji w czasie rzeczywistym, które pochodzą z urządzeń użytkowników na których zainstalowana jest aplikacja włączając w to urządzenia deweloperskie. Pomaga śledzić błędy oraz ustalać ich priorytety dzięki czemu możliwa jest weryfikacja i naprawa problemów związanych ze stabilnością co znacząco wpływa na utrzymanie jakości. Inteligentne grupowanie awarii wraz z informacjami o okolicznościach, które do nich doprowadziły pozwala zaoszczędzić czas w procesie diagnozy, zlokalizować błędny lub potencjalnie niebezpieczny fragment kodu oraz ustalić zakres użytkowników których awaria dotyczy. Crashlytics jest ogromnym wsparciem dla deweloperów w procesie utrzymania i rozwoju aplikacji.

## Zgłaszanie
Crashlytics automatycznie zbiera i wysyła raporty o występujących błędach i awariach do konsoli Firebase (np. gdy zostanie wyrzucony `fatal exception`) oraz umożliwia ręczne logowanie przechwyconych błędów za pomocą metody `logException`. Warto odnotować, że wyjątki nie są zgłaszane pojedynczo lecz grupowo co odbywa się na dedykowanym wątku w tle dzięki czemu wpływ na wydajność aplikacji jest minimalny. Crashlytics przechowuje informacje o 8 ostatnich wyjątkach.

{% highlight kotlin %}
buttonDivide.setOnClickListener {
    try {
    	//mock exception
        val value = div(10, 0)
        //do more work
    } 
    catch (e: Exception) {
        Crashlytics.logException(e)
    }
}

private fun div(a: Int, b: Int): Int {
    return a/b
}
{% endhighlight %}

Ponadto pozwala na personalizację zgłaszanych informacji poprzez wypisywanie logów do konsoli oraz dodawanie niestandardowych kluczy pomagających uzyskać określony stan aplikacji prowadzący do awarii.

{% highlight kotlin %}
buttonThirdValue.setOnClickListener {
    val values = arrayListOf(1,2) //mock values
    try {
    	//mock exception
        val value = getThirdValue(values)
        //do more work
    } 
    catch (e: Exception) {
        Crashlytics.log("buttonThirdValue action error") //to logcat
        Crashlytics.setInt("array_size", values.size)
        Crashlytics.logException(e) //log to Firebase
    }
}

private fun getThirdValue(values: ArrayList<Int>): Int {
    return values[2]
}
{% endhighlight %}

W diagnozie problemów często przydatna może być informacja identyfikująca użytkownika, który doświadczył awarii co możliwe jest poprzez ustawienie id metodą `setUserIdentifier`.

## Debugowanie
Aby sprawdzić poprawność konfiguracji Crashlytics z aplikacją nie trzeba czekać na pierwsze zgłoszenia wyjątków lecz można ręcznie wymuśić przesłanie zgłoszenia metodą `crash`.

{% highlight kotlin %}
Crashlytics.getInstance().crash();
{% endhighlight %}

Dodatkowo w celu weryfikacji przesyłanych informacji warto włączyć tryb debugowania.

{% highlight console %}
//before running app
adb shell setprop log.tag.Fabric DEBUG
adb shell setprop log.tag.CrashlyticsCore DEBUG
//for disable use INFO flag

//view the logs
adb logcat -s Fabric CrashlyticsCore
{% endhighlight %}

## Automatyczne raportowanie
Zbieranie i wysyłanie raportów jest domyślnie włączone dla wszystkich użytkowników. Aby dać użytkownikom możliwość decydowania o przesyłanych informacjach należy wyłączyć automatyczne raportowanie dodając odpowiedni wpis `meta-data` do `AndroidManifest` oraz włączyć w kodzie nasłuchiwanie dla wybranych użytkowników.

{% highlight xml %}
//in AndroidManifest.xml
<meta-data
    android:name="firebase_crashlytics_collection_enabled"
    android:value="false" />
{% endhighlight %}

{% highlight kotlin %}
//enable from one of app's Activity
private fun enableCrashlytics() {
    val enable = true //mock value, get from some preferences
    if(enable)
        Fabric.with(this, Crashlytics())
}
{% endhighlight %}
