---
layout: post
title: "Single Activity"
date: 2019-09-09
categories: ["Wzorce architektoniczne"]
permalink: /blog/wzorce/:title/
image: architecture/single_activity
github: architectural-patterns/tree/master/single_activity
description: "Wzorce projektowe / architektoniczny"
keywords: "wzorzec, pattern, architektura, single, activity, fragment, navigation, android, kotlin, programowanie, programming"
---

## Activity
`Activity` (`Aktywność`) jest jednym z podstawowych komponentów systemu odpowiedzialnym za reprezentacje całego ekranu interfejsu oraz interakcje z użytkownikiem. Jeśli aplikacja posiada graficzny interfejs wówczas ma przynajmniej jedną Activity. W tym miejscu następuje renderowanie widoku, przechwytywanie akcji użytkownika czy startowanie innych komponentów. Użycie Activity tak jak innych komponentów jest ścisle związane z jego cyklem życia. Jednakże cykl życia Activity może sprawić sporo problemów ponieważ jest on przeważnie powiązany z operacjami wykonywanymi zarówno na głównym i roboczym wątku, a także odpowiada za cykl życia innych obiektów. W wyniku nieprawidłowego zarządzania cyklem życia może dojść do wycieku pamięci i innych błędów. Szczególnymi sytuacjami jest zejście do tła i powrót z tła oraz nawigowanie między Activity. Może to prowadzić do wniosku, że ograniczenie nawigacji między Activity zmniejszy ilość potencjalnych problemów. 

## Fragment 
`Fragment` reprezentuje pewną porcje interfejsu graficznego czy też zachowania, która jest częścią `Activity` lub innego `Fragment`. Dzięki zastosowaniu Fragment część interfejsu graficznego i zachowania może być współdzielona w wielu miejscach co zmniejsza ilość nadmiarowego kodu oraz ułatwia tworzenie aplikacji z dedykowanymi widokami dla telefonów i tabletów. Activity może być budowana w oparciu o obiekty typu Fragment w relacji jeden do jeden lub jeden do wielu, tzn. jeden lub wiele obiektów Fragment może być widocznych i aktywnych w tym samym czasie. Activity zarządza stosem w którym przechowywane są instancje Fragment wykorzystując w tym celu `FragmentManager` dzięki czemu może nawigować między obiektami Fragment i zmieniać je w czasie działania co nie jest czynnością trywialną. Cykl życia obiektów Fragment jest zależny od cyklu życia rodzica Activity.

## Kompozycja
Aplikacje mogą być tworzonone w oparciu o różne warianty wykorzystania `Activity` i `Fragment`. Podstawowy sposób polega na użyciu samodzielnych obiektów typu Activity, które są tworzone dla każdego ekranu. Nawigacja zachodzi za pomocą metod `startActivity`, `startActivityForResult` natomiast wymiana danych odbywa się poprzez obiekt typu `Bundle` lub jest zapisywana na dysku. Usprawnieniem tego podejścia jest wykorzystanie obiektów Fragment dzięki czemu część interfejsu i logiki biznesowej może być wyłączona z odpowiedzialności Activity oraz ponownie użyta w innej. Wymaga to jednak dodania mechanizmu komunikacji między Activity i obiektami Fragment co może być rozwiązane na wiele różnych sposobów w tym m.in. callback, szyna zdarzeń, programowanie reaktywne. Rozwinięciem tego podejścia jest zasada jednego aktywnego Fragment dla Activity, tzn. w zależności od stanu, Activity wyświetla odpowiedni Fragment wypełniający stałą przestrzeń - przeważnie cały ekran (Fragment może zawierać inne obiekty Fragment). Dzięki temu redukowana jest ilość Activity lecz nie eliminuje to problemów związanych z nawigacją i komunikacją. Odpowiedzią na wspomniane problemy może być zastosowanie architektury `Single Activity`, której działanie jest oparte o jedną centralną Activity zarzadzającą wieloma obiektami Fragment.

## Zastosowanie
`Single Activity` w większości przypadków zakłada istnienie jednej `głównej Activity`. Wszelkie zmiany widoku oraz nawigacja przebiega poprzez manipulacje stosem instancji `Fragment`. Innymi słowy zmiana widoku nie powoduje pokazania nowego okna lecz wymianę aktywnego Fragment. Z uwagi na eliminacje procesu nawigacji między Activity redukuje się ilość wywołań metod cyklu życia Activity co przekłada się na zmniejszenie ilości potencjalnych problemów. Komunikacja między ekranami zostaje sprowadzona do procesu komunikacji między obiektami Fragment dzięki czemu nie ma już potrzeby konstruowania i przetwarzania podatnego na błędy obiektu `Intent` w `startActivity`, `startActivityForResult` i `onActivityResult` czy też zapisywania danych na dysku lub w obiekcie `Singleton`. Activity staje się centralnym punktem zarządzania i współdzielenia jednego stanu oraz danych, a także widoków nawigacyjnych i paska statusu. Ponadto zastosowanie `Single Activity` zmniejsza ilość klas i wpisów do `AndroidManifest` oraz dba o dostarczenie lepszych wrażeń `User Experience` poprzez szybsze przejścia i animacje.

