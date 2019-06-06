---
layout: post
title: "SharedPreferences"
date: 2019-08-05
categories: ["Przechowywanie"]
image: store/sharedpreferences
github: store/tree/master/sharedpreferences
description: "Przechowywanie danych"
keywords: "store, data, datastore, sharedpreferences, preferences, key, value, primitive, settings, get, put, editor, android, programowanie, programming"
---

## Charakterystyka
`SharedPreferences` umożliwiają przechowywanie i zarządzanie prostymi danymi prymitywów tj.: wartości logiczne, liczby całkowite i zmiennoprzecinkowe oraz teksty w postaci pary `klucz - wartość`. Dane przechowywane są na urządzeniu w plików `xml` niezależnie od cyklu życia aplikacji. Istnieją tak długo dopóki nie zostaną usunięte przez kod programu lub wyczyszczone ręcznie przez użytkownika z danych aplikacji. Aplikacja może posiadać wiele instancji `SharedPreferences`, które najczęściej są prywatne lecz mogą być także publiczne dla innych aplikacji. 

## Przeznaczenie
Nazwa `SharedPreferences` może sugerować przeznaczenie do przechowywania informacji nt ustawień użytkownika, co jest po części prawdą. Jednakże wszystkie informacje, które można sprowadzić do postaci klucz - wartość nadają się do umieszczenia w `SharedPreferences`. Mogą więc to być ustawienia czy preferencje użytkownika, ale równie dobrze także inne dane stanu aplikacji (np. najlepszy wynik) czy dane autoryzacyjne. Pomimo, że wiele informacji może być technicznie zapisanych w `SharedPreferences` (np. konwersja do `String`) to nie wszystkie powinny się tam znaleźć. Umieszczając wartości należy kierować się zasadą prostych, pojedynczych informacji (nie kolekcji). W pozostałych przypadkach warto rozważyć alternatywy w postaci m.in. plików i bazy danych.

## Implementacja
Aby uzyskać referencję do wskazanego pliku `SharedPreferences` należy wywołać metodę `getSharedPreferences` lub `getPreferences`. Czytanie wartości odbywa się za pomocą metod get danego typu, natomiast zapisywanie wartości przebiega jako transakcja na obiekcie `SharedPreferences.Editor`.

{% highlight kotlin %}
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }
 
    fun getSharedPreferences(name : String) : SharedPreferences {
        //must be called on Context
        return getSharedPreferences(name, Context.MODE_PRIVATE) //pass public to allow access outside app
    }

    fun getSharedPreferencesForActivity() : SharedPreferences {
        //SharedPreferences file only for this activity
        return getPreferences(Context.MODE_PRIVATE) 
    }

    fun write(key : String, value : Int) {
        val sharedPref = getSharedPreferences("pl.androidcode.app.FILE_KEY")
        val editor = sharedPref.edit()
		
        //do some transactions
        editor.putInt(key, value) //or use another method to put different types
        editor.commit() //to save immediately or apply to save in background
    }
    
    fun read(key : String) : Int {
        val sharedPref = getSharedPreferences("pl.androidcode.app.FILE_KEY")
        val default = 0
        return sharedPref.getInt(key, default)
    }
}
{% endhighlight %}

## Preference
W przypadku standardowych ustawień aplikacji edytowanych przez interfejs graficzny dobrym pomysłem mogłoby być wykorzystanie biblioteki `Preference`, która ułatwia pracę z `SharedPreferences` poprzez dostarczenie kontrolek widoku dla kluczy ustawień. W tym celu należy stworzyć Fragment rozszerzający `PreferenceFragmentCompat` i w metodzie `onCreatePreferences` przekazać plik `xml` z preferencjami. Wprowadzane zmiany dotyczą globalnych domyślnych ustawień możliwych do uzyskania przez wywołanie `getDefaultSharedPreferences`.

{% highlight kotlin %}
class SettingsActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_settings)

        //add Preference Fragment
        supportFragmentManager
            .beginTransaction()
            .replace(R.id.container, SettingsFragment())
            .commit()
    }
	
    //read some pref when needed, use getDefaultSharedPreferences
    fun readPref() : Boolean {
        val sharedPref = getDefaultSharedPreferences(this)
        return sharedPref.getBoolean("pref2", false)
    }
}

class SettingsFragment : PreferenceFragmentCompat() {

