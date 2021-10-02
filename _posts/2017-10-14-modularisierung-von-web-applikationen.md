---
title: Modularisierung von Web Applikationen
layout: post
comments: true
published: false
description: 
keywords: 
---

### Einleitung

Ziel des Dokuments ist es, mögliche Wege zur Trennung von Daten und Geschäftslogik aufzuzeigen und diese anhand von praktischen Beispielen zu untermauern. Desweiteren sollen Konventionen und Best Practices die Teamarbeit verbessern, die Produktivtät erhöhen und die Einarbeitungszeit reduzieren.

Folgende Fragen sollten beantwortet werden:

* Wann und wo trennen wir was wie und warum. (Beispiele)
* Wo möglich bzw. sinnvoll bitte generelle oder spezifische Aussagen für unsere CakePHP 2.x und 3.x Applikationen machen.
* Der Vorschlag sollte kleine Projekte wie auch grosse Projekte berücksichtigen.
* Die Konventionen und Best Practices des Frameworks in der jeweiligen Version sind grundsätzlich zu berücksichtigen bzw. zu bevorzugen.

### Systemarchitektur

Die Applikation sollte in einer mehrschichtige Systemarchitektur in folgende Schichten aufgeteilt werden:

1. Präsentationsschicht
2. Logikschicht
3. Datenzugriffsschicht
4. Datenhaltungsschicht

### Präsentationsschicht

Die Präsentationsschicht ist verantwortlich für die Darstellung und Entgegennahme von der Software bearbeiteten Daten, sowie der von der Software bereitgestellten Funktionen. Zu den Aufgaben der Präsentationsschicht gehören:

* Die Benutzerschnittstelle
* Die Ausgaben der Daten 
* Die (einfache) Kontrolle der Benutzereingaben (Validation)

### Logikschicht

Die Geschäftslogik realisiert das eigentliche Geschäftsmodell, indem die am Geschäftsmodell beteiligten Geschäftsobjekte mit der zugehörigen Logik implementiert werden. Zu den Funktionen der Logikschicht gehören:

* Eingabeprüfungen
* Berechnungen durchführen
* Workflow / Verarbeitungsschritte / UseCases
* Sicherheitssystem
* Daten von Datenzugriffsschicht erfragen / weitergeben
* Bietet der Präsentationsschicht oft ein Objektmodell (Domänenmodell) an

### Datenzugriffsschicht

Die Datenzugriffsschicht (Data Access Layer) kappselt die Zugriffe auf die Datenhaltungsschicht (Datenbank) und die dabei verwendeten Techniken. Die Isolierung der Anwendung von Datenspeicher erfolgt mit Hilfe von Techniken wie z.B. einer Datenbankabstraktionsschicht (Database Abstraction Layer, Query Builder) und der darauf aufbauenden objektrelationaler Abbildung (ORM, Data Mapper).

Eine **Datenbankabstraktionsschicht** (englisch database abstraction layer) ist eine Programmierschnittstelle, welche die Verbindung zwischen einer Software-Anwendung und damit zu verbindenden Datenbanken vereinheitlicht. Damit kann ein Verwalter bei der Installation der Anwendung aus einer Reihe möglicher Datenbankprodukte wählen, ohne dass der Programmcode angepasst werden muss.

Für die verschiedenen Datenbanken und Programmierumgebungen hatten sich mit der Zeit eigene Datenbankschnittstellen entwickelt, die teils spezifische Funktionen der Datenbanken zur Verfügung stellen, teils nur Syntax-Abweichungen abbilden. Durch eine Datenbankabstraktionsschicht werden die Syntaxunterschiede ausgeglichen und die Programmierung und somit auch die Wartbarkeit der Software verbessert. Darüber hinaus ist durch eine Datenbankabstraktionsschicht eine gewisse Datenbankunabhängigkeit sichergestellt, womit der Lock-in-Effekt stark reduziert wird.

Die PHP Data Objects-Erweiterung (PDO) stellt eine leichte, konsistente Schnittstelle bereit, um mit PHP auf Datenbanken zuzugreifen. Jeder Datenbanktreiber, der die PDO-Schnittstelle implementiert, kann spezifische Features als reguläre Funktionen der Erweiterung bereitstellen. Beachten Sie, dass Sie keine Funktionen der Datenbank mit PDO allein benutzen können. Sie müssen einen datenbankspezifischen PDO-Treiber benutzen, um auf eine Datenbank zuzugreifen.

PDO bietet eine Abstraktionsschicht für den Datenzugriff, das bedeutet, dass Sie, egal welche Datenbank Sie benutzen, dieselben Funktionen verwenden können, um Abfragen zu erstellen und Daten zu lesen. PDO bietet keine Abstraktion für Datenbanken. Es schreibt keine SQL-Abfragen um oder emuliert fehlende Features. Sie sollten eine komplette Abstraktionsschicht verwenden (z.B. Doctrine DBAL), wenn Sie diese Funktionalität benötigen.

**Objektrelationale Abbildung** (englisch object-relational mapping, ORM) ist eine Technik der Softwareentwicklung, mit der ein in einer objektorientierten Programmiersprache geschriebenes Anwendungsprogramm seine Objekte in einer relationalen Datenbank ablegen kann. Dem Programm erscheint die Datenbank dann als objektorientierte Datenbank, was die Programmierung erleichtert. Implementiert wird diese Technik in PHP normalerweise mit Klassenbibliotheken, wie beispielsweise **Doctrine** (Object Relational Mapper).

