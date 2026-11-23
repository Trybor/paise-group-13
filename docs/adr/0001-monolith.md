# ADR 0001: Monolith statt Backend-as-a-Service oder Peer-to-Peer

- **Status:** Akzeptiert
- **Datum:** 2026-11-23

## Kontext

KeyDuel soll ohne Hürde sofort nutzbar sein: Man öffnet die Seite und tippt los. Für den Multiplayer reicht ein Link und ein Name, eine Registrierung gibt es nicht. Die Architektur soll dieses Prinzip unterstützen.

Für die Architektur besonders wichtig sind:

- Der Einzelspieler muss Zeichen innerhalb von 50 ms markieren (NFR-01). Das geht nur lokal im Browser.
- Im Multiplayer müssen Beitritte und der Rundenstart innerhalb von 2 s bei allen ankommen (NFR-02). Dafür braucht es einen Echtzeit-Kanal.
- Es gelten Spielregeln, die nicht umgangen werden dürfen: Nur der Host startet, maximal 8 Personen, alle bekommen dieselben Wörter (US-04, US-05, FR-06).
- Lobbys werden nach 30 min Inaktivität gelöscht, außer dem Namen wird nichts gespeichert (NFR-04).
- Das Hosting muss im Gratis-Tarif laufen (NFR-06).


## Entscheidung

Wir bauen einen **Monolithen**: einen einzigen Node.js-Server (TypeScript, Socket.IO), der über WebSockets die Lobbys und Runden verwaltet.

Das Frontend (React-Single-Page-App) wird als statische Dateien **getrennt vom Server** ausgeliefert (siehe [hosting.md](../hosting.md)). Der Einzelspieler läuft vollständig im Browser und braucht den Server gar nicht. Er funktioniert also auch dann sofort, wenn der Server gerade nicht erreichbar ist oder erst startet.

Innen ist der Server in Schichten aufgeteilt: Transport (Socket-Events empfangen und prüfen) → Logik (Lobby- und Rundenverwaltung) → Daten (Lobby-Speicher, siehe [ADR 0002](0002-keine-datenbank.md)).

## Betrachtete Alternativen

### Backend-as-a-Service (z. B. Firebase, Supabase) mit statischem Frontend

Es gibt keinen eigenen Server. Die Clients schreiben den Lobby-Zustand direkt in einen gehosteten Echtzeit-Dienst.

- ➕ Es gibt keinen Server zu betreiben, Echtzeit und Reconnect übernimmt der Dienst.
- ➕ Es gibt keinen Kaltstart, die Seite ist auch im Gratis-Tarif sofort da.
- ➖ Die Spielregeln (Host, max. 8, gleiche Wörter) müssten in den Security Rules des Anbieters stehen. Ohne sie könnte ein manipulierter Browser die Regeln umgehen. Die Rules sind eine eigene Sprache und schwer zu testen.
- ➖ Lobby-Daten liegen dauerhaft in einer Datenbank. Für NFR-04 bräuchte es zusätzlich einen Cleanup-Job.
- ➖ Zwei Dienste (Static Host + BaaS) und eine starke Bindung an die API eines Anbieters.

**Verworfen**, weil die Nachteile in der Architektur selbst stecken. Der Vorteil (kein Kaltstart) lässt sich dagegen beim Hosting lösen.

### Peer-to-Peer (WebRTC, der Browser des Hosts hält den Zustand)

- ➕ Kaum Serverlast.
- ➖ Für den Verbindungsaufbau braucht es trotzdem einen Signaling-Server.
- ➖ Hinter manchen Firewalls und NATs klappt die Verbindung nur über einen TURN-Server, der meist kostet (NFR-06).
- ➖ Am schwierigsten zu bauen und zu debuggen. Das widerspricht unserem größten Risiko.

**Verworfen.**

### Microservices

Getrennte Dienste für Lobbys, Runden und Wörter. Bei maximal 8 Personen pro Lobby gibt es nichts, was einzeln skaliert werden müsste. **Verworfen** wegen unnötiger Komplexität.

## Konsequenzen

**Positiv**

- Der Server setzt die Spielregeln durch. Die Clients können sie nicht umgehen.
- Eine Server-Anwendung und eine Sprache (TypeScript) für Client und Server. Formeln und Nachrichtentypen werden geteilt.
- Der Einzelspieler ist unabhängig vom Server sofort nutzbar. Das sichert unseren Kern „sofort spielen“ ab.
- Die Logik ist mit normalen Unit-Tests prüfbar (NFR-05).
- Echtzeit ist das einzige neue Thema für das Team.

**Negativ**

- Bei Gratis-Hostern mit Sleep-Modus dauert der erste Multiplayer-Aufruf nach einer Pause länger. Der Einzelspieler ist davon nicht betroffen, weil das Frontend getrennt gehostet wird. → Details in [hosting.md](../hosting.md).
- Zwei Deployments (Frontend und Server) und eine CORS-Konfiguration, damit das Frontend den Server erreichen darf.
- Es läuft nur eine Server-Instanz. Fällt sie aus, sind alle Lobbys weg. Für unsere Größenordnung nehmen wir das in Kauf.
- Wir müssen den Server selbst deployen und aktualisieren.
