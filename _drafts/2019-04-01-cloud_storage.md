---
layout: post
title: "Cloud Storage"
date:  2019-04-01
categories: ["Firebase"]
image: firebase/cloud_storage
github: firebase/tree/master/cloud_storage
description: "Firebase"
version: Firebase-Storage 16.0
keywords: "firebase, chmura, dane, pliki, cloud, storage, files, reference, storagereference, task, pobieranie, wysylanie, usuwanie, upload, download, delete, metadata, android, programowanie, programming"
---

## Wprowadzenie
`Cloud Storage` jest wydajną, skalowalną i prostą usługą przechowywania plików aplikacji i użytkowników w chmurze (obrazy, audio, wideo itp). Niezależnie od jakości połączenia sieci pobieranie i przesyłanie jest niezawodne nawet w przypadku utraty połączenia, ponieważ wznawia się w miejscu zatrzymania co pozwala zaoszczędzić czas i przepustowość użytkowników. Operacje w Cloud Storage wykonywane są z zachowaniem reguł bezpieczeństwa i prywatności oraz mogą być integrowane z uwierzytelnianiem `Firebase Authentication`. Pliki przechowywane są w segmencie Google Cloud Storage dzięki czemu dostępne są zarówno w `Firebase` jak i `Google Cloud` co umożliwia przetwarzanie przesłanych plików przez użytkowników po stronie serwera. Dodatkowo istnieje możliwość rozszerzenia Cloud Storage o usługę `Cloud Functions`, czyli uruchamianie funkcji w odpowiedzi na przechwycone zdarzenia z Cloud Storage.

![Pliki](/assets/img/diagrams/firebase/storage_files.png){: .center-image }

## Referencje
Deklaratywny język reguł pozwala określić sposób w jaki dane są indeksowane, strukturyzowane oraz zdefiniować poziom dostępu. Domyślnie dostęp do odczytu i zapisu przyznany jest tylko dla uwierzytelnionych użytkowników. Reguła publicznego dostępu umożliwia dostęp do zasobów nie tylko użytkownikom aplikacji, ale nawet wszystkim którzy z niej nie korzystają. Cloud Storage pozwala na dodatkowe skonfigurowanie przestrzeni ze względu na rejon geograficzny, przeznaczenie zasobów czy też ich współdzielenie.

{% highlight kotlin %}
private fun getStorages() {
    val storage = FirebaseStorage.getInstance() //just get FirebaseStorage instance
    val customStorage = FirebaseStorage.getInstance("gs://custom-bucket")
}
{% endhighlight %}

Podobnie jak system plików na dysku twardym pliki w Cloud Storage przedstawione są w strukturze hierarchicznej do których dostęp odbywa się poprzez utworzenie referencji metodą `getReference`. Innymi słowy referencja może być traktowana jako wsaźnik do pliku w chmurze. Nawigacja po poziomach struktury plików odbywa się przy użyciu metod `getChild`, `getParent`, `getRoot`.

{% highlight kotlin %}
private fun getReferences(storage: StorageReference) {
    val storageRef = storage.reference
    val folderRef: StorageReference? = storageRef.child("images")

    val fileRef = storageRef.child("images/file.jpg")
    val parentRef = fileRef.parent //equivalent of folderRef
    val rootRef = fileRef.root //equivalent of storageRef
    val anotherFileRef = fileRef.parent?.child("anotherFile.png") //chain navigation together
}
{% endhighlight %}

Każda referencja posiada właściwości opisujące plik takie jak m.in. ścieżka, nazwa, przestrzeń czy metadane.

{% highlight kotlin %}
private fun getInfo(fileRef: StorageReference) {
    val info = "$fileRef.name in $fileRef.path in $fileRef.bucket bucket"
    val metadata = fileRef.metadata
}
{% endhighlight %}

## Operacje
Wszystkie operacje na plikach, tzn. wysyłanie, pobieranie, usuwanie i zmiana metadanych odnoszą się do instancji `StorageReference` i mogą być wykonane dla dowolnej lokalizacji za wyjątkiem nadrzędnego `root`. Wysyłanie plików przeprowadza się za pomocą metod `putFile`, `putBytes`, `putStream`, pobieranie następuje przy użyciu metod `getFile`, `getBytes`, `getStream`, usuwanie zachodzi przy wykorzystaniu metody `delete`, a aktualizacja metadanych metodą `updateMetadata`.

