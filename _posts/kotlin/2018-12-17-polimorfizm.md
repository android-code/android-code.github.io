---
layout: post
title: "Polimorfizm"
date:  2018-12-17
categories: ["Kotlin"]
image: kotlin/polymorphism
github: kotlin/blob/master/polymorphism.kt
description: "Kotlin"
keywords: "kotlin, polimorfizm, dziedziczenie, interfejsy, abstrakcja, klasa, inheritance, polymorphism, interface, abstract, class, open, override, extensions, android, programowanie, programming"
---

## Dziedziczenie
Wszystkie klasy w `Kotlin` `dziedziczą` po superklasie `Any` (podobnie jak `Object` w `Java`), która dostarcza dla klas podtypów metody `equals`, `hashCode` oraz `toString`. Klasa, która może być dziedziczona musi być oznaczona jako `open`, a deklaracja dziedziczenia następuje w nagłówku klasy. Jeśli klasa ma konstruktor podstawowy (primary constructor) wówczas inicjalizacja klasy bazowej musi nastąpić w konstruktorze podstawowym klasy dziedziczącej. Gdy klasa nie posiada konstruktora podstawowego to we wszystkich jej konstruktorach dodatkowych (secondary constructor) musi przebiegać bezpośrednio inicjalizacja typu bazowego za pomocą słowa kluczowego `super` lub pośrednio poprzez delegacje do innego konstruktora. 

{% highlight kotlin %}
//with primary constructor
open class ParentPrimary(number: Int)
class ChildPrimary(number: Int) : Parent(number)

//with secondary constructor
open class ParentSecondary {
    var number: Int
    constructor(number: Int) {
        this.number = number
    }
}

class ChildSecondary : ParentSecondary {
    var text: String
    constructor(number: Int) : super(number) {
        this.text = ""
    }
    constructor(number: Int, text: String) : super(number) {
        this.text = text
    }
}
{% endhighlight %}

W procesie tworzenia obiektu klasy pochodnej wywoływany jest najpierw konstruktor klasy bazowej co ma miejsce przed inicjalizacją nadpisanych i nowych właściwości klasy pochodnej. Jeśli któraś z tych właściwości jest używana w procesie inicjalizacji klasy bazowej może to prowadzić do nieoczkiewanego zachowania lub błędu programu w czasie jego działania. Aby zapobiec takim sytuacją projektując klasę bazową należy unikać używania właściwości open w konstruktorze, inicjalizatorów właściwości oraz bloków `init`.

## Elementy klasy
Metody oraz właściwości, które zezwalają na nadpisywanie w klasach pochodnych podobnie jak klasy wymagają słowa kluczowego `open`, a w miejscu ich nadpisywania adnotacji `override`. Nadpisane metody i właściwości stają się jednocześnie otwarte na modyfikacje (oznaczone jako open), chyba że w ich deklaracji zostanie użyty modyfikator `final`. Ponadto nadpisywanie właściwości może odbywać się przez inicjalizator lub właściwość z metodą dostepową, a zmienne z modyfikatorem `val` mogą być nadpisane jako `var` (lecz nie na odwrót).

{% highlight kotlin %}
//classes derived from Parent class can override text property and action1, action2 functions
open class Parent {
    open val text: String = "parent"
        get() = field.capitalize()
    open fun action1() { print("base action1") }
    open fun action2() { print("base action2") }
    fun showText() { print(text) }
}

//overriding properties can be part of the primary constructor
//any class can derived from Child class because it's not open
//but if Child class will be open then classes derived can override only action1
class Child(final override var text: String) : Parent() {
    override fun action1() { print ("overriden action1") }
    final override fun action2() { print ("final overriden action2") }
}

//execution
var parent = Parent()
var child = Child("child")
parent.showText() //Parent
child.showText() //child
parent.action1() //base action1
child.action1() //overriden action1
{% endhighlight %} 

## Wielokrotne dziedziczenie
Podobnie jak w języku `C++` (przeciwnie do `Java`) w `Kotlin` możliwe jest wielokrotne dziedziczenie. Klasa pochodna może wywoływać funkcje i właściwości klasy bazowej poprzez użycie słowa kluczowego `super`. W przypadku, gdy klasa dziedziczy wiele implementacji tego samego elementu klasy wówczas musi go nadpisać, a aby odwołać się do implementacji konkretnej klasy bazowej należy użyć instrukcji `super<NazwaKlasy>`.

{% highlight kotlin %}
open class A {
    open fun action1() { print("action1 A") }
}

open class B {
    open fun action1() { print("action1 B") }
    open fun action2() { print("action2 B") }
}

class C() : A(), B() {
    override fun action1() { //action1 fun must be overriden
        super<A>.action1()
        print(" from C")
    }
    fun action3() { print("action3 C") }
}

//execution
var a = A()
var b = B()
var c = C()
a.action1() //action1 A
b.action1() //action1 B
c.action1() //action1 A from C
c.action2() //action2 B
c.action3() //action3 C
{% endhighlight %}

## Klasa abstrakcyjna
Klasy oraz jej elementy mogą być w Kotlin `abstrakcyjne` poprzez zadeklarowanie ich jako `abstract`. Jeśli klasa jest abstrakcyjna wówczas nie może posiadać żadnej instancji, natomiast funkcja abstrakcyjna nie posiada implementacji. Klasy abstrakcyjne są automatycznie oznaczone jako open. W przeciwieństwie do Java klasa abstrakcyjna nie musi posiadać elementu oznaczonego jako abstract. Klasa dziedzicząca musi nadpisać wszystkie abstrakcyjne elementy klasy bazowej.

