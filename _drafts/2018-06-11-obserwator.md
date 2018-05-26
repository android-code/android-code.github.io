---
layout: post
title: "Obserwator"
date:  2018-06-11
categories: ["Wzorce projektowe"]
image: observer
github: observer
description: "Wzorce projektowe / behawioralny"
keywords: "obserwator, observer, wzorzec, wzorce projektowe, design patterns, android, java, programowanie, programming, rxandroid, rxjava, szyna zdarzeń, eventbus, event bus, otto"
---

## Zastosowanie
`Obserwator` (ang. `Observer`) (wzorzec behawioralny) służy do powiadamiania obiektów zainteresowanych (`Subskrybentów`) o zmianie stanu obiektu śledzonego (`Obserwowanego`). Obiekty nasłuchujące mogą być zależne od stanu obiektu obserwowanego w związku czym musi istnieć między nimi mechanizm komunikacji. Zmiana stanu jest równoważna za zmianą wartości pól obiektu. Wzorzec `Obserwator` może być stosowany również do powiadamiania o stanie podjętej operacji i wygenerowaniu odpowiedniego zdarzenia (`event`). Obiekt może być obserwowany przez wielu subskrybentów, a także sam może być obserwatorem innego obiektu. Komunikaty nadawane przez obiekt obserwowany trafiają do wszystkich obserwatorów.

## Ograniczenia
Obserwatorzy otrzymują komunikat z którego wiadomo, że coś się zmieniło, ale nie zawsze wiadomo co. Ponadto komunikat ten trafia do wszystkich subskrybentów, którzy niekoniecznie są zainteresowani daną zmianą stanu. Jeśli obiekt obserwowany jest także obserwatorem wówczas istnieje ryzyko zapętlęnia mechanizmu powiadamiania. 

## Użycie
`Obserwator` wykorzystywany jest tam, gdzie istnieje potrzeba informowania wielu obiektów w jednakowy sposób o zmianie stanu obserwowanego lub stanu podjętej operacji. Ze względu na ograniczenia wzorzec ten bywa przede wszystkim używany do informowania elementów systemu o wystąpieniu zdarzenia błędu czy też odpowiedzi sieciowych. 

## Implementacja

![Obserwator diagram](/assets/img/diagrams/observer.svg){: .center-image }

{% highlight java %}

{% endhighlight %}

## Przykład

## Biblioteki
Wzorzec `Obserwator` w implementacji klas `Observer` i `Observable` należał do standardowej biblioteki `Java`, jednakże ze względu na wspomniane zagrożenia oraz brak zapewnienia poprawnego działania w środowisku wielowątkowym, został uznany w `Java 9` jako `deprecated`. Jego następnikiem stały się klasy `PropertyChangeListener` i `PropertyChangeEvent`, a także `reaktywne strumienie` z pakietu `Flow`. Popularnymi bibliotekami wzorca `Obserwator` w Androidzie są `szyny zdarzeń` `EventBus` i `Otto`, a także `RxAndroid` powstały na bazie `RxJava`, który jest implementacją podejścia do `programowania reaktywnego`.
