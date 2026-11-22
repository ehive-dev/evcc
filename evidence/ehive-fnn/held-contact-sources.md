# Zeitbasis für den Kontakt-Test am 22.11.2026

Eine konkrete FNN-Steuerbox und ein Netzbetreiber sind noch nicht bekannt
(Rückmeldung des Nutzers: nein). Deshalb wird keine hersteller- oder
netzbetreiberspezifische Relais-Haltezeit als bestätigt ausgegeben.

Der öffentlich zugängliche VDE-FNN-Hinweis vom Oktober 2024 beschreibt in
Abschnitt 6.1 die Zuordnung W4 zu SteuVE und S1/S2/W3 zu den Einspeisestufen.
In diesem Dokument wurde keine feste Kontakt-Schaltperiode gefunden. Das ist
kein Beweis, dass in anderen Spezifikationen oder konkreten Parametrierungen
keine zusätzlichen Zeitvorgaben bestehen.

Quelle:
https://www.bundesnetzagentur.de/DE/Beschlusskammern/1_GZ/BK6-GZ/2022/BK6-22-300/Mitteilung/Mitteilung_2/VDE%20FNN%20Empfehlung%20zu%20Tenorziffer%202a.pdf?__blob=publicationFile&v=1

Der Hersteller Swistec beschreibt die Umsetzung von Schalthandlungen über
Fahrpläne, unter anderem für Schaltprogramme, Direktbefehle und Wischerbefehle.
Daraus lässt sich keine universelle Dauer für unseren unbekannten Anschluss
ableiten.

Quelle:
https://swistec.de/swistec-glossar/

Das vollständige Lastenheft und die Parametrierungsunterlagen werden von VDE
als Mitgliederdownload beziehungsweise im Shop angeboten. Es wurden keine
Unterlagen gekauft und kein Mitgliedskonto verwendet.

Quelle:
https://www.vde.com/de/fnn/aktuelles/neues-erweitertes-lastenheft-steuerbox-1-5-

## Getroffene Testannahmen

- Physische Kontaktzustände bleiben für 30 oder 60 Sekunden erhalten. Diese
  Werte sind gewählte Testszenarien, keine als FNN-Vorgabe belegten Mindestzeiten.
- Der tatsächliche ioManager erfasst alle 1000 ms, mit fünf gemeinsamen
  GPIO-Abfragen pro Poll und 500 Mikrosekunden Abstand.
- Das getestete evcc-Template verwendet 5 s Regelintervall, 1 s HTTP-Cache und
  2 s HTTP-Timeout, unverändert gegenüber dem Cache-Kandidaten.
- Es wird keine künstliche HTTP-Verzögerung in den physischen Haltezeittest
  eingebaut. Die realen Loopback-Antwortzeiten werden gemessen.
- Ein separater Test erhält die künstlichen 1,2 s HTTP-Body-Verzögerung bei,
  wechselt das Kontaktpaar aber nur einmal und hält danach den Endzustand.
  Damit wird die Erholung gemessen, statt bei jeder HTTP-Abfrage neue
  Kontaktzustände zu erzeugen.

Die 1,2 s des vorherigen Grenzfalltests sind eine Verzögerung der lokalen
HTTP-Antwort, keine Schalt- oder Übertragungszeit einer FNN-Steuerbox. Die
FNN-Kontaktdauer und die HTTP-Antwortdauer werden getrennt bewertet.