{% highlight kotlin %}
abstract class Abstraction {
    fun action1() { print("abstract action1") }
    abstract fun action2() //must be overriden
}
open class Concrete : Abstraction() {
    override fun action2() { print("action2 implementation") }
}
{% endhighlight %}

## Interfejsy
`Interfejsy` w Kotlin podobnie jak w Java wprowadzają poziom abstrakcji w implementacji oraz sprawiają, że od grupy różnego typu obiektów można oczekiwać wspólnego zachowania. Deklaruje się je za pomocą słowa kluczowego `interface`. W przeciwieństwie do klas abstrakcyjnych nie przechowują stanu. Mogą zawierać deklaracje metod abstrakcyjnych i metody z domyślną implementacją, a także właściwości abstrakcyjne lub właściwości z implementacją akcesorów. Klasa implementująca interfejs musi nadpisać wszystkie elementy abstrakcyjne interfejsów, które nie posiadają implementacji. 

{% highlight kotlin %}
interface SomeInterface {
    val text: String //is abstract so must be overriden
    fun action1() //must be implement
    fun action2() {
      print(text + " from interface implementation")
    }
}

class SomeClass : SomeInterface {
    override val text: String = "value"
    override fun action1() {
        print("action1 implementation")
    }
}

var obj: SomeInterface = SomeClass()
obj.action1() //action1 implementation
obj.action2() //value from interface implementation
{% endhighlight %}

Interfejs może również dziedziczyć po innych interfejsach i tak samo jak w przypadku dziedziczenia klas, klasa implementująca interfejsy o tych samych elementach musi je nadpisać, a aby odwołać się do implementacji konkretnego interfejsu należy użyć instrukcji `super<NazwaInterfejsu>`.

{% highlight kotlin %}
interface IA {
    fun action1() //this is abstract
    fun action2() { print("action2 IA") }
}

interface IB : IA {
    fun action3() { print("action3 IB") }
}

interface IC {
    fun action2() { print("action2 IC") }
}

class ClassInterface : IA, IC {
    override fun action1() { print("action1 ClassInterface") }
    override fun action2() { 
        super<IC>.action1()
        print(" from ClassInterface") 
    }
}

var obj: IB = ClassInterface()
obj.action1() //action1 ClassInterface
obj.action2() //action2 IA from ClassInterface
obj.action3() //action3 IB
{% endhighlight %}

## Rozszerzenia
Kotlin dostarcza mechanizmu deklaracji `rozszerzenia` (`extensions`), które umożliwiają rozszerzenie funkcjonalności klas (funkcje i właściwości) bez ich dziedziczenia czy stosowania wzorca typu Dekorator. Nierzadko są one wykorzystywane jako alternatywa dla klas typu Utils. Aby zadeklarować funkcję lub właściwość należy poprzedzić jej nazwę typem odbiorcy rozszerzenia. W ten sam sposób można deklarować i wywołać rozszerzenia dla `companion object`. Słowo kluczowe `this` odnosi się do bieżącego obiektu odbiorcy. Typ odbiorcy rozszerzenia może także `nullable`.

{% highlight kotlin %}
fun Int?.multiply(number: Int): Int {
    if(this == null) return 0
    return this*number //autocast to non-null
}
val String.even: Boolean
    get() = length%2 == 0

var number: Int = 5
var text: String = "text"
print(number.multiply(3)) //15
print(text.even) //true
{% endhighlight %}

Rozszerzenia nie modyfikują ani nie rozszerzają klasy, są wywoływane `statycznie` i związane są z typem wyrażenia dla którego wywoływana jest funkcja, a nie typem argumentu. Rozszerzenie o tej samej sygnaturze co członek klasy nie powoduje jego nadpisania.

{% highlight kotlin %}
open class A
class B: A()
class C {
    fun show() = print("member")
}
fun A.show() = print("A")
fun B.show() = print("B")
fun C.show() = print("extension")
fun showInfo(obj: A) = obj.show()

showInfo(B()) //A printed
C().show() //member printed
{% endhighlight %}

Rozszerzenia dla jednej klasy mogą być definiowane w ciele innej klasy co sprawia, że stają się członkiem klasy. Co więcej mogą być one zadeklarowane jako `open` i nadpisane w klasach pochodnych. Instancja klasy w której jest definiowane rozszerzenie nazywa się `dispatch receiver`, a obiekt typu odbiornika `extension receiver`.

{% highlight kotlin %}
class ExtensionReceiver {
    fun work() { print("some work") }
}
open class DispatchReceiver {
    fun action() { print("ExtensionReceiver.action() from dispatch receiver class") }
    open fun ExtensionReceiver.action() {
        work() //from ExtensionReceiver class
        println("")
        this@DispatchReceiver.action()
    }
    fun caller(obj: ExtensionReceiver) {
        obj.action()
    }
}
class DerivedClass: DispatchReceiver() {
    override fun ExtensionReceiver.action() {
        print("ExtensionReceiver.action() function from derived class")
    }
}

var extension = ExtensionReceiver()
var dispatch = DispatchReceiver()
var derived = DerivedClass()
dispatch.caller(extension) //some work ExtensionReceiver.action() from dispatch receiver class
derived.caller(extension) //ExtensionReceiver.action() function from derived class
{% endhighlight %}