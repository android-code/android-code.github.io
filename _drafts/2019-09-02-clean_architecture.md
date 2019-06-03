---
layout: post
title: "Clean Architecture"
date:  2019-09-02
categories: ["Wzorce architektoniczne"]
permalink: /blog/wzorce/:title/
image: architecture/clean_architecture
github: architectural-patterns/tree/master/clean_architecture
description: "Wzorce projektowe / architektoniczny"
keywords: "clean, architecture, wzorzec, pattern, architektura, warstwy, zaleznosci, layers, dependencies, presentation, domain, data, framework, use cases, repositories, sources, android, kotlin, programowanie, programming"
---

## Zasada zależności
`Clean architecture` nie jest sam w sobie wzorcem architektonicznym lecz propozycją na organizację architektury aplikacji w taki sposób, aby różne obszary oprogramowania były łatwo wymienne, a projekt mógł zostać w całości zaadaptowany w innym środowisku uruchomieniowym bez względu na wykorzystywany framework (np. spójny kod dla aplikacji mobilnej i internetowej). Kod dzielony jest na `warstwy` przypominające koncentryczne `okręgi` zgodnie z zasadą, że `warstwy wewnętrzne` nie wiedzą nic o `warstwach zewnętrznych`, a co za tym idzie nie posiadają do nich `zależności`. Innymi słowy zależności kodu źródłowego mogą wskazywać tylko do wewnątrz. Dotyczy to każdej jednostki kodu tzn. klasy, funkcji, zmiennej itd. Okręgi reprezentują różne obszary oprogramowania, gdzie zewnętrzne koła są mechanizmami niskiego poziomu, a wewnętrzne zasadami wysokiego poziomu. Im głębsza warstwa tym większy poziom abstrakcji. Przekraczanie granic bez naruszania zasady zależności wartwy możliwe jest za pomocą `inwersji zależności` (`dependency inversion`) realizowanej przy użyciu różnych technik programistycznych (np. `polimorfizm`).

## Warstwy
W klasycznym podejściu można wyróżnić cztery warstwy: `Entities`, `Use Cases`, `Interface Adapters`, `Framework and Drivers`. Jednakże nie jest to ogólna i jedyna słuszna propozycja. Podział oprogramowania na warstwy zależy od środowiska, wielkości i złożoności aplikacji, postawionych wymagań oraz kalkulacji kosztów i zysków. Równie dobrze może się okazać, że optymalna implementacja realizowana jest w oparciu inną liczbę okręgów. `Warstwa Entities` to reguły biznesowe wspólne dla wszystkich aplikacji w projekcie. Mogą to być obiekty z metodami czy też zbiory struktur danych i funkcji. `Warstwa Use Cases` zawiera reguły biznesowe specyficzne dla danej aplikacji. Implementuje przypadki użycia systemu, które sterują przepływem danych między podmiotami. `Warstwa Interface Adapters` jest zestawem adapterów odpowiedzialnych za konwersje danych z warstw `Use Cases` i `Entities` do zewnętrznych agentów. To tutaj przeważnie znajdują się klasy implementujące architekturę aplikacji (np. `View`, `Presenter`, `Controller`). `Warstwa Framework and Drivers` składa się z różnych zewnętrznych bibliotek, sterowników i narzędzi. W tym miejscu pojawiają się wszystkie szczegóły zewnętrznych wywołań, które są przekazywane do kolejnych wewnętrznych okręgów.

![Clean Architecture diagram](/assets/img/diagrams/architecture/clean_architecture.svg){: .center-image }

