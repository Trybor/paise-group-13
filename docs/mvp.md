# MVP – KeyDuel

## Im MVP (Must)

- **US-01–03 Einzelspieler:** sofort tippen, Live-Feedback, Ergebnis mit WPM/CPM/Genauigkeit
- **US-04–05 Multiplayer:** Lobby per Link, gemeinsamer Start, gleiche Wörter, Rangliste, weitere Runden

Feste Vereinfachungen: Es gelten nur die Standardeinstellungen (Englisch, leichte Wörter, 30 s). Verlässt der Host die Lobby, endet sie.

## Danach (Should, in dieser Reihenfolge)

1. Testdauer wählen (US-06)
2. Live-Fortschritt der anderen (US-08)
3. Sprache und Schwierigkeit (US-07)
4. Personen entfernen (US-09)
5. Gesamtwertung über mehrere Runden (US-10)

## Nicht geplant

| Funktion | Warum nicht |
|---|---|
| Accounts, globale Bestenliste | Widerspricht dem Prinzip „ohne Anmeldung“ |
| Smartphone | Zum Üben braucht man eine echte Tastatur |
| Chat | Die Zielgruppe nutzt ohnehin Discord o. Ä. |
| Matchmaking mit Fremden | Moderation und Missbrauch sind im Semester nicht machbar |

## Demo (Abnahme)

1. Anna öffnet die Seite, tippt sofort los und sieht ihr Ergebnis.
2. Anna erstellt eine Lobby und schickt Ben den Link.
3. Ben öffnet den Link auf einem anderen Gerät und gibt seinen Namen ein. Anna sieht ihn in der Lobby.
4. Anna startet die Runde. Beide tippen dieselben Wörter und sehen danach die Rangliste.
5. Anna startet eine zweite Runde.

Das MVP ist fertig, wenn diese Demo in Chrome, Firefox und Edge funktioniert und alle Must-Kriterien erfüllt sind.

## Größtes Risiko

Für die Echtzeit-Kommunikation im Multiplayer fehlt uns die Erfahrung. Deshalb bauen wir sie früh als Prototyp. Auch falls der Multiplayer sich verzögert, ist der Einzelspieler für sich schon ein vollständiger Anwendungsfall.
