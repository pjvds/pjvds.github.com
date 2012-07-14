---
layout: post
title: "Report: The Legacy Coderetreat"
date: 2012-07-14 10:02
comments: true
categories: 
---

Iedere software ontwikkelaar heeft er mee te maken. Sommige vinden het leuk, maar vele hebben er een hekel aan. Je kunt het erven van een ander, of tegenkomen van jezelf. Soms is het jaren oud zijn, of vorige week geschreven. Het kan ontwikkeld zijn in Cobol, of in de laatste versie van een moderne taal. Een ding is zeker, er wordt elke dag een hoop van geschreven. Legacy code, een fenomeen dat we maar al te goed kennen. Toch weten we er maar weinig van. Wat is het eigenlijk precies? Hoe kun je er het beste mee omgaan?

Op zaterdag 23 juni kwamen we bij elkaar in het hoofdkantoor van [Finalist](http://www.finalist.nl/) om antwoorden te vinden op deze vragen. Onder leiding van [Erik Talboom](https://twitter.com/talboomerik) en [Frederik Delbroek](https://twitter.com/frederikdelbr) zijn we aan de slag gegaan om [Trivia](https://github.com/jbrains/trivia) aan te gaan pakken. Een variant van Triviant dat duidelijk de stempel legacy code verdient. Het plan was om in vier iteraties in pairs de code te leren begrijpen en te verbeteren.

Ik koos voor de C# variant op Mono met MonoDevelop als IDE. Andere kozen voor Java, PHP, Javascript, Ruby en Scala.

## Iteratie 1

Het doel van deze iteratie was het verkennen van de solution en code. We krijgen als opdracht mee om alleen te gaan verkennen. We mogen geen wijzigingen aanbrengen, anders dan het uitvoerbaar krijgen van de code.

Na een fork en clone van de git repository en het openen van de solution viel gelijk op dat de code niet bouwt door het missen van een startup object. Ik verwijder de lege `Program` klasse deze en stel de `GameRunner` klasse in als startup object in. De solution bouwt en is zelfs executabel. Ik gebruik de overige tijd om een indruk te krijgen van de code. Het volgende valt mij op:

* Code werkt op Mono.
* Na korte fix is de code uitvoerbaar.
* De `Game` klasse bevat de spellogica en state.
* De `GameRunner` lijkt een nieuw spel aan te maken en random te spelen totdat er een winnaar is.
* Er zijn geen tests.
* Documentatie ontbreekt.
* Ik zie veel code duplicatie.
* Code is alles behalve defensief.
* Overal magic strings en nummer.

### Commits

* [Makes solution buildable and runnable.](https://github.com/pjvds/trivia/commit/a0df78efe77664a8c87a7a6b1cc3d979c92ec280)


## Iteratie 2

Het doel van deze iteratie is het onthullen van de intentie van de code. Refactoring is verboden. We gaan zaken alleen een betere naam geven. Ik werk de code van boven tot beneden door waarbij ik opzoek ga naar magic numbers en strings. Daarna zijn de methode namen aan de beurt. Ik geef alle methodes de volledige naam die ze verdienen. Een paar voorbeelden:

* Magic string door categories `"Pop"`, `"Science"`, `"Sport"` en `"Rock"` worden `POP_CATEGORY`, `SCIENCE_CATEGORY`, `SPORT_CATEGORY` en `ROCK_CATEGORY`.
* Magic number `6` wordt ``.
* Variable `players` wordt `playerNames`.
* Variable `purges` wordt `playerCoins`.
* Variable `places` wordt `playerCurrentPlaces`.
* Methode `wasCorrectlyAnswered` wordt `MarkCurrentAnswerAsCorrectAndMoveToNextPlayer`.
* Methode `askQuestion` wordt `printCurrentQuestionAndRemoveIt`

### Commits

* [Removes a bunch of magic numbers/string and renamed methods to reveal intention](https://github.com/pjvds/trivia/commit/c9a4fdabcb9e656276310193c28478a5aa97dc82)
* [Improves code by removing magic's](https://github.com/pjvds/trivia/commit/2f0d31e42d10d6f044601a5e846247a2f7e72e75).

## Iteratie 3

Het doel van deze iteratie is het toevoegen van tests. We gaan een vangnet creëren. Deze hebben we later hard nodig als we gaan refactoren.

Mijn plan van aanpak was het schrijven van functionele integratie tests te gaan schrijven. De functionaliteit zou ik halen door de bestaande code te analyseren. Ik vond de volgende functies:

* Spelers toevoegen aan de game (_shouldAddPlayers_).
* Er zijn minimaal twee spelers nodig om te spelen (_needsTwoPlayersToPlay_).
* Na het geven van een antwoord is de volgende speler aan de beurt (_advancesPlayerAfterAnswer_).
* Een verkeerd antwoord verplaatst de huidige spelen naar de penalty box (_toPenaltyBoxAfterWrongAnswer_).
* Een goed antwoord verplaatst de speler niet naar de penalty box (_notToPenaltyBoxAfterCorrectAnswer_).
* Een speler in de penalty box verlaat deze weer bij het gooien van een oneven getal (_outOfPenaltyBoxAfterOddRoll_).
* Een speler in de penalty box blijft zitten bij het gooien van een even getal (_notOutOfPenaltyBoxAfterEvenRoll_).
* Een goed antwoord wordt beloond met een coin (_getsCoinAfterCorrectAnswer_).
* Een fout antwoord wordt niet beloond met een coin (_doesNotGetCoinAfterWrongAnswer_).
* Na zes goede antwoorden win je de game (_winAfterSixCorrectAnswers_).

Voor deze tests heb ik NUnit gebruikt. Om de state te testen heb ik een aantal interne variabele toegankelijk gemaakt. Een nadeel hiervan is dat de tests nu afhankelijk zijn van implementatie details. Maar omdat ik nog geen code echt wou wijzigen was dit de enige optie.

Het resultaat van de iteratie is iets van een vangnet. Deze vormt de basis voor onze refactoring. De testen tonen ook aan hoe slecht de huidige code er aan toe is. Vijf van tien tests falen!

### Commits

* [Adds tests that cover most game scenario's](https://github.com/pjvds/trivia/commit/d95733b0d95fac480e6fe75f664da7ea0ea4350b)

## Andere bronnen

* Het book [working effective with legacy code](amzn.com/0131177052) van Micheal Feathers.
