---
layout: post
title: "Cloud Messaging"
date: 2019-04-22
categories: ["Firebase"]
image: firebase/cloud_messaging
github: firebase/tree/master/cloud_messaging
description: "Firebase"
version: Firebase-Firestore 17.3
keywords: "firebase, chmura, cloud, message, notification, data, android, programowanie, programming"
---

## Wiadomość
`Cloud Messaging` (`FCM`) dostarcza wiadomości do wybranych użytkowników aplikacji wysłanych z poziomu `konsoli Firebase`, usługi `Cloud Functions` lub zewnętrznego serwera. Wiadomości dzielą się na dwa typy: automatycznie wyświetlane notyfikacje oraz ręcznie obsługiwane wiadomości danych. W przypadku notyfikacji, gdy aplikacja jest w tle powiadomienia dostarczane są do zasobnika powiadomień, a kiedy działa na pierwszym planie wówczas obsługiwana jest przez funkcję zwrotną. Wiadomości mogą przyjmować format `JSON` i muszą zawierać polę `message`, składające się z pola `message.token` oraz `message.notification` (dla notyfikacji) lub `data` (dla wiadomości danych). Wiadomości mogą zawierać także jednocześnie oba pola: `notification` i `data`.

{% highlight json %}
{
  "message":{
    "token":"abcdefghijklmnopqrstuvwxyz",
    "notification":{
      "title":"text",
      "body":"content"
    }
  }
}

{
  "message":{
    "token":"abcdefghijklmnopqrstuvwxyz",
    "data":{
      "value" : "some value",
      "body" : "some body",
    }
  }
}

{
  "message":{
    "token":"abcdefghijklmnopqrstuvwxyz",
    "notification":{
      "title":"text",
      "body":"content"
    },
    "data" : {
      "value" : "some value",
      "body" : "some body"
    }
  }
}
{% endhighlight %}

Wiadomości konfigurowane pod różne platformy (`Android`, `iOS`, `web`) powinny przestrzegać zasad dostępności kluczy. Jeśli wiadomość kierowana jest do wielu platform wówczas należy używać kluczy ogólnych (`notification`, `data`, `notification.title`, `notification.body`), a w przypadku wysyłania tylko na jedną platformę należy wskazać typ (`android`, `apns`, `webpush`) i wykorzystać specyficzne klucze danej platformy (dla `Android` m.in. `ttl`, `priority`, `data`, `notification`).

{% highlight json %}
{
  "message":{
    "token":"abcdefghijklmnopqrstuvwxyz",
    "notification":{
      "title":"text",
      "body":"content"
    },
    "android" : {
      "ttl" : "36000s",
      "notification": {
        "click_action":"OPEN_SETTINGS"
      }
    }
  }
}
{% endhighlight %}

## Usługa
Aby przetwarzać przychodzące wiadomości należy rozszerzyć klasę `FirebaseMessagingService`, zarejestrować usługę oraz dodać opcjonalne informacje w pliku `AndroidManifest`. Usługa `FirebaseMessagingService` odpowiedzialna jest za nasłuchiwanie i podejmowanie akcji dla otrzymanych wiadomości i przyznanego kodu `token`.

{% highlight kotlin %}
class FcmTokenService : FirebaseMessagingService() {

    //override at least onNewToken, onMessageReceived and onDeletedMessages
}
{% endhighlight %}

{% highlight xml %}
<!-- register service -->
<service android:name=".java.FcmTokenService">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>
</service>

<!-- optional add defaults -->
<meta-data
    android:name="com.google.firebase.messaging.default_notification_icon"
    android:resource="@drawable/ic_notification" />
<meta-data
    android:name="com.google.firebase.messaging.default_notification_color"
    android:resource="@color/colorNotification" />
<meta-data
    android:name="com.google.firebase.messaging.default_notification_channel_id"
    android:value="@string/notification_channel_id" />
{% endhighlight %}

## Token
Kluczem identyfikującym użytkownika w usłudze `Cloud Messaging` jest `token`, który przyznany przez aplikację powinien zostać wysłany na serwer dzięki czemu możliwym będzie rozpoznanie adresatów wiadomości. Warto mieć na uwadzę, że token może ulec zmianie m.in. w sytuacji usunięcia aplikacji czy wyczyszczenia danych w związku z czym należy rejestrować jego zmiany w metodzie `onNewToken`. Aby ręcznie pobrać bieżący token należy wywołać `FirebaseInstanceId.getInstance().instanceId`.

{% highlight kotlin %}
class FcmTokenService : FirebaseMessagingService() {

    override fun onNewToken(token: String?) {
        //send token to server to able to get messages
    }

    fun fetchCurrentToken() {
        FirebaseInstanceId.getInstance().instanceId.addOnCompleteListener(OnCompleteListener { task ->
            if (!task.isSuccessful) {
                return@OnCompleteListener
            }

            //get token and do something
            val token = task.result?.token
        })
    }
}
{% endhighlight %}

## Odbieranie
Zarządzanie otrzymanymi i usuniętymi wiadomościami możliwe jest przez nadpisanie metod `onMessageReceived` i `onDeletedMessages`. Metoda `onMessageReceived` dostarcza obsługi dla większości typów wiadomości za wyjątkiem wiadomości notyfikacji otrzymanych podczas działania aplikacji w tle, które trafiają do zasobnika systemowego. Domyślnie kliknięcie na notyfikacje powoduje uruchomienie `launcher` aplikacji, a opcjonalne dane z pola `data` zawarte są w kluczach `extras` obiektu `Intent`.

{% highlight kotlin %}
class FcmTokenService : FirebaseMessagingService() {

    override fun onMessageReceived(remoteMessage: RemoteMessage?) {
        //check message contains data payload
        remoteMessage?.data?.isNotEmpty()?.let {
            //prepare some action
        }

        //check message contains notification payload
        remoteMessage?.notification?.let {
            //prepare some action like show notification
        }
    }

    override fun onDeletedMessages() {
        //do something like server sync for deleted and not received message
    }
}
{% endhighlight %}

## Grupowanie
Wiadomości mogą być wysyłane do grupy użytkowników na podstawie informacji przypisanych do konta, zapisanych do tematu (`topic`) lub należących do grupy urządzeń (zawiera dodatkowo autentykacje). W celu zapisania i wypisania klienta do tematu należy wywołać metodę `subscribeToTopic` lub `unsubscribeFromTopic`.

{% highlight kotlin %}
fun subscribeToTopic() {
    FirebaseMessaging.getInstance().subscribeToTopic("title").addOnCompleteListener { task ->
        if (!task.isSuccessful) {
            //subscribe failed
        }
        //subscribe success
    }
}

fun unsubscribeToTopic() {
    FirebaseMessaging.getInstance().unsubscribeFromTopic("title").addOnCompleteListener { task ->
        if (!task.isSuccessful) {
            //unsubscribe failed
        }
        //unsubscribe success
    }
}
{% endhighlight %}
