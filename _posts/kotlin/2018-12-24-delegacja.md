---
layout: post
title: "Delegacja"
date:  2018-12-24
categories: ["Kotlin"]
image: kotlin/delegation
github: kotlin/blob/master/delegation.kt
description: "Kotlin"
keywords: "kotlin, delegacja, delegat, właściwości, delegation, delegate, properties, lazy, observable, map, android, programowanie, programming"
---

## Wzorzec
`Delegacja` (`Delegation`) jest wzorcem stanowiącym alternatywę dla implementacji przez `dziedziczenie` i polega na tym, że klasa dziedzicząca zamiast implementować zachowanie deleguje je do wstrzykniętego obiektu. Wiele wzorców projektowych m.in. `Strategia`, `Wizytor`, `Obserwator` w swym działania wykorzystują wzorzec `Delegacja`. `Kotlin` natywnie wspiera implementacje wzorca Delegacja całkowicie redukując powtarzający się kod (`boilerplate`). Klasa wykorzystująca wzorzec Delegacja musi w swojej implementacji wstrzyknąć obiekt delegata oraz wskazać go w definicji za pomocą słowa kluczowego `by`.

{% highlight kotlin %}
interface Base {
    val text: String
    fun show()
    fun showFull()
}

class BaseImpl(text: String) : Base {
    override val text = "Delegate implementation: $text"
    override fun show() { print(text) }
    override fun showFull() { print("Delegate implementation full info: + $text") }
}

class Derived(delegate: Base) : Base by delegate

val base = BaseImpl("value")
val derived = Derived(base)
derived.show() //Delegate implementation: value
{% endhighlight %}

## Nadpisywanie
Klasy implementujące wzorzec Delegacja mogą rozszerzyć lub zrezygnować z implementacji obiektu delegata poprzez nadpisanie wybranego członka klasy. Należy jednak mieć na uwadze, że implementacja nadpisanych właściwości nie jest widoczna dla delegata.

{% highlight kotlin %}
class Derived(delegate: Base) : Base by delegate {
    override val text = "Overriden implementation: $text"
    override fun showFull() { print("Overriden implementation full info: $text") }
}

val base = BaseImpl("value")
val derived = Derived(base)
derived.show() //Delegate implementation: value
print(derived.text) //Overriden implementation: value
derived.showFull() //Overriden implementation full info: value
{% endhighlight %}

## Właściwości
`Właściwości delegowane` (`delegated properties`) są mechanizmem dzięki któremu można delegować wywołanie metod akcesorów właściwości `get` oraz `set` do innego obiektu co odbywa w sposób zautomatyzowany. Implementacja jest jednorazowa eliminując potrzebę każdorazowego ręcznego przypisania. Kotlin wspiera właściwości delegowane za pomocą: leniwych właściwości (`lazy properties`), oberwowalnych właściwości (`observable properties`) oraz przechowywania właściwości w mapie. Aby zadeklarować właściwość jako delegowaną należy po nazwie oraz typie użyć słowa kluczowego `by`, a następnie podać wyrażenie reprezentujące delegata, który musi dostarczyć implementację metod `getValue` oraz `setValue` co przedstawia się następująco.

{% highlight kotlin %}
class DelegatedProperty {
    var value: String by Delegate()
}

class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "get '${property.name}' has been delegated"
    }
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
    	print("set '${property.name}' = $value has been delegated")
    }
}

val delegated = DelegatedProperty()
print(delegated.value) //get 'value' has been delegated
delegated.value = "new one" //set 'value' = new one has been delegated
{% endhighlight %}

Wywołanie akcesorów właściwości delegowane jest do metod getValue i setValue delegata, które mogą zostać dostarczone jako funkcje klasy, funkcje rozszerzające czy też implementacje interfejsów `ReadOnlyProperty` lub `ReadWriteProperty`. Delegowanie właściwości może mieć miejsce nie tylko dla właściwości klasy, ale także dla zmiennych lokalnych.

## Leniwa właściwość
Funkcja `lazy` przyjmuje wyrażenie lambda i zwraca instancje typu `Lazy<T>`, która pełni rolę delegata dla właściwości. Pierwsze wywołanie get uruchamia wyrażenie lambda i zapamiętuje rezultat dzięki czemu kolejne odwołania zwracają zapamiętaną już wartość. Domyślnie obliczanie wartości jest w trybie synchronizowanym, tzn. odbywa się w jednym wątku przez co wszystkie wątki mają tą samą wartość. Aby wybrać tryb ręcznie należy przekazać do funkcji lazy jeden z parametrów `LazyThreadSafetyMode`: `SYNCHRONIZED`, `PUBLICATION`, `NONE`.

{% highlight kotlin %}
val lazyValue: String by lazy {
    print("evaluated ")
    "value"
}
print(lazyValue) //evaluted value - at first time is computed
print(lazyValue) //value - is computed already
{% endhighlight %}

## Obserwowalna właściwość
Funkcja `Delegates.observable` przyjmuje dwa argumenty: wartość początkową oraz uchwyt do modyfikacji właściwości, który jest wywoływany przy każdym przypisaniu wartości do właściwości. Słuchacze są powiadamiani o zmianach we właściwości. Uchwyt opisany jest trzema parametrami: właściwość, stara wartość oraz nowa wartość. Użycie funkcji `Delegates.vetoable` umożliwia przechwytywanie przypisania.

{% highlight kotlin %}
class Person {
    var name: String by Delegates.observable("No name ") {
        property, old, new ->
        print("$old changed to $new")
    }
}

val person = Person()
person.name = "Johnnie" //No name changed to Johnnie
person.name = "Jack" //Johnnie changed to Jack
{% endhighlight %}	

## Przechowywanie w Map
W dynamicznych procesach jak np. parsowanie formatów warto wykorzystać przechowywanie właściwości w `Map` (`MutableMap` dla zmiennych var), która zamiast separowania pola dla każdej właściwości może pełnić rolę delegata dla wielu właściwości. Klucze elementów mapy muszą mieć taka samą nazwę co właściwość.

{% highlight kotlin %}
class Worker(var map: MutableMap<String, Any?>) {
    var name: String by map
    var salary: Int by map
}

//execute constructor with init values
var map: MutableMap<String, Any?> = mutableMapOf(
    "name" to "William",
    "salary" to 2000
)
val worker = Worker(map)

print(worker.name) //William
print(worker.salary) //2000
map.set("salary", 3000)
println(worker.salary) //3000
{% endhighlight %}