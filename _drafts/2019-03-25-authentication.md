---
layout: post
title: "Authentication"
date:  2019-03-25
categories: ["Firebase"]
image: firebase/authentication
description: "Firebase"
keywords: "firebase, autentykacja, autoryzacja, authentication, account, user, email, password, phone, google, facebook, twitter, github, rejestracja, logowanie, sign in, log in, sign up, log out, android, programowanie, programming"
---

## Wprowadzenie
Tworzenie i zarządzania kontem użytkownika to dla większości aplikacji podstawowa funkcjonalność, która pozwala na personalizowane wykorzystanie funkcjonalności aplikacji, a znajomość tożsamości ułatwia bezpieczne zapisywanie i powiązanie danych z kontem synchronizowanym na różnych urządzeniach. `Firebase Authentication` dostarcza łatwych w użyciu usług `backend` przeznaczonych do uwierzytelniania użytkowników w aplikacji za pomocą `hasła`, `numeru telefonu`, popularnych dostawców tożsamości typu `Google`, `Twitter`, `Facebook`, `Github` oraz umożliwia integracje z innymi zewnętrznymi usługami. Wykorzystuje także popularne standardy branżowe takie jak `OAuth 2.0` i `OpenID Connect` oraz ściśle współpracuje z innymi usługami Firebase. Operacje procesu uwierzytelniania mogą odbywać się poprzez użycie gotowych komponentów interfejsu użytkownika (`FirebaseUI`) lub wykorzystując `Firebase SDK` do ręcznej konfiguracji.

## Użytkownicy
Obiekt `Firebase User` reprezentuje konto użytkownika zarejestrowanego w aplikacji w projekcie Firebase. Do użytkownika przypisany jest zestaw podstawowych właściwości takich jak: unikalny identyfikator, podstawowy adres email, nazwa i adres URL zdjęcia, które są przechowywane w bazie danych projektu i mogą być aktualizowane. Nie ma możliwości dodania bezpośrednio innych właściwości do obiektu w bazie danych projektu, zamiast tego należy przechowywać dodatkowe właściwości w bazie danych `Firebase Realtime Database`. Przy pierwszej rejestracji do aplikacji profil użytkownika zostanie uzupełniony o dostępne informacje w zależności od udostępnionych danych przez dostarczyciela, np. przy rejestracji adresem email i hasłem zostanie dodany tylko podstawowy adres email. Użytkownik może zalogować się na to samo konto za pomocą różnych metod np. używając adresu email i hasła, konta Google czy konta Twitter. W momencie pomyślnej rejestracji lub logowania następuje przypisanie instancji `Firebase Auth` dla bieżącego użytkownika, której zadaniem jest utrzymanie stanu użytkownika do momentu wylogowania, np. ponowne uruchomienie aplikacji nie powoduje utraty informacji.

## FirebaseUI
`FirebaseUI` jest biblioteką opartą o pakiet `SDK Firebase Authentication`, która dostarcza komponenty interfejsu graficznego umożliwiające w łatwy sposób wykonanie operacji autoryzacji. Już za pomocą jednego kliknięcia możliwe jest zalogowanie się do aplikacji przy użyciu wielu dostarczycieli (email, telefoniczna autoryzacja, `Google Sign-In` itp) czy też wywołanie różnych metod uwierzytelniania i zadań zarządzania kontem. FirebaseUI automatycznie integruje się z mechanizmem `Smart Lock for Password` i pozwala także na personalizacje stylu graficznego. Aby skorzystać z wybranych dostarczycieli autoryzacji należy dodać odpowiednie zewnętrzne zależności do pliku `build.gradle`, włączyć wspieranie logowania i uzupełnić wymagane informacje w konsoli Firebase oraz dla Facebook i Twitter dodać klucz aplikacji do zasobów tekstowych. Poniższy listing przedstawia sposób wykorzystanie FirebaseUI w procesie logowania, wylogowania i usuwania konta.

{% highlight kotlin %}
class FirebaseUIActivity : AppCompatActivity() {

    companion object {
        const val SIGN_IN = 100
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_firebase_ui)

        //add available providers
        val providers = arrayListOf(
            AuthUI.IdpConfig.EmailBuilder().build(),
            AuthUI.IdpConfig.PhoneBuilder().build(),
            AuthUI.IdpConfig.GoogleBuilder().build(),
            AuthUI.IdpConfig.TwitterBuilder().build())

        //start sign in screen and optional customize by logo, theme and urls
        val intent = AuthUI.getInstance()
            .createSignInIntentBuilder()
            .setAvailableProviders(providers)
            .setLogo(R.drawable.logo)
            .setTheme(R.style.AppTheme)
            .setTosAndPrivacyPolicyUrls("http://androidcode.pl/terms", "http://androidcode.pl/privacy")
            .build()
        startActivityForResult(intent, SIGN_IN)
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == SIGN_IN) {
            val response = IdpResponse.fromResultIntent(data)
            if (resultCode == Activity.RESULT_OK) {
                //success, get user and do some work
                val user = FirebaseAuth.getInstance().currentUser
                startActivity(Intent(this, UserActivity::class.java))
            }
            else {
                //fail, show some error message
            }
        }
    }
}

class UserActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_user)

        signOut.setOnClickListener {
            AuthUI.getInstance().signOut(this)
                .addOnCompleteListener {
                    //do something like navigate to home screen
                }
        }

        delete.setOnClickListener {
            AuthUI.getInstance().delete(this)
                .addOnCompleteListener {
                    //do something like navigate to home screen
                }
        }
    }

    //more methods
}
{% endhighlight %}

## Firebase SDK
Kiedy zachodzi potrzeba przejęcia większej kontroli nad procesami uwierzytelniania należy w tym celu wykorzystać `Firebase SDK`, który umożliwia zdefiniowanie zachowania i obsługę zdarzeń na każdym kroku danego procesu. Poniższy listing prezentuje wykorzystanie Firebase SDK w procesie rejestracji za pomocą email i hasła, logowania i wylogowania użytkownika.

{% highlight kotlin %}
//TODO code
{% endhighlight %}

Co więcej istnieje możliwość ręcznej konfiguracji zewnętrznych API autoryzacji dla m.in. Google, Twitter, Facebook, Github i integracji z kontem użytkownika Firebase. Ponadto Firebase Authentication oferuje także mechanizm uwierzytelniania kont anonimowych i ich konwersji do kont stałych oraż możliwość logowania się za pomocą linka w wiadomości email.

## Zarządzanie użytkownikami
Firebase Authentication poza podstawowymi właściwościami użytkownika Firebase umożliwia także uzyskanie dostępu do informacji profilowych dostarczonych przez zewnętrznych usługodawców autoryzacji. Ponadto możliwe jest dodanie lub zmiana bieżących danych (informacje profilowe, email, hasło, itp), a także usuwanie konta oraz wysyłanie maili zmiany hasła, weryfikacji czy ponownej autoryzacji konta.

{% highlight kotlin %}
//TODO code
{% endhighlight %}