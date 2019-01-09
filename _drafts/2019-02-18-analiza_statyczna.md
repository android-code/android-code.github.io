---
layout: post
title: "Analiza statyczna"
date:  2019-02-18
categories: ["Testowanie"]
image: testing/static_analysis
github: testing/tree/master/static_analysis
description: "Testowanie"
keywords: "testowanie, testing, testy, analiza, statyczna, gradle, lint, pmd, findbugs, android, programowanie, programming"
---

## Definicja
Statyczna analiza kodu wykorzystywana jest w celu znajdywania błędów w kodzie, wyszukiwanie luk i niedopatrzeń oraz do sprawdzania czy napisany kod spełnia zasady dobrego programowania i przestrzega wytycznych. Co więcej pozwala na detekcję hipotetycznych błędów, które mogły nie zostać wykrytę przez testy jednostkowe i manualne, a także ułatwia definiowanie i przestrzeganie standardów kodu w zespole. Dzięki temu utrzymanie odpowiedniej jakości kodu staje się łatwiejsze. Statyczna analiza jest częścią procesu ciągłej integracji (continous integration) i może być wykorzystywana również jako narzędzie wspierające testowanie (białoskrzynkowe) oraz refactoring. Dokonywana analiza przeprowadzana jest bez wykonania kodu i odbywa się przy pomocy różnych narzędzi analitycznych takich jak np. lint, pmd, findbugs czy checkstyle dostępnych jako plugin dla Gradle oraz platform wspierających ciągłą integracje jak np. SonarQube. Weryfikują one zgodność kodu w zestawieniu do zbioru reguł. W przypadku niespełnienia zasad informują o potencjalnych błędach i zagrożeniach. 

## Checkstyle
Checkstyle analizuje kod źródłowy weryfikując jego standardy oraz konwencję w stosunku do zbioru wybranych zasad. Skupia się przede wszystkim na analizie stylu kodu (np. nazwenictwo, klamry).

//TODO

## Pmd
PMD podobnie jak checkstyle przeprowadza analizę kodu w stosunku do zbioru zasad, jednakże skupia się on newralgicznych i opuszczonych fragmentach kodu (np. nieużywane zmienne, puste bloki).

//TODO

## Findbugs
Findbugs działając w oparciu o kod bajtowy wykonuje analizę w poszukiwaniu potencjalnych problemów z listy znanych błędów projektowych i złych praktyk (np. jawnie zapisane hasło, klasa testowa nie posiada testów, metoda prywatna nie jest nigdy wywoływana).

//TODO

## Lint
Lint jest narzędziem dostarczonym przez Android Studio, który podobnie jak wspomniane wcześniej pluginy sprawdza pliki źródłowe pod kątem potencjalnych błędów, optymalizacji poprawności, bezpieczeństwa, wydajności, użyteczności i dostępności kodu. Z uwagi na integracje ze środowiskiem programistycznym nie wymaga konfiguracji.

//TODO

## SonarQube
//TODO