---
layout: post
title: "Analiza statyczna"
date:  2019-02-18
categories: ["Testowanie"]
image: testing/static_analysis
github: testing/tree/master/static_analysis
description: "Testowanie"
keywords: "testowanie, testing, testy, analiza, statyczna, gradle, lint, pmd, findbugs, sonarqube, quality, bugs, guideline, continous integration, integracja, android, programowanie, programming"
---

## Definicja
`Statyczna analiza kodu` wykorzystywana jest w celu znajdywania błędów w kodzie, wyszukiwanie luk i niedopatrzeń oraz sprawdzania czy napisany kod spełnia zasady dobrego programowania i przestrzega wytycznych. Co więcej pozwala na detekcję hipotetycznych błędów, które mogły nie zostać wykrytę przez testy jednostkowe i manualne, a także ułatwia definiowanie i przestrzeganie `standardów kodu` w zespole. Dzięki temu utrzymanie odpowiedniej jakości kodu staje się łatwiejsze. Statyczna analiza jest częścią procesu `ciągłej integracji` (`continous integration`) i może być wykorzystywana również jako narzędzie wspierające `testowanie` (`białoskrzynkowe`) oraz `refactoring`. Dokonywana analiza przeprowadzana jest bez wykonania kodu i odbywa się przy pomocy różnych narzędzi analitycznych takich jak np. `lint`, `checkstyle`, `pmd` czy `findbugs`  dostępnych jako `plugin` dla `Gradle` oraz platform wspierających ciągłą integracje jak np. `SonarQube`. Weryfikują one zgodność kodu w zestawieniu do zbioru reguł. W przypadku niespełnienia zasad informują o potencjalnych błędach i zagrożeniach. Nie rzadko różne pluginy zawierają podobną funkcjonalność co nie wyklucza ich ze wzajemnego użycia, wręcz przeciwnie, uzupełniają się.

## Checkstyle
`Checkstyle` analizuje kod źródłowy weryfikując jego standardy oraz konwencję w stosunku do zbioru wybranych zasad. Skupia się przede wszystkim na analizie stylu kodu (np. nazewnictwo, klamry). Aby stworzyć konfigurację dla checkstyle należy dodać plik zasad (np. `checkstyle.xml`) oraz uzupełnić plik `gradle` aplikacji (lub stworzyć nowy) o nowe zadanie co przedstawia poniższy listing.

{% highlight xml %}
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC "-//Puppy Crawl//DTD Check Configuration 1.3//EN"
    "http://www.puppycrawl.com/dtds/configuration_1_3.dtd">

<module name="Checker">
    <!-- more modules and properties -->

    <module name="TreeWalker">

        <module name="MethodName"/>
        <module name="ConstantName"/>
        <module name="UnusedImports"/>
        <module name="ParameterNumber">
            <property name="max" value="2"/>
        </module>

        <!-- more modules and properties -->
    </module>

</module>
{% endhighlight %}

{% highlight gradle %}
apply plugin: 'checkstyle'

task checkstyle(type: Checkstyle) {
    description 'checkstyle analyze'
    group 'verification'
    configFile file('./analyze/checkstyle.xml')
    source 'src/main/java'
    include '**/*.java'
    exclude '**/gen/**'
    classpath = files()
}
{% endhighlight %}

Wykonane zadanie `checkstyle` dla klasy `CheckstyleCode` zwróci raport z uwagami dotyczącymi nie używanego import, nie poprawnych nazw dla stałej i metody oraz zbyt dużą ilość argumentów metody.

{% highlight java %}
import java.math.BigInteger; //unused import

public class CheckstyleCode {

    public final static int someConstant = 1; //should be SOME_CONSTANT instead

    public void some_method() { //should be someMethod instead
        //body
    }

    public void methodTooManyParameters(int a, int b, int c) { //too many parameters - max 2
        //body
    }
}
{% endhighlight %}

