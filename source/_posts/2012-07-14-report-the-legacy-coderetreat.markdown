---
layout: post
title: "Report: The Legacy Coderetreat"
date: 2012-07-14 10:02
comments: true
categories: 
---

Iedere software ontwikkelaar heeft er mee te maken. Sommige vinden het leuk, maar vele hebben er een hekel aan. Je kunt het erven van een ander, of tegenkomen van jezelf. Soms is het jaren oud zijn, of vorige week geschreven. Het kan ontwikkeld zijn in Cobol, maar ook in de laatste versie van je favoriete moderne taal. Een ding is zeker, elke dag komt er meer bij. Legacy code, een fenomeen dat we maar al te goed kennen. Toch weten we er maar weinig van. Wat is het eigenlijk precies en hoe kun je er het beste mee omgaan? Op zaterdag 23 juni kwamen we bij elkaar in het hoofdkantoor van [Finalist](http://www.finalist.nl/) om antwoorden te vinden op deze vragen. 

Onder leiding van [Erik Talboom](https://twitter.com/talboomerik) en [Frederik Delbroek](https://twitter.com/frederikdelbr) zijn we aan de slag gegaan. Het doel was om [Trivia](https://github.com/jbrains/trivia) aan te gaan pakken. Een variant van Triviant dat duidelijk de stempel legacy code verdient. Geen tests, documentatie ontbreekt, de kwaliteit lijkt ver te zoeken en functioneel schort er ook nog het een en ander aan.

Het eerste wat in mij op kwam was als een gek door de code te stormen en alle troep die ik tegen kwam op te ruimen. Maar Erik en Frederik hadden een ander plan. Ze wouden ons leren dat er een betere manier is om met legacy code om te gaan. Dat je het niet moet behandelen met brute kracht, maar met een fluwelen handschoen. Hun plan was om in vier iteraties met ieder een duidelijk doel te gebruiken om in gecontroleerde stappen naar verbetering toe te werken.

Ik koos voor de C# variant op Mono met MonoDevelop als IDE, een keuze waar ik later spijt van zal krijgen. Andere kozen voor Java, PHP, Javascript, Ruby en Scala.

## Iteratie 1: Uitvoerbaar krijgen

Het doel van deze eerste iteratie was het verkennen van de solution en code. We mogen geen wijzigingen aanbrengen anders dan die nodig zijn om de code uitvoerbaar te krijgen.

Na een [fork](https://github.com/pjvds/trivia/) gemaakt te hebben clone ik de git repository en het open de solution. Ik zie drie klasse: `Program`, `GameRunner` en `Game`. Zodra ik de solution probeer te bouwen krijg ik een error. Er ontbreekt een startup object. Ik verwijder de lege `Program` klasse deze en stel de `GameRunner` klasse in als startup object in. De solution bouwt en is zelfs uitvoerbaar. Ik gebruik de overige tijd om een indruk te krijgen van de code. Het volgende valt mij op:

* `GameRunner` kan dienen als startup object.
* De `Game` klasse bevat de spellogica en state.
* De `GameRunner` lijkt een nieuw spel aan te maken en random te spelen totdat er een winnaar is.
* Er zijn geen tests.
* Documentatie ontbreekt.
* Ik zie veel code duplicatie.
* Code is alles behalve defensief.
* Overal waar ik kijk zie ik magic strings en nummer.

Aan het einde van deze iteratie ben ik blij dat ik deze code heb kunnen uitvoeren. Ondanks het ontbreken van tests en documentatie lijkt de code het wel te doen. Wat ik erg moeilijk vond was het niet direct doorvoeren van wijzigingen. Maar door mijn tijd echt te gebruiken om een indruk te krijgen van het totaal beeld heb ik nu een beter overzicht dan als ik gelijk zou beginnen met het aanbrengen van wijzigingen.

### Commits

* [Makes solution buildable and runnable.](https://github.com/pjvds/trivia/commit/a0df78efe77664a8c87a7a6b1cc3d979c92ec280)


## Iteratie 2: Onthullen van intentie

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

## Iteratie 3: Een veiligheidsnet

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

## Iteratie 4: Fix & refactor

In de vorige iteraties is er gewerkt aan het begrijpen van de code en het domein, het verbeteren van de kwaliteit in naamgeving en het creëren van geautomatiseerde testen. Alle wijzigingen die gemaakt zijn aan de code zijn renames geweest. Er is geen code verplaatst of aangepast. Het doel van deze iteratie is om de code aan te gaan passen in de vorm van refactoring en bug fixes. Het begrip als resultaat van iteratie een en twee, en de tests van iteratie drie vormen hiervoor de basis.

### Commits

* [Fixes bug that player is not removed from penalty box](https://github.com/pjvds/trivia/commit/e38d850adf1db03f40a5f3c0acb19ddbf4d51af1)
* [Adds IsCurrentPlayerInPenaltyBox method. Method is used in guard clauses](https://github.com/pjvds/trivia/commit/e9a5bde01b3e856a4e9d363819f2e3850410ec1b)
* [Refactores guard clauses and hiddes some implementation details](https://github.com/pjvds/trivia/commit/36af6dfcaf031583f9ee4fb08f5fa151777a188e)
* [Fixes all tests](https://github.com/pjvds/trivia/commit/eeff46347511242ddaf280c085cd9409ede15739)

## Andere bronnen

* Het book [working effective with legacy code](amzn.com/0131177052) van Micheal Feathers.
