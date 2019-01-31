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
Dane reprezentowane są w postaci dokumentów (zbliżonych w stukurze do formatu `JSON`) przechowywanych w zorganizowanej kolekcji w hierarchicznej strukturze bazy danych `NoSQL` (w przeciwieństwie do `SQL` nie ma tabel ani wierszy). Każdy `dokument` składa się z nazwy, zbioru par `klucz - wartość` (prymityw lub obiekt złożony) i może posiadać obiekty zagnieżdzone i podkolekcje. `Kolekcje` pełnią rolę kontenera dla różnych dokumentów i nie mogą posiadać innych kolekcji, a w doborze ich zawartości warto zachować logiczny porządek dzieląc dokumenty ze względu na kategorie i przeznaczenie co upraszcza poruszanie się po strukturze w kodzie klienta. `Podkolekcja` jest kolekcją w dokumencie i służy do budowania zagnieżdzonej struktury folderów. 

//TODO rysunek

Ze względu na optymalizacje wydajności dostępu do bazy danych wprowadzony został mechanizm indeksowania, który wyróżnia dwa rodzaje indeksów: `single-field` (przechowuje posortowaną mapę wszystkich dokumentów zawierających dane pole) i `composite` (przechowuje posortowaną mapę wszystkich dokumentów zawierających wiele danych pól). Cloud Firestore jest zaprojektowany przede wszystkim z myślą o dużych kolekcjach małych dokumentów. Aby uzyskać dostęp do dokumentu lub kolekcji należy uzyskać `referencje`.

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
	person["name"] = "Jasper"
	person["second"] = "Newton"
	person["surname"] = "Daniel"
	person["born"] = 1850

	//add to database by passing those objects
}

private fun createDataByObject() {
	val person = Person("William", "Grant", 1839)
	//add to database by passing object
}
{% endhighlight %}

## Operacje
//TODO

## Transakcje
//TODO