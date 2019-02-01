---
layout: post
title: "Cloud Firestore"
date:  2019-04-08
categories: ["Firebase"]
image: firebase/cloud_firestore
github: firebase/tree/master/cloud_firestore
description: "Firebase"
version: Firebase-Firestore 18.0
keywords: "firebase, chmura, dane, baza, cloud, storage, database, nosql, android, programowanie, programming"
---

## Wprowadzenie
`Cloud Firestore` jest elastyczną, skalowalną, hierarchiczną bazą danych `NoSQL` w chmurze służącą do przechowywania i synchronizowania w czasie rzeczywistym danych między klientami i serwerem. Oferuje wsparcie w trybie `offline` co ułatwia budowania responsywnych aplikacji działających niezależnie od jakości połączenia sieciowego. Umożliwia integracje z `Google Cloud Platform` i funkcjonalnościami `Firebase` jak np. `Cloud Functions` czy `Authentication` oraz innymi zewnętrznymi usługami. Dostęp i modyfikacja danych odbywa się zgodnie z zasadami zachowania bezpieczeństwa i prywatności, które mogą być dodatkowo definiowane za pomocą reguł w konsoli Firebase.

## Model
Dane reprezentowane są w postaci dokumentów (zbliżonych w stukurze do formatu `JSON`) przechowywanych w zorganizowanej kolekcji w hierarchicznej strukturze bazy danych `NoSQL` (w przeciwieństwie do `SQL` nie ma tabel ani wierszy). Każdy `dokument` (`document`) składa się z nazwy, zbioru par `klucz - wartość` (prymityw lub obiekt złożony) i może posiadać obiekty zagnieżdzone i podkolekcje. `Kolekcje` (`collection`) pełnią rolę kontenera dla różnych dokumentów i nie mogą posiadać innych kolekcji, a w doborze ich zawartości warto zachować logiczny porządek dzieląc dokumenty ze względu na kategorie i przeznaczenie co upraszcza poruszanie się po strukturze w kodzie klienta. `Podkolekcja` jest kolekcją w dokumencie i służy do budowania zagnieżdzonej struktury folderów.

//TODO rysunek

Ze względu na optymalizacje wydajności dostępu do bazy danych wprowadzony został mechanizm indeksowania, który wyróżnia dwa rodzaje indeksów: `single-field` (przechowuje posortowaną mapę wszystkich dokumentów zawierających dane pole) i `composite` (przechowuje posortowaną mapę wszystkich dokumentów zawierających wiele danych pól). Cloud Firestore jest zaprojektowany przede wszystkim z myślą o dużych kolekcjach małych dokumentów. Aby uzyskać dostęp do dokumentu lub kolekcji należy uzyskać `referencje` - obiekt typu `DocumentReference`.

{% highlight kotlin %}
private fun createReferences() {
    val database = FirebaseFirestore.getInstance()
    val collectionRef = database.collection("collection")
    val documentRef = collectionRef.document("document")
    val documentRef2 = database.document("collection/document") //or use direct path instead
    val nestedDocumentRef = documentRef.collection("subcollection").document("nested")
}
{% endhighlight %}

## Typy
Dokumenty mogą przechowywać wartości `prymitywów` (numeryczne, logiczne, tekstowe), `obiekty złożone` (koordynaty geograficzne, data i czas), `tablice`, `mapy` i `referencje` do innych dokumentów.

{% highlight kotlin %}
private fun createExampleData() {
    val document = HashMap<String, Any?>()
    document["text"] = "value"
    document["logic"] = true
    document["number"] = 9.99
    document["date"] = Timestamp(Date())
    document["list"] = arrayListOf(1,2,3)
    document["null"] = null
}

private fun createSimilarData() {
    val person = HashMap<String, Any>()
    person["name"] = "John"
    person["surname"] = "Walker"
    person["born"] = 1805

    //structure of data in the same collection don't have to be exactly the same
    val person2 = HashMap<String, Any>()
    person2["name"] = "Jasper"
    person2["second"] = "Newton"
    person2["surname"] = "Daniel"
    person2["born"] = 1850

    //add to database by passing those objects
}

private fun createDataByObject() {
    val person = Person("William", "Grant", 1839)
    //add to database by passing object
}
{% endhighlight %}

## Zapis
Aby dokonać zapisu w Cloud Firestore należy podać identyfikator dokumentu oraz przekazać obiekt danych metodą `set` lub w przypadku automatycznego generowania id wystarczy tylko przekazać danę metodą `add`. Jeśli dokument nie istnieje wówczas zostanie utworzy, w przeciwnym razie jego zawartośc zostanie nadpisana, chyba że zostaną użyte odpowiednie opcje. Dodanie obserwatorów umożliwia śledzenie stanu operacji.

{% highlight kotlin %}
private fun addDataWithId() {
    //create data
    var club = Club("Milan", "Italy", 1899)
    club.uclTrophiesYear = listOf(1963, 1993, 1989, 1990, 1994, 2003)
    club.budget = 1000L

    //put data to database
    database.collection("club").document("acmilan").set(club)
        .addOnSuccessListener {
            //some action
        }
        .addOnFailureListener {
            //some action
        }

    //or use reference.set(club, SetOptions.merge()) for merge if file exists
}

private fun addDataWithoutId() {
    var club = Club("Chelsea FC", "England", 1905)
    database.collection("club").add(club)
        .addOnSuccessListener { 
            //some action 
        }
        .addOnFailureListener { 
            //some action 
        }
}
{% endhighlight %}

W celu aktualizacji wybranych pól dokumentu bez nadpisywania całej jego zawartości należy użyć metodę `update` na referencji. Odwołując się do obiektów zagnieżdzonych należy wykorzystać właściwą notację oddzielając pola i wartości przecinkiem, a w przypadku modyfikacji elementów tablic użyć metod `FieldValue.arrayUnion` oraz `FieldValue.arrayRemove`.