### Datenhaltungsschicht

In der Datenhaltungsschicht werden die Entity-Objekte persistent gespeichert. In der Regel handelt es sich dabei im eine relationalen Datenbank. Alternativ sind auch andere Formen der Speicherung möglich, wie z.B. eine nicht relationale Datenbank (NoSQL) oder gar das Dateisystem selbst.

Eine relationale Datenbank dient zur elektronischen Datenverwaltung in Computersystemen und beruht auf einem tabellenbasierten relationalen Datenbankmodell. Dieses wurde 1970 von Edgar F. Codd erstmals vorgeschlagen und ist bis heute trotz einiger Kritikpunkte ein etablierter Standard für Datenbanken.

Das zugehörige Datenbankmanagementsystem wird als relationales Datenbankmanagementsystem oder RDBMS (Relational Database Management System) bezeichnet. Zum Abfragen und Manipulieren der Daten wird überwiegend die Datenbanksprache SQL (Structured Query Language) eingesetzt.

### Trennung Daten & Geschäftslogik

### Examples

Very simple example used as an introduction:

```php
<?php echo 'Hello World';
```

A more complex example would be:

....

## Konventionen und Bewährte Praktiken (Best Practice)

### Codierungsstandards

Ein Codierungsstandard ist eine Menge von Regeln, deren Einhaltung dafür sorgen soll, dass Code les- und wartbar bleibt, vor allem...

* sobald mehr als eine Person denselben Code bearbeiten soll.
* bei Programmen, die zwar nur von einer Person geschrieben werden, aber irgendwann einmal von jemandem anderen gewartet werden sollen.
* wenn man ein Programm auch ein paar Wochen, nachdem man es geschrieben hat, noch selber verstehen möchte.
(Fast) jeder Codierungsstandard ist besser als gar keiner, und es ist sehr oft das kleinere Übel, sich von Anfang an an einen vorgegebenen Codierungsstandard zu halten, als größere Mengen Code nachträglich an diesen anpassen zu müssen.

### Allgemeine Stilkriterien

Der [PSR-1 Basic Coding Standard](http://www.php-fig.org/psr/psr-1/) definiert die Basis Standards, die erforderlich sind, um ein hohes Mass an technischer Interoperabilität zwischen gemeinsam genutztem PHP-Code zu gewährleisten.

Der [PSR-2 Coding Style Guide](http://www.php-fig.org/psr/psr-2/) erweitert den PSR-1 Basic Coding Standard.

Das Ziel dieses Standards ist es, die kognitiven Anstrengungen beim Lesen von Code verschiedener Autoren zu reduzieren. Dies geschieht durch die Definition von Stilregeln für die Formatierung von PHP-Code.

Wenn verschiedene Autoren über mehrere Projekte hinweg zusammenarbeiten, ist es hilfreich, wenn für alle Projekte die selben Richtlinen verwendet werden. Der Nutzen dieses Standards liegt also nicht in den Regeln selbst, sondern in der breiten Anwendung dieser Regeln.

### CakePHP spezifische Konventionen

Als Basis für alle weiteren Konventionen und Best Pratices dienen die CakePHP spezifischen Konventionen. Alle Details zu diesem Thema befinden sich auf diesen Seiten:

* CakePHP 3.x: https://book.cakephp.org/3.0/en/intro/conventions.html
* CakePHP 2.x: https://book.cakephp.org/2.0/en/getting-started/cakephp-conventions.html


### Allgemeine Best Pratices

## Bewährte Praktiken und Prinzipien

Mit folgende Prinzipien solle sich jeder Entwickler vertraut machen in der Praxis umsetzen:

* [KISS](https://en.wikipedia.org/wiki/KISS_principle), [KISS](https://people.apache.org/~fhanik/kiss.html) 
* Don’t repeat yourself (DRY) - https://en.wikipedia.org/wiki/Don't_repeat_yourself
* Return early, return often - http://blog.humphd.org/vocamus-1421/
* Global Variables Are Bad - http://stackoverflow.com/a/10525602
* thin controllers and fat models - http://symfony.com/doc/current/best_practices/controllers.html
* SOLID (Wiki) - https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)
* The First 5 Principles of Object Oriented Design - https://scotch.io/bar-talk/s-o-l-i-d-the-first-five-principles-of-object-oriented-design 
* How to write SOLID code that doesn’t suck - https://medium.com/web-engineering-vox/how-to-write-solid-code-that-doesnt-suck-2a3416623d48#.61pr1ym4b
* YAGNI - https://blog.codinghorror.com/kiss-and-yagni/
* PHP Dos and Don’ts aka Programmers I Don’t Like - https://blog.radwell.codes/2016/11/php-dos-donts-aka-programmers-dont-like/
* Object Calisthenics - https://medium.com/web-engineering-vox/improving-code-quality-with-object-calisthenics-aa4ad67a61f1#.ggel1wt46
* Composition over inheritance - https://www.thoughtworks.com/pt/insights/blog/composition-vs-inheritance-how-choose


### Unit testing

### Documentation

### Errors and exception handling

### Text translations

### Version control

### Security

### Continuous integration (CI) and Continuous Delivery (CD)


### Summary