# ADR 0002: Keine Datenbank, Lobbys im Arbeitsspeicher

- **Status:** Akzeptiert
- **Datum:** 2026-11-23

## Kontext

KeyDuel funktioniert ohne Accounts (US-01, US-04). Es gibt deshalb keine Nutzerprofile, keine Historie und keine globale Bestenliste (mvp.md, „Nicht geplant“).

Die einzigen Daten, die der Server kennt, sind laufende Lobbys: Code, Namen der Teilnehmenden, aktuelle Runde und deren Ergebnisse. Laut NFR-04 wird außer dem Anzeigenamen nichts gespeichert, und Lobbys werden nach 30 min Inaktivität gelöscht.

Der Server läuft als einzelne Instanz ([ADR 0001](0001-monolith.md)).

## Entscheidung

Wir verwenden **keine Datenbank**. Alle Lobbys liegen im Arbeitsspeicher des Servers in einer Map (`Lobby-Code → Lobby`).

- Ein Timer prüft regelmäßig, welche Lobbys seit 30 min inaktiv sind, und entfernt sie.
- Verlässt der Host die Lobby, wird sie sofort entfernt (US-05).
- Der Einzelspieler speichert nichts auf dem Server.

## Betrachtete Alternativen

### Relationale Datenbank (z. B. PostgreSQL, SQLite)

- ➕ Daten überleben einen Neustart des Servers.
- ➖ Wir müssten Daten speichern, die wir laut NFR-04 gar nicht dauerhaft halten wollen, und sie danach wieder löschen.
- ➖ Ein weiterer Dienst, ein Schema, Migrationen. Im Gratis-Tarif ist eine gehostete Datenbank oft begrenzt.

**Verworfen.**

### Key-Value-Speicher (z. B. Redis)

- ➕ Daten können automatisch ablaufen (TTL), das passt zu NFR-04.
- ➕ Würde mehrere Server-Instanzen ermöglichen.
- ➖ Mehrere Instanzen brauchen wir nicht. Ein weiterer Dienst wäre nur Aufwand.

**Verworfen.** Wäre die richtige Wahl, falls später mehrere Instanzen nötig werden.

## Konsequenzen

**Positiv**

- NFR-04 ist fast automatisch erfüllt: Was nicht gespeichert wird, muss auch nicht gelöscht werden.
- Kein zusätzlicher Dienst, keine Kosten, keine Zugangsdaten.
- Lesen und Schreiben im Speicher ist schnell, das hilft bei NFR-02.
- Wenig Code und leicht zu testen.

**Negativ**

- Bei einem Neustart oder Absturz des Servers (auch beim Einschlafen eines Gratis-Hosters) sind alle offenen Lobbys weg. Die Spielenden müssen eine neue Lobby erstellen.
- Der Server kann nicht auf mehrere Instanzen verteilt werden.
- Funktioniert der Cleanup-Timer nicht, wächst der Speicher. → Der Cleanup bekommt einen eigenen Unit-Test.
- Funktionen wie eine Historie oder Bestenliste würden diese Entscheidung neu aufrollen. Sie sind bewusst nicht geplant.