## Ograniczenia
Głównym zagrożeniem zastosowania `Single Activity` jest sprowadzenie Activity do roli `super obiektu` nawet pomimo implementacji logiki biznesowej w obiektach Fragment. Jedyną odpowiedzialnością Activity powino być współdzielenie stanu i manipulowanie obiektami Fragment oraz ewentualnie uczestniczenie w procesie komunikacji. Jednakże zarządzanie stosem o dużej ilości instancji Fragment przez Activity nie należy do łatwych. Co więcej należy brać pod uwagę także cykl życia wszystkich Fragment, który dodatkowo komplikuje sytuacje. Rozważając wybór architektury `Single Activity` należy zastanowić się nad bilansem zysków i strat. Być może lepszym wyborem będzie rezygnacja ze standardowego podejścia z jedną Activity na rzecz wariantu opartego o kilka Activity, które tym samym dokonują podziału aplikacji na niezależne lub słabo powiązane ze sobą obszary. Czynnikiem przemawiającym za taką decyzją poza biznesową niezależnością obszarów mogą być także różnice w pasku statusu i nawigacji. Niezależnie od wyboru architektury warto dążyć do minimalizacji ilości Activity na rzecz Fragment.

## Navigation
Biblioteka `Navigation` znacząco ułatwia tworzenia aplikacji w architekturze `Single Activity`. Automatycznie przejmuje odpowiedzialność zarządzania transakcjami i stosem Fragment co praktycznie wyklucza ręczne wywołanie operacji na `FragmentManager`. Ponadto w łatwy sposób dostarcza animacje i przejścia oraz wspiera nawigacyjne wzorce interfejsu użytkownika poprzez współprace z kontrolkami widoku dzięki czemu eliminuje problem aktualizacji pasku statusu i nawigacji. Co więcej implementuje przechwytywanie `Deep Link`, usprawnia użycie `ViewModel` oraz umożliwia jednoznaczną wymianę argumentów przy pomocy `SafeArgs`. Składa się trzech kluczowy elementów: grafu nawigacyjnego, pustego kontenera hostującego `NavHost` oraz kontrolera `NavController` odpowiedzialnego za zarządzanie nawigacją. Komponenty te zostaną przedstawione na przykładzie aplikacji `TeleContact`, która umożliwia wykonywanie i śledzenie historii połączeń oraz zarządzanie kontaktami.

## Graf nawigacyjny
`Graf nawigacyjny` zawiera zestaw informacji nt relacji nawigacji między miejscami w aplikacji zwanymi `destinations`. Mogą to być m.in. obiekty `Fragment` oraz inne grafy wewnętrzne (`nested graphs`). Deklaruje możliwe przejścia, animacje, zachowanie stosu czy argumenty w postaci wpisów `actions`. Jego rola ogranicza się do definicji ścieżek nawigacyjnych, które mogą zostać wykorzystane przez `NavController`. Jest plikiem `xml` w folderze zasobów `navigation`. Może być tworzony także przez narzędzie graficzne `Navigation Editor` w `Android Studio` co zostało przedstawione na poniższym diagramie.

![Navigation graph](/assets/img/diagrams/architecture/navigation_graph.png){: .center-image }

Każdy graf rozpoczyna się klauzulą navigation określającą miejsce początkowe `startDestination`. W jej ciele znajdują się wpisy lokalizacji `fragment` składające się m.in. z deklaracji identyfikatora `id`, etykiety `label`, klasy `name` i widoku `layout` oraz zawierające definicję akcji `action`, argumentów `argument` i linków `deepLink`. Możliwe jest także dodanie wpisów grafu wewnętrznego `navigation`, dialogu informacyjnego `dialog` oraz akcji globalnej `action`.