## Zastosowanie
Zastosowanie dowolnej architektury systemu pomimo różnic posiada jeden wspólny cel, podział odpowiedzialności i zagrożeń poprzez rozdzielenie oprogramowania na warstwy. `Clean Architecture` wpisuje się w ten trend i podobnie jak inne architektury jego użycie dostarcza wielu korzyści. Pozwala na tworzenie systemów niezależnych od framework dzięki czemu biblioteki moga być wymienne i traktowane jako narzędzia. Ułatwia testowanie reguł biznesowych ze względu na ich separacje od interfejsu użytkownika i źródeł danych. Ponadto umożliwia zmianę interfejsu użytkownika oraz źródeł danych bez dokonywania modyfikacji w pozostałych obszarach kodu. `Clean Architecture` nie jest związany z żadną konkretną architekturą w związku z czym może zostać zaadaptowany do większości wzorców architektonicznych takich jak `MVC`, `MVP`, `MVMM`, `MVI`.

## Android
Jedną z najpopularniejszych praktyk realizacji `Clean Architecture` dla `Android` jest wyróżnienie trzech warstw: `Presentation`, `Domain`, `Data`. Warstwa `Presentation` zawiera interfejs użytkownika oraz jego obsługę, `Domain` posiada klasy danych i definicję przypadków użycia natomiast `Data` składa się z repozytoriów i zewnętrznych źródeł danych. Odwołując się do klasycznej definicji `Clean Architecture` możliwe jest zastosowanie jeszcze szerszego podziału poprzez wyłączenie m.in. przypadków użycia z `Domain` do osobnej warstwy `Use Cases` oraz właściwych źródeł danych do warstwy `Framework`.

![Clean Architecture Android diagram](/assets/img/diagrams/architecture/clean_architecture_flow.svg){: .center-image }

## Przykład
Aplikacja Filmhead umożliwia zarządzanie prywatną listą filmów użytkownika. Filmy mogą zostać dodane do listy obserwowanych, natomiast te obejrzane mogą być ocenione. Projekt posiada dedykowane aplikacje w wersji `przeglądarkowej` oraz na urządzenia z systemem `Android`. Z uwagi na zachowanie spójności działania aplikacji na wszystkich platformach oraz współdzielenie części kodu zdecydowano się na zastosowanie `Clean Architecture` w wariancie pięciu warstw: `Presentation`, `Use Cases`, `Domain`, `Data`, `Framework`. W przypadku aplikacji mobilnej należy podjąć także decyzję o wyborze wzorca architektonicznego (np. `MVP`).

{% highlight kotlin %}
//DOMAIN
data class Film (val title: String, val vote: Int?)
{% endhighlight %}

W warstwie `Domain` definiowane są modele danych wykorzystywane przez kolejne warstwy. Klasyczny `Clean Architecture` proponuje posiadanie jednej reprezentacji modelu danego bytu dla każdej warstwy w celu całkowitej niezależności warstw od modeli wewnętrznych. Jednakże w wielu przypadkach może się to wiązać z przerostem formy nad treścią.

{% highlight kotlin %}
//DATA
class FilmsRepository (private val localSource: FilmLocalSource, private val remoteSource: FilmRemoteSource) {

    fun getFilms() : List<Film> {
        val remotes = remoteSource.downloadFilms()
        if(remotes.isNotEmpty()) {
            localSource.merge(remotes)
            return remotes
        }
        else {
            return localSource.getLocalFilms()
        }
    }

    fun putFilm(film: Film) {
        localSource.saveFilm(film)
        remoteSource.uploadFilm(film)
    }
}

interface FilmLocalSource {

    fun getLocalFilms() : List<Film>
    fun saveFilm(film: Film)
    fun merge(films: List<Film>)
}

interface FilmRemoteSource {

    fun downloadFilms() : List<Film>
    fun uploadFilm(film: Film)
}
{% endhighlight %}

Warstwa `Data` odpowiedzialna jest za definicję repozytoriów (`Repositories`) realizujących logikę biznesową żądań poprzez wywołanie odpowiednich operacji na deklarowanych źródłach danych (`Sources`).

{% highlight kotlin %}
//USE CASES
class GetFilms (private val filmsRepository: FilmsRepository) {

    operator fun invoke(): List<Film> = filmsRepository.getFilms()
}

