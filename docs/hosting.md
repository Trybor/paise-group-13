# Hosting – KeyDuel

> Stand der Recherche: 23.11.2026. Preise und Gratis-Tarife ändern sich oft, vor dem Deployment erneut prüfen.

## Anforderungen an das Hosting

KeyDuel besteht aus zwei Teilen ([ADR 0001](adr/0001-monolith.md)):

| Teil | Was er braucht |
|---|---|
| **Frontend** (React-SPA) | Statische Dateien ausliefern. Rewrite auf `index.html`, damit Links wie `/lobby/AB12CD` funktionieren. |
| **Server** (Node.js, Socket.IO) | Dauerhaft laufender Prozess mit **WebSockets** und Lobbys im Arbeitsspeicher ([ADR 0002](adr/0002-keine-datenbank.md)). Eine einzige Instanz. |

Dazu kommen diese Anforderungen:
- **NFR-06:** gratis, keine kostenpflichtigen Dienste
- **NFR-02:** Beitritte und Rundenstart kommen innerhalb von 2 s an
- **NFR-04:** Datenschutz. Ein Serverstandort in der EU ist wünschenswert.
- **Kernidee „sofort spielen“:** Der Einzelspieler soll ohne Wartezeit starten.

## Vergleich: Server

| Anbieter | Kosten | Kreditkarte | Sleep / Kaltstart | WebSockets | EU-Region | Bewertung |
|---|---|---|---|---|---|---|
| **Render** (Free Web Service) | 0 €, 750 h/Monat pro Workspace (reicht für Dauerbetrieb einer Instanz) | laut Doku nicht nötig | nach 15 min ohne Traffic, Start ca. 1 min | ✅ | Frankfurt (im Free-Tarif noch zu bestätigen) | ✅ **gewählt** |
| **Railway** (Free Plan) | 0 €, 1 $ Guthaben/Monat | nicht nötig | nach 5–10 min ohne Traffic | ✅ | – | ⚠️ unklar, ob 1 $ für den Betrieb reicht |
| **Fly.io** | 5 $ Guthaben/Monat **nur mit Kreditkarte**. Kleinste Maschine 2,19 $/Monat. | ja | konfigurierbar | ✅ | ✅ | ⚠️ Plan B (Kreditkarte nötig) |
| **Koyeb** | keine kostenlose Server-Instanz mehr auf der Preisseite | – | – | – | Frankfurt | ❌ verletzt NFR-06 |
| **Vercel** (Functions) | Gratis-Tarif vorhanden | – | – | ✅ seit Okt. 2026 | ✅ | ❌ siehe unten |
| **Eigener VPS** | wenige €/Monat *(Preis nicht geprüft)* | ja | kein Sleep | ✅ | ✅ | ⚠️ kostet, Betrieb selbst (Updates, TLS) |

### Warum nicht Vercel?

Vercel unterstützt inzwischen WebSockets in seinen Functions, auch mit Socket.IO. Für unsere Architektur passt es trotzdem nicht:
- **Verbindungen werden nach einer maximalen Laufzeit getrennt.** Auf den Pro- und Enterprise-Tarifen sind das standardmäßig 300 s, für den Gratis-Tarif ist kein Wert angegeben. Eine Lobby-Sitzung dauert oft länger.
- **Kein gemeinsamer Speicher.** Clients derselben Lobby können auf verschiedenen Instanzen landen. Der Lobby-Zustand müsste deshalb in einem externen Speicher wie Redis liegen. Das widerspricht ADR 0002.

## Vergleich: Frontend

Statische Seiten sind bei fast allen Anbietern gratis und schlafen nicht ein.

| Anbieter | Bewertung |
|---|---|
| **Render Static Site** | ✅ **gewählt**: gleicher Anbieter wie der Server, ein Dashboard, Rewrite-Regeln für Single-Page-Apps eingebaut |
| Vercel, Netlify | ✅ ebenfalls geeignet, aber ein zusätzliches Konto |
| GitHub Pages | ⚠️ Routen wie `/lobby/:code` brauchen Umwege (404-Trick oder Hash-Routing) |

## Entscheidung

**Beide Teile laufen bei Render:**

```
Browser ──HTTPS──▶ Render Static Site   (React-Frontend, kein Sleep)
   └─────WebSocket──▶ Render Web Service (Node.js + Socket.IO, Free, Frankfurt)
```

**Begründung:**
- Kostenlos und ohne Kreditkarte (NFR-06).
- Das Frontend schläft nie ein. Der **Einzelspieler ist also immer sofort nutzbar**, auch wenn der Server gerade schläft.
- Ein Anbieter, ein Konto, automatisches Deployment bei jedem Push auf GitHub.
- Die Region Frankfurt ist möglich (Datenschutz, kurze Latenz für NFR-02).

## Umgang mit dem Kaltstart

Der Gratis-Server schläft nach 15 min ohne Traffic ein. Das Aufwachen dauert ca. 1 min.

| Situation | Lösung |
|---|---|
| Einzelspieler | Nicht betroffen, er braucht den Server nicht. |
| Erster Multiplayer-Aufruf | Das Frontend ruft beim Laden der Seite `GET /health` auf. Der Server wacht auf, während die Person noch tippt oder ihren Namen eingibt. |
| Verlorene Lobbys beim Einschlafen | Unkritisch: Der Server schläft nur ein, wenn 15 min lang nichts passiert ist. Dann gibt es auch keine aktive Lobby mehr. |
| Demo bei der Abnahme | Die Seite einige Minuten vorher öffnen. |

## Kosten

| Posten | Kosten pro Monat |
|---|---|
| Render Static Site | 0 € |
| Render Free Web Service | 0 € |
| **Gesamt** | **0 €** |

**Plan B**, falls der Kaltstart oder die Limits stören: Fly.io (kleinste Maschine ca. 2,19 $/Monat, mit Kreditkarte) oder ein kleiner VPS. Die Architektur bleibt in beiden Fällen gleich, nur der Server zieht um.

## Risiken

- Gratis-Tarife können sich ändern oder eingestellt werden. Bei Koyeb ist das 2026 bereits passiert. Weil der Server ein normaler Node-Prozess ist, ist ein Umzug zu einem anderen Anbieter mit wenig Aufwand möglich.
- Render darf Gratis-Dienste jederzeit neu starten. Offene Lobbys gehen dann verloren (siehe ADR 0002).

## Quellen

- [Render – Free instances](https://render.com/docs/free)
- [Render – Regions](https://render.com/docs/regions)
- [Railway – Pricing](https://railway.com/pricing)
- [Fly.io – Pricing](https://docs.fly.io/about/pricing)
- [Koyeb – Pricing](https://www.koyeb.com/pricing)
- [Vercel – WebSocket-Support in Functions](https://vercel.com/kb/guide/do-vercel-serverless-functions-support-websocket-connections)
- [Vercel Changelog – WebSocket support generally available](https://vercel.com/changelog/websocket-support-is-now-generally-available)