## Pmd
`Pmd` podobnie jak checkstyle przeprowadza analizę kodu w stosunku do zbioru zasad, jednakże koncentruje się na newralgicznych i opuszczonych fragmentach kodu (np. nieużywane zmienne, puste bloki). Aby stworzyć konfigurację dla pmd należy dodać plik zasad (np. `pmd.xml`) oraz uzupełnić plik `gradle` aplikacji o nowe zadanie co przedstawia poniższy listing.

{% highlight xml %}
<?xml version="1.0"?>
<ruleset xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         name="PMD rules"
         xmlns="http://pmd.sourceforge.net/ruleset/2.0.0"
         xsi:schemaLocation="http://pmd.sourceforge.net/ruleset/2.0.0 http://pmd.sourceforge.net/ruleset_2_0_0.xsd">

    <exclude-pattern>.*/R.java</exclude-pattern>
    <exclude-pattern>.*/gen/.*</exclude-pattern>

    <rule ref="rulesets/java/unusedcode.xml"/>
    <rule ref="rulesets/java/empty.xml"/>
    <rule ref="rulesets/java/braces.xml"/>
</ruleset>
{% endhighlight %}

{% highlight gradle %}
apply plugin: 'pmd'

task pmd(type: Pmd) {
    description 'pmd'
    group 'verification'
    ruleSetFiles = files("./analyze/pmd.xml")
    source 'src/main/java'
    include '**/*.java'
    exclude '**/gen/**'
}
{% endhighlight %}

Wykonane zadanie `pmd` dla klasy `PmdCode` zwróci raport z uwagami dotyczącymi nie używanej zmiennej, pustej metody, braku klamr dla pętli oraz bloku warunkowego zawsze prawdziwego.

{% highlight java %}
public class PmdCode {

    public void someMethod() {
        int variable = 1; //unused variable

        if(true) {
            //always true
        }

        for(int i=0; i<=1; i++)
            emptyMethod(); //no braces in loop
    }

    public void emptyMethod() {
    }
}
{% endhighlight %}

## Findbugs
`Findbugs` działając w oparciu o kod bajtowy wykonuje analizę w poszukiwaniu potencjalnych problemów z listy znanych błędów projektowych i złych praktyk (np. jawnie zapisane hasło, klasa testowa nie posiada testów, metoda prywatna nie jest nigdy wywoływana). Aby stworzyć konfigurację dla findbugs należy dodać plik filtrów (np. `findbugs.xml`) oraz uzupełnić plik `gradle` aplikacji o nowe zadanie co przedstawia poniższy listing.

{% highlight xml %}
<FindBugsFilter>

    <Match><Class name="~.*R\$.*"/></Match>
    <Match><Class name="~.*Manifest\$.*"/></Match>

</FindBugsFilter>
{% endhighlight %}

{% highlight gradle %}
apply plugin: 'findbugs'

task findbugs(type: FindBugs) {
    description 'findbugs'
    group 'verification'
    excludeFilter file('./analyze/findbugs.xml')
    classes = files("$project.buildDir/intermediates/javac")
    source 'src/main/java'
    effort 'max'
    classpath = files()
}
{% endhighlight %}

Wykonane zadanie `findbugs` dla kodu bajtowego klasy `FindbugsCode` zwróci raport z uwagami dotyczącymi nie używanej metody prywatnej, porównywania obiektów typu `String` za pomocą operatora == oraz ignorowanie rezultatu metody.

{% highlight java %}
public class FindbugsCode {

    public void someMethod() {
        //boxed primitive allocated only for String value
        String text = new Integer(1).toString(); //should use just Integer(1).toString()
        equalityMethod(text, "b");
    }

    private boolean equalityMethod(String a, String b) {
        boolean equality = (a == b); //== instead of equals
        return equality;
    }

    private void unusedPrivateMethod() {
    }
}
{% endhighlight %}

## Lint
`Lint` jest narzędziem dostarczonym przez `Android Studio`, który podobnie jak wspomniane wcześniej pluginy sprawdza pliki źródłowe pod kątem potencjalnych błędów, optymalizacji poprawności, bezpieczeństwa, wydajności, użyteczności i dostępności kodu. Z uwagi na integracje ze środowiskiem programistycznym nie wymaga dodatkowej konfiguracji.

//TODO

## SonarQube
//TODO