{% highlight kotlin %}
private fun startUploadFile() {
    //get file and create reference
    val storageRef = FirebaseStorage.getInstance().reference
    val file = Uri.fromFile(File("path/file.jpg"))
    val fileRef = storageRef.child("images/${file.lastPathSegment}")

    //create custom metadata if needed
    val metadata = StorageMetadata.Builder().setContentType("image/jpg").build()

    //upload file
    val uploadTask = fileRef.putFile(file)
    //or fileRef.putFile(file, metadata) for using custom metadata
}

private fun startDownloadFile() {
    val storageRef = FirebaseStorage.getInstance().reference
    val file = File.createTempFile("images", "jpg")
    val fileRef = storageRef.child("images/file.jpg")
    val downloadTask = fileRef.getFile(file)
}

private fun startDeleteFile() {
    val storageRef = FirebaseStorage.getInstance().reference
    val fileRef = storageRef.child("images/file.jpg")
    val deleteTask = fileRef.delete()
}

private fun startUpdateMetadata() {
    val storageRef = FirebaseStorage.getInstance().reference
    val fileRef = storageRef.child("images/file.jpg")

    val metadata = StorageMetadata.Builder()
        .setContentType("image/jpg")
        .setCustomMetadata("myCustomProperty", "myValue")
        .build()

    val metadataTask = fileRef.updateMetadata(metadata)
}
{% endhighlight %}

## Status
Rozpoczęcie wykonywania zadania zwraca właściwy dla danej operacji obiekt rozszerzający klasę `StorageTask`, który pozwala na zarządzanie i monitorowanie statusu operacji poprzez podpięcie słuchaczy dla różnych stanów, tj. `OnProgressListener`, `OnPausedListener`, `OnSuccessListener`, `OnFailureListener`. Metody `pause`, `resume`, `cancel` umożliwiają manualne zarządzanie statusem zadania. W przypadku zamknięcia procesu aplikacji wszystkie zadania są przerywane. Należy więc zadbać o ich właściwe wznowienie w miejscu przerwania (nie od początku) co pozwoli na oszczędzenie czasu i transmisji danych użytkownika.

{% highlight kotlin %}
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    uploadButton.setOnClickListener { startUploadFileWithListeners() }
    cancelButton.setOnClickListener { uploadTask?.cancel() }

    //check if upload task has been interrupted
    val taskInterrupted = false //mock, get from persistent storage
    if(taskInterrupted) {
        restartUploadFile()
    }
}

private fun startUploadFileWithListeners() {
    val storageRef = FirebaseStorage.getInstance().reference
    val file = Uri.fromFile(File("path/file.jpg"))
    fileRef = storageRef.child("images/${file.lastPathSegment}")

    //start and add listeners
    uploadTask = fileRef.putFile(file)
    uploadTask.addOnProgressListener { taskSnapshot ->
        //save session URI in persistent storage in case of process die
        val sessionUri = taskSnapshot.uploadSessionUri
        //show some progress
    }.addOnFailureListener {
        //action on error like show message
    }.addOnSuccessListener {
        //action on success like show file in UI
    }
}

private fun restartUploadFile {
    val storageRef = FirebaseStorage.getInstance().reference
    val file = Uri.fromFile(File("path/file.jpg"))
    val sessionUri = Uri.EMPTY //mock value, get from persistent storage
    uploadTask = storageRef.putFile(file, StorageMetadata.Builder().build(), sessionUri)
}

//do in the similar way for other operations
{% endhighlight %}

## Cykl życia
Przesyłanie kontynuowane jest w tle niezależnie od zmian cyklu życia `Aktywności` co pomimo niewątpliwej zalety może prowadzić do wycieku pamięci dla dodanych wcześniej obserwatorów. Aby temu zapobiec należy we właściwym miejscu cyklu życia wyrejestrować obiekty słuchaczy, a następnie zarejestrować ponownie pobierając instancje bieżącego zadania za pomocą metody `getActiveUploadTask` lub `getActiveDownloadTask`.

{% highlight kotlin %}
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    //save the reference to upload task if is in progress
    fileRef?.let { outState.putString("reference", it.toString()) }
    //remove passed listeners
}

override fun onRestoreInstanceState(savedInstanceState: Bundle) {
    super.onRestoreInstanceState(savedInstanceState)

    //get reference to upload task if was in progress
    val reference = savedInstanceState.getString("reference")
    if(reference != null) {
        fileRef = FirebaseStorage.getInstance().getReferenceFromUrl(reference)

        //get all UploadTask - in this case there should be one
        fileRef?.activeUploadTasks?.let { it ->
            if (it.size > 0) {
                val task = it[0]
                //set listeners to task here
            }
        }
    }
}
{% endhighlight %}
