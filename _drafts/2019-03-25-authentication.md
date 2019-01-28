---
layout: post
title: "Authentication"
date:  2019-03-25
categories: ["Firebase"]
image: firebase/authentication
github: firebase/tree/master/authentication
description: "Firebase"
version: Firebase-Auth 16.1, Firebase-UI-Auth 4.1
keywords: "firebase, autentykacja, autoryzacja, authentication, firebaseui, firebaseauth, auth, account, user, email, password, phone, google, facebook, twitter, github, rejestracja, logowanie, sign in, log in, sign up, log out, android, programowanie, programming"
---

## Wprowadzenie
Tworzenie i zarządzania kontem użytkownika to dla większości aplikacji podstawowa funkcjonalność, która pozwala na personalizowane wykorzystanie funkcjonalności aplikacji, a znajomość tożsamości ułatwia bezpieczne zapisywanie i powiązanie danych z kontem synchronizowanym na różnych urządzeniach. `Firebase Authentication` dostarcza łatwych w użyciu usług `backend` przeznaczonych do uwierzytelniania użytkowników w aplikacji za pomocą `hasła`, `numeru telefonu`, popularnych dostawców tożsamości typu `Google`, `Twitter`, `Facebook`, `Github` oraz umożliwia integracje z innymi zewnętrznymi usługami. Wykorzystuje także popularne standardy branżowe takie jak `OAuth 2.0` i `OpenID Connect` oraz ściśle współpracuje z innymi usługami Firebase. Operacje procesu uwierzytelniania mogą odbywać się poprzez użycie gotowych komponentów interfejsu użytkownika (`FirebaseUI`) lub wykorzystując `Firebase SDK` do ręcznej konfiguracji.

## Użytkownicy
Obiekt `Firebase User` reprezentuje konto użytkownika zarejestrowanego w aplikacji w projekcie Firebase. Do użytkownika przypisany jest zestaw podstawowych właściwości takich jak: unikalny identyfikator, podstawowy adres email, nazwa i adres URL zdjęcia, które są przechowywane w bazie danych projektu i mogą być aktualizowane. Nie ma możliwości dodania bezpośrednio innych właściwości do obiektu w bazie danych projektu, zamiast tego należy przechowywać dodatkowe właściwości w bazie danych `Cloud Firestore` lub `Firebase Realtime Database`. Przy pierwszej rejestracji do aplikacji profil użytkownika zostanie uzupełniony o dostępne informacje w zależności od udostępnionych danych przez dostarczyciela, np. przy rejestracji adresem email i hasłem zostanie dodany tylko podstawowy adres email. Użytkownik może zalogować się na to samo konto za pomocą różnych metod np. używając adresu email i hasła, konta Google czy konta Twitter. W momencie pomyślnej rejestracji lub logowania następuje przypisanie instancji `Firebase Auth` dla bieżącego użytkownika, której zadaniem jest utrzymanie stanu użytkownika do momentu wylogowania, np. ponowne uruchomienie aplikacji nie powoduje utraty informacji.

![Konta użytkowników](/assets/img/diagrams/firebase/users.png){: .center-image }

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

        //check if user is already signed in
        //if not then show sign in screen

        //add available providers
        val providers = arrayListOf(
            AuthUI.IdpConfig.EmailBuilder().build(),
            AuthUI.IdpConfig.PhoneBuilder().build(),
            AuthUI.IdpConfig.GoogleBuilder().build())
        //add more like Twitter, Facebook etc

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

        signOutButton.setOnClickListener {
            AuthUI.getInstance().signOut(this)
                .addOnCompleteListener {
                    //do something like navigate to home screen
                }
        }

        deleteAccountButton.setOnClickListener {
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
class FirebaseAuthActivity : AppCompatActivity() {

    private lateinit var auth: FirebaseAuth

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_firebase_auth)
        auth = FirebaseAuth.getInstance()

        signUpButton.setOnClickListener { updateProfile() }
        signInButton.setOnClickListener { changeEmail() }
    }

    override fun onStart() {
        super.onStart()
        if(auth.currentUser != null) {
            //user is signed in, update UI or navigate
        }
    }

    private fun signUp() {
        //validate fields before
        val email = emailEditText.getText().toString()
        val password = passwordEditText.getText().toString()
        auth.createUserWithEmailAndPassword(email, password).addOnCompleteListener { task ->
            if (task.isSuccessful) {
                //sign up success, update UI or navigate
                val user = auth.currentUser
                startActivity(Intent(this, ProfileActivity::class.java))
            }
            else {
                //sign up fails, show error message
            }
        }
    }

    private fun signIn() {
        //validate fields before
        val email = emailEditText.getText().toString()
        val password = passwordEditText.getText().toString()
        auth.signInWithEmailAndPassword(email, password).addOnCompleteListener { task ->
            if (task.isSuccessful) {
                //sign ip success, update UI or navigate
                val user = auth.currentUser
                startActivity(Intent(this, ProfileActivity::class.java))
            }
            else {
                //sign in fails, show error message
            }
        }
    }
}
{% endhighlight %}

