# Anforderungen – KeyDuel

## Begriffe

- **WPM** (Wörter pro Minute) = (korrekte Zeichen ÷ 5) ÷ Minuten
- **CPM** (Zeichen pro Minute) = korrekte Zeichen ÷ Minuten
- **Genauigkeit** = korrekte Anschläge ÷ alle Anschläge × 100 %
  - Später korrigierte Fehler zählen trotzdem als Fehler.
  - Backspace zählt nicht als Anschlag.
- **Lobby:** ein Raum für max. 8 Personen, erreichbar über einen Link. **Host** ist, wer die Lobby erstellt hat.
- **Standardeinstellungen:** Englisch, die 200 häufigsten Wörter, 30 Sekunden

## User Stories

**M** = Must · **S** = Should · **C** = Could

### Must

**US-01 – Sofort üben** (M)
*Als Tastatur-Käufer:in möchte ich ohne Anmeldung sofort einen Tipptest machen, damit ich ohne Hürde üben kann.*
- Die Startseite zeigt direkt einen Test ohne Login. Die Eingabe hat den Fokus.
- Der Timer startet mit dem ersten Tastendruck. Die Restzeit wird als Countdown angezeigt.
- Nach Ablauf der Zeit werden Eingaben ignoriert, und das Ergebnis erscheint.

**US-02 – Live-Feedback** (M)
*Als Nutzer:in möchte ich beim Tippen sehen, welche Zeichen richtig oder falsch sind, damit ich Fehler sofort bemerke.*
- Richtige und falsche Zeichen werden unterschiedlich markiert. Falsche Zeichen erkennt man auch ohne Farbe, z. B. an einer Unterstreichung.
- Mit Backspace lassen sich Fehler im aktuellen Wort korrigieren. Die Leertaste springt zum nächsten Wort.

**US-03 – Ergebnis sehen** (M)
*Als Tastatur-Käufer:in möchte ich nach dem Test meine Werte sehen, damit ich meinen Fortschritt messen kann.*
- Angezeigt werden WPM, CPM, Genauigkeit und die Anzahl richtiger und falscher Zeichen.
- Die Werte entsprechen den Formeln oben. Das wird mit den Testfällen in NFR-05 geprüft.

**US-04 – Lobby per Link** (M)
*Als Gamer möchte ich eine Lobby erstellen und Freunde per Link einladen, damit wir ohne Account gegeneinander spielen können.*
- „Lobby erstellen“ und die Eingabe eines Namens (1–20 Zeichen) erzeugen einen Link, den man kopieren kann.
- Wer den Link öffnet, tritt nach Eingabe eines Namens bei. Alle sehen die Mitgliederliste.
- Ist die Lobby voll (8 Personen) oder existiert sie nicht, erscheint eine Fehlermeldung.

**US-05 – Gemeinsames Rennen** (M)
*Als Gamer möchte ich mit allen gleichzeitig dieselben Wörter tippen und danach die Ergebnisse vergleichen, damit wir sehen, wer am schnellsten ist.*
- Nur der Host kann starten, und nur ab 2 Personen. Danach sehen alle einen 3-2-1-Countdown.
- Alle bekommen dieselben Wörter, und der Timer startet für alle gleichzeitig.
- Nach der Runde sehen alle eine Rangliste, sortiert nach WPM.
- Der Host kann danach eine weitere Runde mit neuen Wörtern starten.
- Verliert jemand die Verbindung, läuft die Runde für die anderen weiter. Verlässt der Host die Lobby, endet sie für alle.

### Should / Could

| ID | Story | Prio |
|---|---|---|
| US-06 | Als Nutzer:in möchte ich die Testdauer wählen (15/30/60/120 s), damit ich kurz oder lang üben kann. | S |
| US-07 | Als Nutzer:in möchte ich Sprache (DE/EN) und Schwierigkeit wählen, damit der Test zu mir passt. | S |
| US-08 | Als Gamer möchte ich während der Runde den Fortschritt der anderen sehen, damit echtes Rennen-Gefühl entsteht. | S |
| US-09 | Als Host möchte ich Personen aus der Lobby entfernen, damit ich Störenfriede ausschließen kann. | S |
| US-10 | Als Gamer möchte ich nach mehreren Runden eine Gesamtwertung sehen, damit wir ein „Best of“ spielen können. | S |
| US-11 | Als Nutzer:in möchte ich ein Diagramm meines Tempoverlaufs sehen, damit ich erkenne, wann ich langsamer werde. | C |

