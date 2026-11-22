# Gehaltene FNN-Kontaktzustände auf dem Entwicklungsgerät geprüft, 22.11.2026

**Ergebnis:** Der Cache-Kandidat bestand den vierminütigen physischen Test mit
30/60 Sekunden gehaltenen Kontaktzuständen. In den beobachteten begrenzten
Übergängen trat keine ungewollte Freigabe auf. Beim separaten künstlichen
Verzögerungsfall mit einem einmaligen Wechsel korrigierte sich der Regler im
nächsten Durchlauf nach 6,205 Sekunden und blieb danach bei 30 %.

## Zeitbasis und Grenzen

Die konkrete FNN-Steuerbox und der Netzbetreiber sind noch nicht bekannt.
30 und 60 Sekunden sind deshalb ausdrücklich gewählte Testhaltezeiten, keine
bestätigten FNN-Mindestzeiten oder ein nachgewiesener realer Netzbetreiberfahrplan.
Quellen und Abgrenzung stehen in `held-contact-sources.md`.

Quelle zur FNN-Kontaktzuordnung, Abschnitt 6.1:
https://www.bundesnetzagentur.de/DE/Beschlusskammern/1_GZ/BK6-GZ/2022/BK6-22-300/Mitteilung/Mitteilung_2/VDE%20FNN%20Empfehlung%20zu%20Tenorziffer%202a.pdf?__blob=publicationFile&v=1

## Aufbau und Beobachtung

Zwei Shelly-Plus-1-Kontakte steuerten die echten DI1/DI2-Leitungen. Zwei
isolierte evcc-Instanzen werteten denselben Aufbau gleichzeitig als W4/W3
beziehungsweise S1/S2 aus. Die realen Verbraucher waren nicht eingebunden.

Verwendet wurde der unveränderte Cache-Kandidat `0.317.1-ehive.4`, Quellcommit
`d190f63`, SHA-256
`3e52cd99792d6eccc3bf364d3d4454173ffaae0be8618d218139f1f764503916`.
Der eigentliche FNN-Regler blieb unverändert; nur die Vorlage ist neu.

Tatsächliche Einstellungen: ioManager 1000 ms Pollintervall, fünf gemeinsame
GPIO-Abfragen mit 500 Mikrosekunden Abstand; evcc 5 s FNN-Regelintervall,
1 s HTTP-Cache, 2 s Timeout. Keine künstliche HTTP-Verzögerung im Hardwaretest.

| DI1/DI2 | Haltezeit | W4/W3-Erwartung | S1/S2-Erwartung |
|---|---:|---|---:|
| aus/aus | 30 s | frei | 100 % |
| ein/aus | 60 s | Verbrauch 4200 W | 60 % |
| ein/ein | 30 s | Verbrauch 4200 W, Einspeisung 0 % | 30 % |
| aus/ein | 60 s | Einspeisung 0 % | 30 % |
| ein/aus | 30 s | Verbrauch 4200 W | 60 % |
| aus/aus | 30 s | frei | 100 % |

Beim Wechsel zwischen zwei begrenzten Stufen wurden neue aktive Kontakte
zuerst gesetzt und alte anschließend gelöst. Dadurch erzeugt die Fixture keine
beabsichtigte Zwischenfreigabe. Die tatsächliche Umschaltreihenfolge einer
späteren FNN-Steuerbox ist damit nicht bestätigt.

Die HEMS-Anzeigen wurden fortlaufend ungefähr alle 0,1 s zuzüglich der
HTTP-Laufzeiten gelesen. Nach dem erstmaligen Erreichen des Sollzustands musste
er bis zum nächsten physischen Wechsel stabil bleiben. Übergänge mussten in
8 s konvergieren. Diese 8 s sind eine Testgrenze für unsere Abfragekette, keine
regulatorische Reaktionsfrist. Zusätzlich wurde die erfasste Änderungsfolge auf
100-%-Freigaben zwischen begrenzten S1/S2-Stufen geprüft.

## Messwerte

- 6 physische Haltephasen, 12 Instanz-/Phasenprüfungen: bestanden.
- Haltephasen insgesamt 240.539 s.
- Höchste beobachtete Zeit bis zum passenden HEMS-Zustand: 4.567 s,
  einschließlich der anfänglichen Statusübernahme.
- 1726 echte ioManager-Snapshot-Abrufe, längster vollständiger Abruf
  65.895 ms, 0 API-Fehler.
- Keine beobachtete unerwartete Freigabe bei den begrenzten S1/S2-Übergängen.

## Einmaliger langsamer Übergang und Erholung

Der separate ARM64-Test injiziert einmal den bereits bekannten gemischten
Abfragezustand mit 1,2 s verzögertem HTTP-Inhalt. Anschließend liefert er nur noch
dasselbe endgültige Kontaktpaar S1=false/S2=true. Bei einem Regeltakt von 5 s:

- Erste fehlerhafte Auswertung: 100 %.
- Nächster Durchlauf: nach 6,205 s korrekt 30 %.
- Weitere Durchläufe: nach 11,208 s und 16,208 s weiterhin 30 %.

Damit ist die Erholung bei gehaltenem Endzustand gemessen. Der vorherige
Grenzfall wird dadurch nicht widerlegt: Er verlangte nicht nur längere
Kontakt-Haltezeiten, sondern verzögerte gezielt den lokalen HTTP-Antwortinhalt.
Der jetzige Versuch zeigt die Selbstkorrektur nach dem einmaligen Übergang;
er belegt keine zulässige Unterbrechung einer Begrenzung.

## Nachweise und Gerätestand

- Full hardware traces retained locally; compact summary published here.
- `held-contact-summary.json`
- `held-recovery.log`
The tested template is published alongside this report as `cache-candidate.yaml`.

Beide Testinstanzen wurden beendet. Beide Shellys sind wieder OFF. Haupt-evcc,
Haupt-ioManager und Hauptkonfiguration blieben unverändert; kein Dienstneustart
und keine neue Geräteinstallation in diesem Lauf. Nur zusätzliche Testdateien
wurden übertragen. Der Testlauf selbst änderte den PR-Code nicht. Dieser Bericht wird als
Nachweis zum anschließenden Review-Feedback veröffentlicht.
