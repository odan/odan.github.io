---
title: Geheime Informationen ausserhalb der Versionskontrolle
layout: post
comments: true
published: false
description: 
keywords: 
---

Geheime Informationen wie z.B. Passwörter, API-Keys, Zertifikate etc. sollten nicht in der Versionskontrolle (Git, SVN) gespeichert werden. Gerade bei OpenSource-Projekten muss man besonders darauf zu achten. Mit Suchmaschinen ist es relativ leicht nach Passwörtern zu suchen und diese Daten zu entwenden.

Man darf zwar keine geheime Informationen versionieren, gleichzeitig muss man alle Änderungen zurückverfolgen und im Entwicklerteam teilen können. Ausserdem müssen diese Informationen in den automatischen Deployment-Prozess integriert werden.

Die Verlagerung dieser Informationen in ein anderes System mündet in einer “Schatten-Versionsverwaltung” bzw. einem Konfigurations-Repository. Diese Information werden also trotzdem versioniert, nur eben an einem anderen Ort. Die Komplexität steigt. Schlussendlich muss auch der Zugang zum Konfigurations-Repository irgendwie gesichert werden. Doch wohin mit diesen Zugangsdaten? Leider beißt sich hier die Katze in den Schwanz. Schlussendlich löst man damit das Problem leider nicht.

Versioniert man in einem firmeninternen (nicht öffentlich) Bereich (LAN) ist der komplette Quellcode, inkl. aller geheimen Passwörter, per se geheim. Nicht eine einzige Zeile Quellcode darf davon in die Öffentlichkeit. Für firmeninterne Projekte würde der Aufwand den Nutzen übersteigen. Die separate Versionierung ergibt für firmeninterne Projekte also weniger Sinn.

Anders ist es bei OpenSource Projekten. Der Quellcode ist öffentlich, die Zugangsdaten dürfen daher nicht unter die Versionskontrolle fallen.

## Deployment Script mit Konfigurations-Builder

Ein deployment script (PHP-Datei) welches die geheimen Informationen aus dem Konfigurations-Repository (Datenbank, Git, SVN) holt und in einer Config-Datei (PHP, Yaml, XML) speichert. Nachteil: Die Zugangsdaten für das Konfigurations-Repository müssen auch “irgendwo” gespeichert und eventuell auch versioniert werden. Alternativ muss das Passwort für das Konfigurations-Repository in der Console manuell eingetippt werden.

## Umgebungsspezifische Variablen in Konfigurationsdateien

Mittels nativen PHP-Code können die serverspezifischen und sensiblen Parameter aus einem lokalen File gelesen und mit der Default-Konfiguration ergänzt werden. Diese env-Datei darf höchstens als Beispieldatei für andere Entwickler im Repo abgelegt werden (z.B. env.example.php). Jeder Entwickler erstellt seine eigene lokale Kopie und speichert sie als env.php. Die env.php Datei kann somit an die individuellen Bedürfnisse der lokalen Entwicklungsumgebung angepasst werden. Beim deployment wird die Datei env.php nicht automatisch verteilt, sondern muss/sollte manuell hochgeladen / aktualisiert werden. Da sich die Zugangsdaten selten ändern, hält sich der manuelle Aufwand in Grenzen. Die Sicherheit ist dadurch gewährleistet, dass jeder Server seine eigene und exklusive env-Datei erhält.