{% highlight xml %}
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools" 
    android:id="@+id/nav_graph" app:startDestination="@id/mainFragment">
			
    <fragment android:id="@+id/mainFragment" android:name="MainFragment"
        android:label="fragment_main" tools:layout="@layout/fragment_main">
        <action android:id="@+id/main_to_calls" app:destination="@id/callsFragment"/>
        <action android:id="@+id/main_to_contacts" app:destination="@id/contactsFragment"/>
        <action android:id="@+id/main_to_about" app:destination="@id/aboutFragment"/>
    </fragment>

    <fragment android:id="@+id/callsFragment" android:name="CallsFragment"
              android:label="fragment_calls" tools:layout="@layout/fragment_calls"/>
    
    <!-- actions can include animations behavior in four different cases -->
    <fragment android:id="@+id/contactsFragment" android:name="ContactsFragment"
              android:label="fragment_contacts" tools:layout="@layout/fragment_contacts">                
        <action android:id="@+id/contacts_to_addContact" app:destination="@id/addContactFragment"/>
        <action android:id="@+id/contacts_to_contactDetails" app:destination="@id/contactDetailsFragment" 
            app:popEnterAnim="@anim/pop_enter_anim" app:popExitAnim="@anim/pop_exit_anim" 
            app:enterAnim="@anim/enter_anim" app:exitAnim="@anim/exit_anim"/>
    </fragment>
    
    <fragment android:id="@+id/aboutFragment" android:name="AboutFragment"
              android:label="fragment_about" tools:layout="@layout/fragment_about"/>
    
    <fragment android:id="@+id/addContactFragment" android:name="AddContactFragment"
              android:label="fragment_add_contact" tools:layout="@layout/fragment_add_contact">      
        <action android:id="@+id/addContact_to_contacts" app:destination="@id/contactsFragment" />
    </fragment>
    
    <!-- actions can include argument with type and default values for primitives -->
    <fragment android:id="@+id/contactDetailsFragment" android:name="ContactDetailsFragment"
              android:label="fragment_contact_details" tools:layout="@layout/fragment_contact_details">              
        <action android:id="@+id/contactDetails_to_editContact" app:destination="@id/editContactFragment"/>
        <argument android:name="CONTACT" app:argType="Contact"/>
    </fragment>
    
    <!-- notice that sometimes there is need to manage stack by pop previous elements -->
    <fragment android:id="@+id/editContactFragment" android:name="EditContactFragment"
              android:label="fragment_edit_contact" tools:layout="@layout/fragment_edit_contact">              
        <action android:id="@+id/editContact_to_contacts" app:destination="@id/contactsFragment" 
            app:popUpTo="@+id/contactsFragment" app:popUpToInclusive="true"/>
        <argument android:name="CONTACT" app:argType="Contact"/>
    </fragment>
	
    <!-- here could be some definitions of navigation or action elements -->

</navigation>
{% endhighlight %}

## NavHost
`NavHost` jest pustym kontenerem posiadającym kontroler nawigacyjny przeznaczony dla miejsc docelowych z wybranego grafu nawigacyjnego. Deklarowany jest w głównej `Activity` w kodzie źródłowym jako obiekt `NavHostFragment` lub w pliku `layout`. W jego miejscu pojawiać się będą lokalizacje zdefiniowane w grafie w odpowiedzi na wywołaną nawigacje.

{% highlight xml %}
<!-- activity_main layout -->
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- only one NavHost can be default which ensures intercept system back button -->
    <fragment
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph"/>

</RelativeLayout>
{% endhighlight %}

## NavController
Proces nawigacyjny dla wybranego grafu realizowany jest przez `NavController` i zachodzi w powiązanym z nim kontenerze `NavHostFragment`. Instancja kontrolera może zostać uzyskana z obiektu `NavHostFragment` lub przez metodę `findNavController` w kontekście `Activity`, `Fragment` lub `View`. Nawigacja przebiega w metodzie `navigate` na podstawie przekazanego identyfikatora zdefiniowanej akcji lub miejsca docelowego czy też obiektu `Uri` dla `DeepLink`. Dobrą praktyką jest współdzielenie stanu w obiekcie `ViewModel`, który dodatkowo ułatwia implementację nawigacji warunkowej.

{% highlight kotlin %}
class MainActivity : AppCompatActivity() {

    //only inflate layout with NavHostFragment
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //get instance of NavHostFragment and NavController
        val navHostFragment: NavHostFragment = 
            supportFragmentManager.findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        val navController: NavController = 
            navHostFragment.navController //or just use findNavController
    }
	
    //optionaly provide some manage of navigation views like BottomNavigationView
}

class MainFragment : Fragment() {

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_main, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        button_calls.setOnClickListener {
            findNavController().navigate(R.id.main_to_calls)
        }
        button_contacts.setOnClickListener {
            findNavController().navigate(R.id.main_to_contacts)
        }
        button_about.setOnClickListener {
            findNavController().navigate(R.id.main_to_about)
        }
    }
}

class ContactsFragment : Fragment(), ContactsAdapter.OnClickListener {

    private val viewModel: PhoneViewModel by lazy {
        ViewModelProviders.of(activity!!).get(PhoneViewModel::class.java)
    }
    private val adapter = ContactsAdapter().apply { setOnClickListener(this@ContactsFragment) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewModel.getContacts().observe(this, Observer {
            adapter.addItems(it)
        })
    }

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_contacts, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        recycler_view_contacts.apply {
            adapter = this@ContactsFragment.adapter
            layoutManager = LinearLayoutManager(activity)
        }

