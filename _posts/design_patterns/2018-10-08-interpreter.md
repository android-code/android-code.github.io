---
layout: post
title: "Interpreter"
date:  2018-10-08
categories: ["Wzorce projektowe"]
permalink: /blog/wzorce/:title/
image: patterns/interpreter
github: design-patterns/tree/master/interpreter
description: "Wzorce projektowe / behawioralny"
keywords: "interpreter, wzorzec, wzorce projektowe, wzorzec behawioralny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`Interpreter` (ang. `Interpreter`) (wzorzec behawioralny) jest sposobem na zdefiniowanie oraz dokonanie oceny gramatyki języka interpretowanego. Wzorzec oparty jest o hierarchię wyrażeń (`symboli terminalnych i nieterminalnych`) i na podstawie struktury drzewa zbudowanych zasad przeprowadzana jest interpretacja wyrażenia wejściowego. W rezultacie otrzymywane jest wyrażenie wyjściowe, podejmowana jest akcja lub zmienia się stan obiektu.

## Ograniczenia
W przypadku żłożonej gramatyki może powstać zbyt wiele klas (przynajmniej jedna dla każdej reguły). Takie gramatyki są trudne do zarządzania i utrzymania. Jako alternatywe należy rozważyć zastosowanie `parsera`.

## Użycie
Wzorzec `Interpreter` wykorzystywany jest w celu interpretowania zdań zapisanych w pewnym języku o prostej gramatyce, które mogą przyjąć reprezentacje drzewa składniowego. `Interpreter` może być użyty w konwerterach notacji, przetwarzaniu języka naturalnego (`wyrażenia regularne`), automatach formalnych (`kompilatory`) czy też w logice (`sprawdzanie poprawności reguł`).

## Implementacja
Klasy wyrażeń implementują interfejs `Expression` w taki sposób, aby móc zinterpretować wyrażenie wejściowe i przekstałcić je na odpowiednie wyrażenie wyjściowe. Implementacja powinna być odporna na błędny format wprowadzonego zapytania. Klasy wyrażeń mogą być symbolami terminalnymi i nieterminalnymi.

![Interpreter diagram](/assets/img/diagrams/patterns/interpreter.svg){: .center-image }

Poniższy listing przedstawia implementacje wzorca `Interpreter` dla koniukcji wyrażeń.

{% highlight java %}
public class TerminalExpression implements Expression {
	
    //some fields
    private String data;

    @Override
    public boolean interpret(Context context) {
        //do something specific for TerminalExpression e.g. contains text
        return context.getText().contains(data);
    }
}

public class NonTerminalExpression implements Expression {
	
    private TerminalExpression expression1;
    private TerminalExpression expression2;

    public NonTerminalExpression(TerminalExpression expr1, TerminalExpression expr2) {
        this.expression1 = expr1;
        this.expression2 = expr2;
    }

    @Override
    public boolean interpret(Context context) {
        //do something specific for NonTerminalExpression e.g. AndExpression
        return expression1.interpret(context) && expression2.interpret(context);
    }
}

interface Expression {
	
    boolean interpret(Context context);
}

public class Context {
	
    //some fields
    private String text;

    //constructor and methods
    public Context(String text) {
        this.text = text;
    }

    public String getText() {
        return text;
    }
}
{% endhighlight %}

Klient tworzy reguły wykorzystując klasy wyrażeń. Następnie wprowadza dane dla których sprawdzane są reguły.

{% highlight java %}
//create concrete rule based on brothers expression rule
Expression areBrothers = getBrothersExpression("Jack", "Johnnie");

//check if input implements the rule
areBrothers.interpret("Jack is Johnnie's brother"); //true
areBrothers.interpret("Jack has one brother. His name is John"); //false

//brothers expression rule
public static Expression getBrothersExpression(String name1, String name2) {
    Expression person1 = new TerminalExpression(new Context(name1));
    Expression person2 = new TerminalExpression(new Context(name2));
    Expression brother = new TerminalExpression(new Context("brother"));

    Expression names = new NonTerminalExpression(person1, person2);
    return new NonTerminalExpression(names, brothers);
}
{% endhighlight %}