## Funktionale Anforderungen

| ID | Anforderung | Prio | Story |
|---|---|---|---|
| FR-01 | Ein Test ist ohne Anmeldung direkt auf der Startseite startbar. | M | US-01 |
| FR-02 | Pro Test wird eine zufällige Wortfolge erzeugt. | M | US-01 |
| FR-03 | Jedes getippte Zeichen wird sofort als richtig oder falsch markiert. | M | US-02 |
| FR-04 | Nach dem Test werden WPM, CPM und Genauigkeit berechnet und angezeigt. | M | US-03 |
| FR-05 | Lobbys werden mit zufälligem Code und teilbarem Link erstellt. Der Beitritt erfordert nur einen Namen. | M | US-04 |
| FR-06 | Alle in einer Runde bekommen dieselben Wörter und einen gemeinsamen Start. | M | US-05 |
| FR-07 | Nach jeder Runde wird allen eine Rangliste angezeigt. | M | US-05 |
| FR-08 | Testdauer, Sprache und Schwierigkeit sind einstellbar. | S | US-06, US-07 |
| FR-09 | Fortschritt der anderen live anzeigen; der Host kann Personen entfernen. | S | US-08, US-09 |
| FR-10 | Eine Gesamtstatistik über alle Runden wird angezeigt. | S | US-10 |

## Nichtfunktionale Anforderungen

| ID | Anforderung | Prüfung | Prio |
|---|---|---|---|
| NFR-01 | **Reaktionszeit:** Ein getipptes Zeichen ist innerhalb von 50 ms markiert. | Messung mit Browser-Dev-Tools | M |
| NFR-02 | **Multiplayer:** Beitritte und Rundenstart kommen bei allen innerhalb von 2 s an. | Test mit 2 Geräten in verschiedenen Netzen | M |
| NFR-03 | **Browser:** Läuft auf aktuellem Chrome, Firefox und Edge (Desktop). Mobile Geräte werden nicht unterstützt. | Demo-Szenario in allen 3 Browsern | M |
| NFR-04 | **Datenschutz:** Außer dem Anzeigenamen werden keine Daten gespeichert. Lobbys werden nach 30 min Inaktivität gelöscht. | Code-Review | M |
| NFR-05 | **Korrektheit:** Die Berechnungen sind durch Unit-Tests abgedeckt, mindestens mit diesen Fällen (Ziel `the cat sat`, 60 s): <br>• `the cat sat` → 11 korrekte Zeichen, 100 % <br>• `thx⌫e cat sat` → 11 korrekte Zeichen, 91,7 % | Automatisierte Tests | M |
| NFR-06 | **Kosten:** Keine kostenpflichtigen APIs, Hosting im Gratis-Tarif. | Kostenübersicht | M |

## Priorisierung

- **Must:** US-01 bis US-05. Ohne diese gibt es keinen funktionierenden Tipptest und keinen Multiplayer, also das, was uns von anderen Tipptrainern unterscheidet.
- **Should:** US-06 bis US-10. Diese Stories machen das Produkt besser, es ist aber auch ohne sie nutzbar.
- **Could:** US-11.

Prüffrage für Must: *„Ist das Produkt ohne diese Story noch nutzbar?“* Nur bei „Nein“ ist die Story Must.

## Geklärte Unklarheiten aus der Ideensammlung

| Unklarheit | Entscheidung |
|---|---|
| „Countdown“: Startcountdown oder Restzeit? | Die Restzeit wird immer angezeigt. Der 3-2-1-Countdown gibt es nur im Multiplayer. |
| „Kein Login“, aber „Leute kicken“: Wie erkennt man den Host? | Der Host wird über die Browser-Sitzung erkannt, nicht über einen Account. |
| „Schwierigkeit“ und „visuelle Statistik“ waren nicht definiert. | Konkretisiert in US-07 und US-11. |
| „Leute kicken“ stand doppelt. | Zusammengeführt in US-09. |
| Wer stellt im Multiplayer Zeit und Sprache ein? | Im MVP gelten die Standardeinstellungen für alle. |
| Was passiert, wenn der Host geht? | Die Lobby endet (US-05). |