Co więcej istnieje możliwość ręcznej konfiguracji zewnętrznych API autoryzacji dla m.in. Google, Twitter, Facebook, Github i integracji z kontem użytkownika Firebase. Ponadto Firebase Authentication oferuje także mechanizm uwierzytelniania kont anonimowych i ich konwersji do kont stałych oraż możliwość logowania się za pomocą linka w wiadomości email.

## Zarządzanie użytkownikami
Firebase Authentication poza podstawowymi właściwościami użytkownika Firebase umożliwia także uzyskanie dostępu do informacji profilowych dostarczonych przez zewnętrznych usługodawców autoryzacji. Ponadto możliwe jest dodanie lub zmiana bieżących danych (informacje profilowe, email, hasło, itp), a także usuwanie konta oraz wysyłanie maili zmiany hasła, weryfikacji czy ponownej autoryzacji konta.

{% highlight kotlin %}
class ProfileActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_profile)
        
        showUserInfo()
        showProfiles()

        updateProfileButton.setOnClickListener { updateProfile() }
        changeEmailButton.setOnClickListener { changeEmail() }
        changePasswordButton.setOnClickListener { changePassword() }
        deleteAccountButton.setOnClickListener { deleteAccount() }
        signOutButton.setOnClickListener { signOut() }
    }

    private fun showUserInfo() {
        val user = FirebaseAuth.getInstance().currentUser
        user?.let {
            var info = String()
            info += "uid: ${user.uid}\n"
            info += "name: ${user.displayName}\n"
            info += "email: ${user.email}\n"
            info += "is verified: ${user.isEmailVerified}\n"
            info += "photoUrl: ${user.photoUrl.toString()}\n"
            //get more info
            userInfo.setText(info)
        }
    }

    private fun showProfiles() {
        val user = FirebaseAuth.getInstance().currentUser
        user?.let {
            for(profile in it.providerData) {
                var info = String()
                info += "PROVIDER: ${profile.providerId}\n"
                info += "uid: ${profile.uid}\n"
                info += "name: ${profile.displayName}\n"
                //get more info
                userProfiles.setText(info)
            }
        }
    }

    private fun updateProfile() {
        val user = FirebaseAuth.getInstance().currentUser
        val profileUpdates = UserProfileChangeRequest.Builder()
                .setDisplayName(editText.getText().toString())
                .setPhotoUri(Uri.parse("http://androidcode.pl/assets/img/logo.png"))
                .build()

        user?.updateProfile(profileUpdates)?.addOnCompleteListener { task ->
            if (task.isSuccessful) {
                //show some message and change UI
            }
        }
    }

    private fun changeEmail() {
        val user = FirebaseAuth.getInstance().currentUser
        val email = editText.getText().toString() //validate
        user?.updateEmail(email)?.addOnCompleteListener { task ->
            if (task.isSuccessful) {
                //show some message and change UI
            }
        }
    }

    private fun changePassword() {
        val user = FirebaseAuth.getInstance().currentUser
        val newPassword = editText.getText().toString() //validate
        user?.updatePassword(newPassword)?.addOnCompleteListener { task ->
            if (task.isSuccessful) {
                //show some message and change UI
            }
        }
    }

    private fun deleteAccount() {
        val user = FirebaseAuth.getInstance().currentUser
        user?.delete()?.addOnCompleteListener { task ->
            if (task.isSuccessful) {
                //show some message and change UI
            }
        }
    }

    private fun signOut() {
        FirebaseAuth.getInstance().signOut()
        finish()
    }

    //more methods
}
{% endhighlight %}