{% highlight kotlin %}
private fun updateData() {
    val reference = database.collection("club").document("acmilan")
    reference.update("name", "AC Milan")
    //add some listeners
}

private fun updateArrayData() {
    //add new data
    database.collection("club").document("acmilan").set(club, SetOptions.merge())

    reference.update("uclTrophiesYear", FieldValue.arrayUnion(2007))
    //add some listeners
}
{% endhighlight %}

Usunięcie dokumentu następuje poprzez wywołanie metody `delete` na referencji dokumentu (nie powoduje usunięcia jego podkolekcji), natomiast przypisanie wartości `FieldValue.delete()` do pola w operacji `update` powoduje jego usunięcie. Usuwanie całych kolekcji po stronie klient jest niezalecane ze względu na zagrożenia wydajności, wycieku pamięci, bezpieczeńśtwa po stronie klienta wynikające z potrzeby wysyłania wielu żądań.

{% highlight kotlin %}
private fun deleteDocument() {
    database.collection("club").document("acmilan").delete()
    //add some listeners
}

private fun deleteField() {
    val reference = database.collection("club").document("acmilan")

    val updates = HashMap<String, Any>()
    updates["uclTrophiesYear"] = FieldValue.delete()

    reference.update(updates)
    //add some listeners
}
{% endhighlight %}

## Odczyt
Pobieranie dokumentów odbywa się przez wywołanie metody `get` na referencji dokumentu lub kolekcji. Dodatkowo można ustawić źródło pobierania danych na `serwer` lub pamięć `cache` oraz `parsować` otrzymane dane do instancji klasy. Rozszerzenie żądania o żłożone zapytania pozwala filtrowanie (`where`), sortowanie (`orderBy`) i ograniczanie liczby wyników spełniających założenia (`limit`).

{% highlight kotlin %}
private fun getDocument() {
    val reference = database.collection("club").document("acmilan")

    //add optional source and listeners and retrieve document
    val source = Source.SERVER //change to Source.CACHE for cached data
    reference.get(source)
        .addOnSuccessListener { document ->
            //some action on DocumentSnapshot instance
            val club = document.toObject(Club::class.java)
        }
        .addOnFailureListener { exception ->
            //some action 
        }
}

private fun getMultipleDocuments() {
    //add some listeners and optional query clausule
    database.collection("club").whereEqualTo("country", "Italy").get()
        .addOnSuccessListener { documents ->
            //some action on QueryDocumentSnapshot instance
            for(document in documents) { }
        }
}

private fun getDocumentsByQueries() {
    //add queries and listeners
    database.collection("club")
        .whereEqualTo("country", "Italy")
        .whereLessThan("year", 1930)
        .orderBy("year", Query.Direction.DESCENDING)
        .limit(5)
        .get()
        .addOnSuccessListener { documents ->
            //some action on QueryDocumentSnapshot instance
        }
}
{% endhighlight %}

Ponadto istnieje możliwość nasłuchiwania modyfikacji danych dla dokumentu i kolekcji w czasie rzeczywistym poprzez dodanie obiektu słuchacza z uwzględnieniem źródła i typu zmian.

{% highlight kotlin %}
private fun listenForDocumentChanges() {
    val reference = database.collection("club").document("acmilan")

    reference.addSnapshotListener(EventListener<DocumentSnapshot> { snapshot, exception ->
        //check is exception
        if (exception != null) return@EventListener

        //check source of the changes
        if (snapshot != null && snapshot.metadata.hasPendingWrites()) {
            //local changes
        }
        else { 
            //server changes
        }

        //check type of the change
        for (changes in snapshots!!.documentChanges) {
            //could be: DocumentChange.Type.ADDED, MODIFIED or REMOVED
        }
    })

    //do the same on the multiple documents from collection to listen more documents
}
{% endhighlight %}

## Transakcje
Cloud Firestore wspiera atomowe operacje dla zapisu i odczytu, które są implikowane tylko wtedy kiedy wszystkie operacje zbioru zakończą się pozytywnie, zatem wykonanie zbioru operacji atomowych jest spójne i nierozdzielne. Wyróżnia się dwa typy operacji atomowych: `transakcje` (`transaction`) - zbiór operacji odczytu i zapisu na wielu dokumentach oraz `zestaw zapisu` (`batched write`) - zbiór operacji zapisu dla wielu dokumentów. Transakcje pozwalają na grupowanie wielu operacji w jedną transakcje i wykorzystywane są przede wszystkim do modyfikacji dokumentów bazując na ich bieżącym stanie. Wykonywane są za pomocą metody `runTransaction` lub zbioru operacji zapisu na obiekcie `WriteBatch`.

{% highlight kotlin %}
private fun makeTransaction() {
    val reference = database.collection("club").document("acmilan")

    database.runTransaction { transaction ->
        val snapshot = transaction.get(reference)
        val budget = snapshot.getLong("budget")
        if(budget <= 1000)
            transaction.update(reference, "budget", budget + 500L)
    }
}

private fun makeWriteBatch() {
    val batch = database.batch()

    //add new document
    val realMadrid = Club("Real Madrid", "Spain", 1902)
    val rmReference = database.collection("club").document("realmadrid")
    batch.set(rmReference, realMadrid)

    //update current
    val acmReference = databae.collection("club").document("acmilan")
    batch.update(acmReference, "budget", 2000L)

    //do more add, update, delete operations

    //commit operations and add some listeners
    batch.commit.addOnCompleteListener {
        //some action
    }
}
{% endhighlight %}