        button_add_contact.setOnClickListener {
            findNavController().navigate(R.id.contacts_to_addContact)
        }
		
        //if needed provide some conditional navigation based on ViewModel state here
    }

    override fun contactClick(contact: Contact) {
        //pass small data between destinations by Bundle or Safe Args
        //in real example it could be only some id instead of full object
        val args = Bundle().apply { putSerializable("CONTACT", contact) }
        findNavController().navigate(R.id.contacts_to_contactDetails, args)
    }
}

class ContactDetailsFragment : Fragment() {

    private val viewModel: PhoneViewModel by lazy {
        ViewModelProviders.of(activity!!).get(PhoneViewModel::class.java)
    }

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        return inflater.inflate(R.layout.fragment_contact_details, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val contact = arguments?.run {
            if(containsKey("CONTACT")) {
                getSerializable("CONTACT") as Contact
            }
            else null
        }

        contact?.run {
            text_view_name.text = name
            text_view_phone_number.text = phoneNumber
            button_call.setOnClickListener {
                //in real app make some call, but for simplify just add info about call
                viewModel.addCall(name)
            }
            button_edit.setOnClickListener {
                val args = Bundle().apply { putSerializable("CONTACT", this@run) }
                findNavController().navigate(R.id.contactDetails_to_editContact, args)
            }
        }
    }
}

//define rest fragments: CallsFragment, AboutFragment, EditContactFragment, AddConctactFragment
{% endhighlight %}

## NavigationUI
Klasa `NavigationUI` zawiera zbiór statycznych metod umożliwiających powiązanie widoków nawigacyjnych takich jak m.in. `Toolbar`, `ActionBar`, `DrawerLayout`, `BottomNavigationView` z grafem oraz kontrolerem nawigacyjnym. W tym celu należy zadeklarować wybrany widok w `Activity` oraz wywołać na nim metodę `setupWithNavController`. Obiekty `Fragment` są dodawane do widoku za pomocą zasobu `menu` w którym identyfikatory wpisów odpowiadają identyfikatorą zdefiniowanym w grafie. Interfejs `OnDestinationChangedListener` pozwala na wykrycie aktywnego miejsca docelowego co ułatwia m.in. na aktualizacje stanu widoków nawigacyjnych.

{% highlight kotlin %}
//remove MainFragment from code and graph
//put CallsFragment, ContactsFragment into BottomNavigationView and use AboutFragment in app bar menu

class NavigatioUIActivity : AppCompatActivity(), NavController.OnDestinationChangedListener {

    private lateinit var navController: NavController

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_navigation_ui)
        initNavController()
        initNavUI()
    }

    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        menuInflater.inflate(R.menu.menu_toolbar, menu)
        return true
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        //delegate menu items to NavController if connected
        return item.onNavDestinationSelected(navController) || super.onOptionsItemSelected(item)
    }

    private fun initNavController() {
        val navHostFragment = supportFragmentManager.findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        navController = navHostFragment.navController
        navController.addOnDestinationChangedListener(this)
    }

    private fun initNavUI() {
        //setup app bar with default up navigate button and home destinations
        val appBarConfiguration = AppBarConfiguration(setOf(R.id.callsFragment, R.id.contactsFragment))
        
        //connect Toolbar with NavController
        setSupportActionBar(toolbar)
        toolbar.setupWithNavController(navController, appBarConfiguration)
        
        //connect BottomNavigationView with NavController
        bottom_navigation_view.setupWithNavController(navController)
    }

    override fun onDestinationChanged(controller: NavController, destination: NavDestination, arguments: Bundle?) {
        //hide or show bottom navigation and menu items based on current destinations
        if(destination.id == R.id.callsFragment || destination.id == R.id.contactsFragment) {
            bottom_navigation_view.visibility = View.VISIBLE
            toolbar.menu?.findItem(R.id.aboutFragment)?.isVisible = true
        }
        else {
            bottom_navigation_view.visibility = View.GONE
            toolbar.menu?.findItem(R.id.aboutFragment)?.isVisible = false
        }
    }
}
{% endhighlight %}

{% highlight xml %}
<!-- activity_navigation_ui layout -->
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary"/>

    <fragment
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@id/toolbar"
        android:layout_above="@id/bottom_navigation_view"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_ui_graph"/>

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_navigation_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        app:menu="@menu/menu_bottom_nav"/>

</RelativeLayout>

<!-- menu_bottom_nav -->
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- id of item is destination id from navigation graph -->
    <item android:id="@+id/callsFragment" android:title="Calls"/>
    <item android:id="@+id/contactsFragment" android:title="Contacts"/>
</menu>

<!-- menu_toolbar -->
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/aboutFragment" android:title="About"/>
</menu>
{% endhighlight %}