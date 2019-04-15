---
layout: post
title: "Remote Config"
date: 2019-04-15
categories: ["Firebase"]
image: firebase/remote_config
github: firebase/tree/master/remote_config
description: "Firebase"
version: Firebase-Config 16.1
keywords: "firebase, chmura, konfiguracja, ustawienia, zdalna, parametr, warunek, remote, config, settings, parameter, condition, testy a/b, testing a/b, predykcje, prognoza, predictions, android, programowanie, programming"
---

## Wprowadzenie
`Remote Config` jest usługą w chmurze, która pozwala na zmianę zachowania, ustawień czy wyglądu aplikacji bez konieczności pobierania aktualizacji. Za pomocą `konsoli Firebase` lub `REST API` można kontrolować i propagować wartości konfiguracji dla wszystkich lub wybranej grupy użytkowników. Dodatkowo wykorzystanie danych z usługi `Analytics` ułatwia analizę grup odbiorców. Aplikacja sprawdza dostępność aktualizacji, przeprowadza modyfikacje ustawień (bez wpływu na wydajność) przechowując konfiguracje w obiekcie `Singleton`, a w przypadku braku zmian na serwerze wykorzystuje lokalne wartości domyślne. 

## Parametry i warunki
Konfiguracja składa się z parametrów opisanych `parą klucz-wartość` i `wartościami domyślnymi` oraz `warunkami` (zbiór reguł) używanymi w celu kierowania konfiguracji do wybranego segmentu odbiorców. Konstruując wyrażenia warunkowe można wykorzystać m.in. następujące reguły: `wersja aplikacja`, `wersja systemu`, `kraj`, `region`, `język`, `właściwość użytkownika`, `losowość`. Zgodnie z ustalonymi priorytetami najpierw sprawdzane są wartości po stronie serwera, które przyjmują wartość dla pierwszego spełnionego warunku. Jeśli żaden warunek nie został spełniony wówczas zwracana jest wartość domyślna z serwera lub lokalna, a w przypadku jej braku przyjmowana jest domyślna wartość inicjalizacyjna danego typu (np. `Int = 0`, `Boolean = false`).

![Parametry i warunki](/assets/img/diagrams/firebase/remote_config_parameters.png){: .center-image }

## Użycie
Aby wykorzystać Remote Config w aplikacji należy dodać parametry i warunki w konsoli Firebase (lub przy pomocy REST API), uzyskać instancję `FirebaseRemoteConfig`, ustawić domyślne lokalne wartości parametrów w aplikacji w pliku zasobów `xml` (lub w obiekcie `Map`), użyć parametry w kodzie oraz podjąć próbę pobrania konfiguracji z serwera metodą `fetch`.

{% highlight xml %}
<!-- remote config defaults -->
<?xml version="1.0" encoding="utf-8"?>
<defaultsMap>
    <entry>
        <key>show_tutorial</key>
        <value>false</value>
    </entry>
    <entry>
        <key>skin</key>
        <value>old</value>
    </entry>
</defaultsMap>
{% endhighlight %}

{% highlight kotlin %}
private fun startRemoteConfig() {
    //get instance
    val remoteConfig = FirebaseRemoteConfig.getInstance()

    //set remote config singleton
    val configSettings = FirebaseRemoteConfigSettings.Builder()
        .setDeveloperModeEnabled(BuildConfig.DEBUG) //for developer purpose
        .build()
    remoteConfig.setConfigSettings(configSettings)

    //set defaults value from xml or Map object
    remoteConfig.setDefaults(R.xml.remote_config_default)

    //fetch parameters from remote config and add some listeners
    remoteConfig.fetch(36000L) //cache expiration for 10h
        .addOnCompleteListener(this) { task ->
            if (task.isSuccessful) {
                remoteConfig.activateFetched() //active new data before use
                //do some action
            } 
            else {
                //do some action
            }

            //get current values and use them in some variable, preferences etc
            val showTutorial = remoteConfig.getBoolean("show_tutorial")
            val skin = remoteConfig.getString("skin")
            //do something with fetched values
        }
}
{% endhighlight %}

## Prognozy
Usługa `Firebase Predictions` wykorzystuje `uczenie maszynowe` oparte o dane analityczne (pochodzące z `Firebase Analytics`) w celu tworzenia `dynamicznych segmentów` użytkowników na podstawie ich zachowań w aplikacji. Utworzone w ten sposób prognozy grupy użytkowników mogą być wykorzystane do wybierania odbiorców zdalnej konfiguracji (`Remote Config`), wiadomości i notyfikacji (`Cloud Messaging`, `In-App Messaging`) czy też testów A/B (`Testing A/B`). Dzięki temu zwiększenie liczby konwersji, kierowanie kampanii, dostosowanie treści i interfejsu użytkownika zgodnie z jego preferencjami i strategią marketingową staje się jeszcze prostsze. Dodatkowo testy A/B pozwalają na przeprowadzenie eksperymentów pod kątem optymalizacji zajścia pewnej akcji dla usług zdalnej konfiguracji oraz wiadomości i notyfikacji. W rezultacie zwracane są wyniki konwersji dla różnych wybranych wariantów co ułatwia testowanie i porównanie zmian w interfejsie użytkownika i kierowanych kampaniach oraz podjęcia decyzji o ich wprowadzeniu dla szerszej grupy odbiorców.