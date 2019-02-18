---
layout: post
title: "Dynamic Links"
date: 2019-05-06
categories: ["Firebase"]
image: firebase/dynamic_links
github: firebase/tree/master/dynamic_links
description: "Firebase"
version: Firebase-Dynamic-Links 16.1
keywords: "firebase, link, łącze, internetowe, odnośnik, dynamiczne, dynamic, deep, web, udostępnianie, share, android, programowanie, programming"
---

## Przeznaczenie
`Dynamic Links` jest sposobem na generowanie i przetwarzanie dynamicznych łączy internetowych działających w dowolnie zaprojektowany sposób niezależnie od platformy. W przypadku uruchomienia łącza na urządzeniu z systemem `Android` i zainstalowaną aplikacją obsługującą ustalony format Dynamic Links otwierany jest oczekiwany ekran z podaną zawartością. Gdy urządzenie nie posiada wskazanej aplikacji lub usług `Play Services` wówczas użytkownik może zostać przekierowany do właściwej strony `Play Store` w celu zainstalowania brakującej aplikacji. Jeśli łącze zostanie otworzone w przeglądarce urządzenia bądź komputera nastąpi wyświetlenie strony internetowej. Dynamic Links mogą zostać wykorzystane m.in. w zadaniu konwersji użytkowników z internetowej do mobilnej wersji aplikacji, przeprowadzaniu kampanii marketingowej, wysyłaniu zaproszeń do korzystania z aplikacji czy też dzieleniu treści z innym użytkownikami. Tworzenie łączy może odbywać się z poziomu `konsoli Firebase`, ręczne formowanie `URL` z parametrami, programowo z kodzie aplikacji przy użyciu `Android API` czy `REST API`.

## Tworzenie
Generowania łącza z poziomu konsoli Firebase pozwala na wybranie niestandardowego linku oraz śledzenia skuteczności co z uwagi na łatwą konfiguracje jest przydatne przede wszystkim w sytuacji tworzenia odnośnika promocyjnego do udostępnienia w mediach społecznościowych. Korzystanie z `Dynamic Link Builder API` po stronie kodu aplikacji jest preferowanym sposobem w większości sytuacji, a przede wszystkim w zadaniach udostępniania i przesyłania treści między użytkownikami oraz tam gdzie potrzebne jest wiele linków. Jeśli projekt nie wymaga śledzenia i analizy danych, łącza mogą zostać stworzone także ręcznie przy wykorzystaniu parametrów w adresie URL co pozwala na minimalizacje ruchu sieciowego.

## Budowniczy
Aby utworzyć obiekt dynamicznego łącza `DynamicLink` przy użyciu API budowniczego należy wywołać metodę `createDynamicLink`, dokonać konfiguracji dla wybranych platform i następnie wywołać `buildDynamicLink`.

{% highlight kotlin %}
private fun prepareDynamicLink() {
    val dynamicLink = FirebaseDynamicLinks.getInstance().createDynamicLink()
        .setLink(Uri.parse("http://www.androidcode.pl/"))
        .setDomainUriPrefix("http://androidcode.page.link")
        //set parameters for target platforms like Android, iOS, GoogleAnalytics or provide social media metatags
        .setAndroidParameters(DynamicLink.AndroidParameters.Builder("pl.androidcode")
            .setMinimumVersion(2) //only for apps with version 2 code
            .build())
        .setSocialMetaTagParameters(DynamicLink.SocialMetaTagParameters.Builder()
            .setTitle("Title")
            .setDescription("Description")
            .build())
        .buildDynamicLink()

    val dynamicLinkUri = dynamicLink.uri

    //the link could be like below:
    //http://androidcode.page.link/?link=http://www.androidcode.pl&apn=pl.androidcode&amv=2&st=title&sd=description
}
{% endhighlight %}

Skrócona wersja odnośnika składa się domyślnie 17 znakowego unikalnego sufiksu co wymaga zapytania sieciowego oraz obiektu słuchacza i tworzona jest przy użyciu metody `buildShortDynamicLink`.

{% highlight kotlin %}
private fun prepareShortDynamicLink() {
    val shortLinkTask = FirebaseDynamicLinks.getInstance().createDynamicLink()
        .setLink(Uri.parse("http://www.androidcode.pl/"))
        .setDomainUriPrefix("http://androidcode.page.link")
        .setAndroidParameters(DynamicLink.AndroidParameters.Builder("pl.androidcode").build())
        .buildShortDynamicLink(ShortDynamicLink.Suffix.SHORT) //pass this arg to get shorten sufix
        .addOnSuccessListener { result ->
            val shortLink = result.shortLink
            val previewLink = result.previewLink
        }.addOnFailureListener {
            //some action
        }

    //the link could be like below:
    //http://androidcode.page.link/abcd
}
{% endhighlight %}

## Odbieranie
Aby odebrać i przetworzyć otrzymany `DynamicLink` należy dodać w `AndroidManifest` poniższy wpis konfiguracyjny `intent-filter` do aktywności odpowiedzialnej za przechwytywanie linków (`Deep Link` i `App Link`), wywołać metodę `getDynamicLink` oraz wykonać akcję na podstawie odebranych danych. Należy mieć na uwadzę, że wykonanie metody `getDynamicLink` czyści przekazane dane więc jej użycie jest jednorazowe. Ponadto dodanie w `AndroidManifest` obsługi `Android App Links` umożliwia pominięcie dialogu wyboru aplikacji która ma obsłużyć odnośnik i bezpośrednie skierowanie do wyznaczonej aplikacji (co w przypadku `Deep Links` może nie być możliwe).

{% highlight xml %}
<!-- autoVerify flags is responsible for handling Dynamic Links using App Links -->
<!-- add filter to MainActivity -->
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <category android:name="android.intent.category.BROWSABLE"/>
    <data android:host="androidcode.pl" android:scheme="http"/>
</intent-filter>
{% endhighlight %}

{% highlight kotlin %}
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        receiveDynamicLink() 
    }

    private fun receiveDynamicLink() {
        FirebaseDynamicLinks.getInstance()
            .getDynamicLink(intent)
            .addOnSuccessListener(this) { pendingDynamicLinkData ->
                // Get deep link from result (may be null if no link is found)
                var deepLink: Uri? = null
                if (pendingDynamicLinkData != null) {
                    deepLink = pendingDynamicLinkData.link
                }
            }.addOnFailureListener {
                //some action
            }
    }
}
{% endhighlight %}

## Indeksowanie aplikacji
`App Indexing` jest usługą `indeksowania aplikacji` do wyników wyszukiwarki `Google`, gdzie wybrane strony witryny internetowej zostają powiązane z treściami zawartymi w aplikacji mobilnej. Dzięki temu zapytania użytkowników mogą być przetworzone w taki sposób, aby bezpośrednio kierować użytkownika do wybranego ekranu w aplikacji, `Sklepu Play` lub standardowo do witryny internetowej. Indeksowanie aplikacji ułatwia przyciąganie potencjalnych nowych użytkowników, którzy mogą dowiedzieć się o istnieniu aplikacji z wyników wyszukiwarki, a bieżącym użytkownikom pozwala na zwiększenie jakości odbioru korzystania z aplikacji (`User Experience`). Aby indeksować zawartośc aplikacji przez Google należy po stronie witryny dokonać konfiguracji adresów `URL` obsługiwanych przez mechanizm `Dynamic Links` w aplikacji. 
