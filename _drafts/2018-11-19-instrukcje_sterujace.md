---
layout: post
title: "Instrukcje sterujące"
date:  2018-11-19
categories: ["Kotlin"]
image: kotlin/control_flow
github: kotlin/blob/master/control-flow.kt
description: "Kotlin"
keywords: "kotlin, instrukcje sterujące, control flow, pętla, for, while, loop, when, if, else, range, break, continue, return, android, programowanie, programming"
---

## Instrukcje if-else
Instrukcja warunkowa `if` jest wyrażaniem - zwraca wartość logiczną. Blok instrukcji warunowych `if`, `else if`, `else` działa podobnie jak w `Javie`. Jeśli blok operacji dla spełnionenego (lub nie) warunku zawiera się w jednej linii wówczas klamry są opcjonalne.

{% highlight kotlin %}
var a = 1
var b = 2
var c = 0

//standard way
if(a < b) 
    c = a
else if(a > b) 
    c = b
else {
    //do something
    c = a + b
}

//as expression
c = if(a < b) a 
    else if(a > b) b 
    else { //else branch is required when if is using as expression
        //do something
        a+b //note that last expression must be returning value
    }
{% endhighlight %}

## Wyrażenie range
Wyrażania zakresu służą do sprawdzania czy dana wartość znajduje się w przedziale wartości, a także mogą pełnić rolę iteratora dla zmiennej na zadanym przedziale. Wyrażenia zakresu są zdefiniowane dla typów `Comparable`, natomiast typy proste (takie jak `Int`, `Long`, `Char`) posiadają zoptymalizowaną implementacje. 

{% highlight kotlin %}
var a = 3
if(a in 1..5) print("a is in 1-5 range") //1<=a && a<=5
if(a !in 0..9) print("a is not a single number") //0>a || a>9
{% endhighlight %}

## Operator when
W `Kotlin` operator `when` jest tym samym czym `switch` w `Java` - dopasowuje on argument (w sposób sekwencyjny) do gałęzi warunków dopóki nie jeden z warunków zostanie spełniony. Podobnie jak instrukcje `if-else` może zostać użyty w formie wyrażenia.

{% highlight kotlin %}
var a = 3
when(a) {
    1, 2, 3 -> print("a is in small range")
    in 4..10 -> print("a is in big range")
    0 -> print("a is zero")
    else -> { 
        print("a is not in range")
    }
}
{% endhighlight %}

## Pętla for
Iteruje po elementach instancji klas, które dostarczają implementacji `Iterator`. Działanie pętli jest odpowiednikiem pętli `for-each` z Javy.

{% highlight kotlin %}
var array = ArrayOf(1, 2, 3)
for(item: Int in array) {
    print(item)
}

//or just
for(item in array) print(item)

//or iterate by indices
for(i in array.indices) print(array[i])
{% endhighlight %}

Iteracja w pętli for może odbywać się także przy wykorzystaniu wyrażenia `range`.

{% highlight kotlin %}
for(i in 1..3) print(i) //1,2,3
for(i in 1 until 3) print(i) //1,2,3
for(i in 5..3) print(i) //5,4,3
for(i in 5 downTo 3) print(i) //5,4,3
for(i in 1..10 step 2) print(i) //1,3,5,7,9
{% endhighlight %}

## Pętla while
Działanie i konstrukcja pętli `while` oraz `do..while` przypomina tą znaną z Java. Obie pętle wykonują się tak długo dopóki warunek jest spełniony. Programista sam musi zadbać o zwiększanie się iteratora lub zmianę wartości logicznej warunku. W przypadku pętli do-while jest gwarancja wykonania instrukcji przynajmniej raz.

{% highlight kotlin %}
var i = 1
while(i <= 5) {
    print(i)
    i++
}

var j = 0
do {
    print(j)
    j++
} while(j < 0)
{% endhighlight %}

## Instrukcje skoku
Kotlin podobnie jak większość języków dostarcza standardowych instrukcji wyjścia: `break` (wychodzi z ciała bieżącej pętli lub warunku) oraz `continue` (wychodzi z bieżącego kroku bieżącej pętli lub warunku). 

{% highlight kotlin %}
var a = 9
for(i in 1..10) {
    if(i == a) break
    if(i % 2 == 0) continue
    print(i)
}
{% endhighlight %}

Instrukcje skoku mogą zostać użyte wraz z `etykietą` pętli do której się odnoszą.

{% highlight kotlin %}
loop@ for(i in 1..10) {
    for(j in 1..10) {
        if (i*j % 2 == 0) continue@loop
        print(" " + i*j)
    }
}
{% endhighlight %}

Instrukcja `return` zwracająca wartość funkcji może współpracować z etykietami dzięki czemu znajduje zastosowanie jak innstrukcja wyjścia w funkcjach i przede wszystkim wyrażeniach `lambda`.

{% highlight kotlin %}
listOf(1, 2, 3, 4, 5).forEach {
    if(it %2 == 0) return // non-local return directly to the caller of foo()
    print(" " + it)
}
//returned from entire caller function body so this point is unreachable

listOf(1, 2, 3, 4, 5).forEach loop@{
    if(it %2 == 0) return@loop 
    print(" " + it)
}
//returned only from the local caller - foreach loop, not from caller function so this point is reachable
{% endhighlight %}