class AddFilm (private val filmsRepository: FilmsRepository) {

    operator fun invoke(film: Film) = filmsRepository.putFilm(film)
}
{% endhighlight %}

Warstwa `Use Cases` konwertuje akcje i zdarzenia użytkownika oraz systemu do żądań delegowanych do kolejnych wewnętrznych warstw.

{% highlight kotlin %}
//FRAMEWORK
class RoomFilmsSource : FilmLocalSource {

    //mock implementation of Room database
    private var items = mutableListOf<Film>()

    override fun getLocalFilms(): List<Film> {
        return items
    }

    override fun saveFilm(film: Film) {
        items.add(film)
    }

    override fun merge(films: List<Film>) {
        for(film in films) {
            if(!items.contains(film))
                items.add(film)
        }
    }
}

class RetrofitFilmsSource : FilmRemoteSource {

    //mock implementations of Retrofit framework
    private val items = mutableListOf<Film>()

    override fun downloadFilms(): List<Film> {
        return items
    }

    override fun uploadFilm(film: Film) {
        items.add(film)
    }
}
{% endhighlight %}

Warstwa `Framework` wykorzystuje specyficzne zewnętrzne zależności (np. biblioteka systemowa, wybrany framework) implementując szczegóły realizacji przypadków użycia. 

{% highlight kotlin %}
//PRESENTATION
data class Film (val title: String, val status: String)

//for this layer use own Film model, convert Film model from domain
//use Kotlin import as feature
fun DomainFilm.toPresentation(): Film {
    if(vote == null || vote == 0) return Film(title, "To watch!")
    else return Film(title, "$vote/10")
}

class FilmsPresenter (private val view: FilmsView, private val getFilms: GetFilms, private val addFilm: AddFilm) {

    fun init() {
        view.showProgress(true)
        val films = getFilms.invoke()
        view.renderFilms(films.map (DomainFilm::toPresentation) )
        view.showProgress(false)
    }

    fun uninit() {

    }

    fun addFilmClicked(title: String, vote: Int?) {
        view.showProgress(true)
        val film = Film(title, vote)
        addFilm.invoke(film)
        view.renderNewFilm(film.toPresentation())
        view.showProgress(false)
    }
}

interface FilmsView {

    fun showProgress(enable: Boolean)
    fun renderFilms(films: List<Film>)
    fun renderNewFilm(film: Film)
}

class MainActivity : AppCompatActivity(), FilmsView {

    private val filmsAdapter = FilmsAdapter()
    private val presenter: FilmsPresenter

    init {
        //mostly use dependency injection instead of manual creating
        val room = RoomFilmsSource()
        val retrofit = RetrofitFilmsSource()
        val repository = FilmsRepository(room, retrofit)

        presenter = FilmsPresenter(this, GetFilms(repository), AddFilm(repository))
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        recyclerView.apply {
            adapter = filmsAdapter
            layoutManager = LinearLayoutManager(this@MainActivity)
        }
        button.setOnClickListener { 
        	//this is mock implementation
        	//show some add dialog or activity/fragment instead of
        	presenter.addFilmClicked("Film", (0..10).random()) 
        }

        presenter.init()
    }

    override fun onDestroy() {
        presenter.uninit()
        super.onDestroy()
    }

    override fun renderFilms(films: List<Film>) {
        filmsAdapter.items.clear()
        filmsAdapter.items.addAll(films)
        filmsAdapter.notifyDataSetChanged()
    }

    override fun renderNewFilm(film: Film) {
        filmsAdapter.items.add(film)
        filmsAdapter.notifyDataSetChanged()
    }

    override fun showProgress(enable: Boolean) {
        //show or hide some progress
    }
}
{% endhighlight %}

Warstwa `Presentation` odpowiedzialna jest za prezentację i obsługę interfejsu graficznego użytkownika. W tym miejscu poza widokiem definiowane są klasy architektury (np. `MVP`).