    override fun onCreatePreferences(savedInstanceState: Bundle?, rootKey: String?) {
        setPreferencesFromResource(R.xml.preferences, rootKey)
    }
	
    override fun onPreferenceTreeClick(preference: Preference?): Boolean {
        return when (preference?.key) {
            "pref2" -> {
                //do something when pref has changed if needed
                true
            }
            else -> {
                super.onPreferenceTreeClick(preference)
            }
        }
    }
	
    //implement more methods to react for interactions
}
{% endhighlight %}

{% highlight kotlin %}
<!-- PreferenceScreen must be parent -->
<androidx.preference.PreferenceScreen
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <Preference
        app:key="pref1"
        app:title="Some title"
        app:summary="More details"/>

    <SwitchPreferenceCompat
        app:key="pref2"
        app:title="Some message"
        app:summary="More details"/>

    <!-- use more preferences from Preference class and subclasses -->

</androidx.preference.PreferenceScreen>
{% endhighlight %}

## DataStore
Biblioteka `Preference` domyślnie zarząda danymi wykorzystując implementację `SharedPreferences`. Nierzadko jednak taka realizacja zapisu i odczytu danych może nie być wystarczająca ponieważ może np. zachodzić potrzeba dodatkowego zapisu danych w chmurze ze względu na synchronizację preferencji między urządzeniami. W takiej sytuacji z pomocą przychodzi `PreferenceDataStore` dostępny od wersji systemu `Android O`.

{% highlight kotlin %}
//use custom PreferenceDataStore inside onCretePreferences
class DataStoreFragment : PreferenceFragmentCompat() {

    override fun onCreatePreferences(savedInstanceState: Bundle?, rootKey: String?) {
        setPreferencesFromResource(R.xml.preferences, rootKey)

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {

            //enable data store only for specific preference
            val preference = findPreference("pref2")
            preference.preferenceDataStore = DataStore()

            //ore enable for entire hierarchy
            preferenceManager.preferenceDataStore = DataStore()
        }
    }
}

//override only used operations
@RequiresApi(Build.VERSION_CODES.O)
class DataStore : PreferenceDataStore {
    
    //make sure to provide proper non blocking threading during extra work
    
    override fun getBoolean(key: String?, defValue: Boolean): Boolean {
        //try to get pref from remote
        //if failed or doesn't exists then try to get from local
        return super.getBoolean(key, defValue)
        //log operation
    }

    override fun putBoolean(key: String?, value: Boolean) {
        //try to put pref to remote
        //save also in local
        super.putBoolean(key, value)
        //log operation
    }
}
{% endhighlight %}

## Bezpieczeństwo
Biblioteka `Security` dostarcza dodatkowej ochrony przed niepowołanym dostępem do danych stosując się do kryptograficznych reguł bezpieczeństwa z jednoczesnym zachowaniem wydajności. Wykorzystuje dwuczęściowy system zarządzania kluczami składający się ze zbioru kluczy `keyset` oraz z klucza głównego `master key`. `Keyset` zawiera klucze szyfrujące dane, które są przechowywane w `SharedPreferences`, natomiast `master key` jest kluczem szyfrującym klucze z `keyset` i przechowywany jest w `KeyStore`. Dostęp do zaszyfrowanych `SharedPreferences` odbywa się przez instancję `EncryptedSharedPreferences`.

{% highlight kotlin %}
class EncryptedActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val encryptedSharedPref = createEncryptedSharedPref()
        encryptedSharedPref.edit().putInt("key", 100).commit()
        val encrypted = encryptedSharedPref.getInt("key", 0) //100
		
        //encrypted and standard SharedPreferences are different despite the same file key declaration
        val sharedPref = getSharedPreferences("pl.androidcode.app.FILE_KEY", Context.MODE_PRIVATE)
        val normal = sharedPref.getInt("key", 0) //it will be 0
    }

    fun createEncryptedSharedPref(): SharedPreferences {
        return EncryptedSharedPreferences.create(
            "pl.androidcode.app.FILE_KEY",
            getMasterKeyAlias(),
            this,
            EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
            EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
        )
    }

    fun getMasterKeyAlias(): String {
        val keyGenParameterSpec = MasterKeys.AES256_GCM_SPEC
        return MasterKeys.getOrCreate(keyGenParameterSpec)
    }
}
{% endhighlight %}