## Przykład
Aplikacja `GuessColor` jest grą w której sprawdzane są umiejętności znajomości kolorów zapisanych heksadecymalnie. Gracz na wejściu otrzymuje informacje o kolorach w formacie szesnastkowym i przed upływem ustalonego czasu ma za zadanie odgadnąć ich reprezentacje w notacji RGB lub na odwrót (z zapisu RGB odgaduje format heksadecymalny). W celu dokonania tłumaczeń między formatami zastosowano wzorzec `Interpreter`. 

{% highlight java %}
public class HexToRgb implements Expression {

    @Override
    public String interpret(String number) {
        Pattern pattern = Pattern.compile("#([0-9a-f]{6}", Pattern.CASE_INSENSITIVE);
        Matcher matcher = pattern.matcher(number);
        if (matcher.matches()) {
            Expression hexToInt = new HexToInt();
            String red = hexToInt.interpret(number.substring(1,3));
            String green = hexToInt.interpret(number.substring(3,5));
            String blue = hexToInt.interpret(number.substring(5,7));
            if(!red.contains("Invalid") && !green.contains("Invalid") && !blue.contains("Invalid"))
                return "#" + red + green + blue;
            else
                return "Invalid input. Should be #rrggbb";
        }
        else
            return "Invalid input. Should be #rrggbb";
    }
}

public class RgbToHex implements Expression {

    @Override
    public String interpret(String number) {
        Pattern pattern = Pattern.compile("rgb *\\( *([0-9]+), *([0-9]+), *([0-9]+) *\\)");
        Matcher matcher = pattern.matcher(number);
        if (matcher.matches()) {
            Expression intToHex = new IntToHex();
            String red = intToHex.interpret(matcher.group(1));
            String green = intToHex.interpret(matcher.group(2));
            String blue = intToHex.interpret(matcher.group(3));
            if(!red.contains("Invalid") && !green.contains("Invalid") && !blue.contains("Invalid"))
                return "rgb(" + red + green + blue + ")";
            else
                return "Invalid input. Should be rgb(int, int, int)";
        }
        else
            return "Invalid input. Should be rgb(int, int, int)";
    }
}

public class HexToInt implements Expression {

    @Override
    public String interpret(String number) {
        Pattern pattern = Pattern.compile("([0-9a-f]{2}", Pattern.CASE_INSENSITIVE);
        Matcher matcher = pattern.matcher(number);
        if (matcher.matches()) {
            return Integer.parseInt(number, 16);
        }
        else
            return "Invalid input. Should be [0-9a-f]{2}";
    }
}

public class IntToHex implements Expression {

    @Override
    public String interpret(String number) {
        Pattern pattern = Pattern.compile("([0-9]{1,3}");
        Matcher matcher = pattern.matcher(number);
        if (matcher.matches() && Integer.parseInt(number) <= 255) {
            return Integer.toHexString(number);
        else
            return "Invalid input. Should be a number between 0-255";
    }
}

interface Expression {
	
    String interpret(String number);
}
{% endhighlight %}

Na podstawie wylosowanych kolorów, gracz próbuje zgadnąć ich zapis w formacie heksadecymalnym lub RGB.

{% highlight java %}
//user starts the game
Expression rgbToHex = new RgbToHex();
Expression hexToRgb = new HexToRgb();

//user solves the tasks
rgbToHex.interpret("rgb(50, 100, 200)"); //interpreted as #3264C8
hexToRgb.interpret("#123ABC"); //interpreted as rgb(18,58,188)
{% endhighlight %}

## Biblioteki
Przykładem realizacji wzorca `Interpreter` w standardowym pakiecie `Java` są wszystkie implementacje klasy `Format` jak np.: `DateFormat`, `NumberFormat` oraz klasa wyrażeń regularnych `Pattern`.