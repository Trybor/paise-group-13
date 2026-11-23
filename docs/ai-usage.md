# AI-Einsatz und Reflexion

Werkzeug: Claude Code (Claude Opus 5.5) in VS Code.

## 1. Anforderungen aus der Idee generieren

**Prompt (gekürzt):**
> du hast in der ideen md alles was ich zu dem projekt recherchiert habe. nun kriegst du von mir die definition und req aus der aufgabe *(+ Aufgabenbeschreibung)*

**Ergebnis:** Aus unseren Notizen in [Ideen.md](../Ideen.md) entstanden User Stories, Anforderungen, eine Priorisierung und ein MVP-Vorschlag. Die AI hat außerdem Unklarheiten in unserer Idee gefunden, z. B. dass „Countdown“ mehrdeutig war und dass „kein Login“ mit „Leute kicken“ kollidiert.

**Bewertung:**
- ✅ Struktur und die gefundenen Unklarheiten waren hilfreich.
- ❌ **Viel zu umfangreich:** Die erste Fassung hatte 16 Stories und 25 Anforderungen. Wir haben sie auf 11 Stories und 10 Anforderungen gekürzt. Die Spezifikation soll lesbar bleiben.
- ❌ **Zu viele Must-Haves:** „Testdauer wählen“ war Must, obwohl das Produkt auch mit einer festen Dauer funktioniert. Jetzt ist es Should.
- ❌ **Rechenfehler:** Ein generierter Testfall hatte eine falsche Genauigkeit. Das haben wir erst beim manuellen Nachrechnen gemerkt.

## 2. Anforderungen auf Widersprüche prüfen

Ein **separater AI-Agent**, der die Begründungen der ersten Sitzung nicht kannte, hat die Dokumente kritisch geprüft.

**Prompt (gekürzt, auf Englisch):**
> You are a critical requirements-engineering reviewer. Find contradictions between documents, ambiguous or untestable wording, missing edge cases, wrong priorities and unmeasurable NFRs. Do not invent issues.

**Ergebnis:** 15 Befunde. Die wichtigsten, die wir übernommen haben:
- Das Verhalten beim Weggang des Hosts war nicht definiert. Jetzt gilt: Die Lobby endet.
- Die Demo verlangte eine zweite Runde, die aber nur als Should geplant war. Sie ist jetzt Teil von US-05.
- Edge stand in den Anforderungen, fehlte aber in der Abnahme.
- Es gab keine Testfälle für die Berechnungen. Sie stehen jetzt in NFR-05.

**Nicht übernommen:** Sehr detaillierte Regeln für Sonderfälle beim Tippen, z. B. eine Leertaste mitten im Wort. Sie wären korrekt, blähen die Spezifikation aber auf. Wir klären sie bei der Implementierung über die Unit-Tests.

**Bewertung:** Das Review fand echte Widersprüche zwischen den Dokumenten. Den Rechenfehler hat es aber nicht gefunden, und unsere Grundannahmen hat es nicht infrage gestellt. Ein Beispiel ist die Frage, ob Multiplayer überhaupt Must sein soll.

## Fazit Phase 1

AI ist stark darin, eine Struktur zu schaffen und Lücken zu finden. Sie neigt aber dazu, zu viel zu spezifizieren, und setzt Zahlen, die plausibel klingen. Was ins Produkt kommt, haben wir entschieden.

---

# Phase 2 – Architektur

## 3. Architekturvorschläge erzeugen lassen

**Prompt (gekürzt):**
> Gib mir das, was die Aufgabe AI erlaubt mir zu liefern, und sage mir, was ich zu tun habe. *(+ Aufgabenbeschreibung Phase 2)*

**Ergebnis:** Die AI hat unsere Anforderungen auf ihre Folgen für die Architektur abgeklopft und drei Varianten vorgeschlagen: Monolith, Backend-as-a-Service (z. B. Firebase) und Peer-to-Peer. Microservices hat sie gleich als Overengineering ausgeschlossen. Dazu kamen Technologie-Optionen, erste Entwürfe für Diagramme, Datenmodell und API sowie eine Liste mit Risiken. Den Vorschlag haben wir unverändert in [ai-input/phase2-architektur-vorschlaege.md](ai-input/phase2-architektur-vorschlaege.md) abgelegt, damit man sieht, was von der AI kam.

**Bewertung:**
- ✅ Der Vergleich der Varianten war eine gute Grundlage. Wir mussten nicht bei null anfangen.
- ✅ Die Risikoliste hat zwei Lücken in unseren Anforderungen aufgedeckt: Was passiert, wenn der Host die Seite neu lädt? Und was, wenn jemand während einer laufenden Runde beitritt? Beides haben wir entschieden und in [requirements.md](requirements.md) nachgetragen.
- ❌ Manche Formulierungen waren zu knapp. „Host, max. 8, gleiche Wörter“ haben wir zuerst als „8 gleiche Wörter“ gelesen.
- ❌ Die AI hatte Express als Web-Framework vorgeschlagen. Unser Server liefert aber nur WebSockets und einen Health-Check aus, dafür braucht es kein Framework. Wir haben es gestrichen.

## 4. Unsere Entscheidung hinterfragen lassen: Monolith oder BaaS?

Wir haben uns für den Monolithen entschieden und das zuerst so begründet: KeyDuel soll ohne Registrierung sofort spielbar sein, also soll die Architektur einfach sein und wenige Dienste brauchen. Das haben wir die AI prüfen lassen.

**Prompt (gekürzt):**
> Spricht etwas mehr für die anderen Vorschläge?

**Ergebnis:** Die AI hat eine Schwäche in unserer Begründung gefunden. „Ohne Login“ trennt die Varianten gar nicht, denn auch BaaS und Peer-to-Peer funktionieren ohne Anmeldung. Die eigentlichen Argumente für den Monolithen sind andere: Der Server setzt die Spielregeln durch, Daten im Arbeitsspeicher verschwinden von selbst (NFR-04), und die Logik ist normal testbar. Gleichzeitig hat sie einen echten Vorteil von BaaS genannt: Es gibt keinen Kaltstart, während ein Gratis-Server nach einer Pause erst aufwachen muss. Das widerspricht unserem Ziel „sofort spielen“.

Darauf haben wir gefragt: *„Also wäre Variante B besser?“* Die Antwort war Nein, weil der Kaltstart ein Problem des Gratis-Hosters ist und nicht der Architektur. Die Nachteile von BaaS stecken dagegen in der Architektur selbst.

**Was wir daraus gemacht haben:**
- Unser Selling Point steht im [ADR 0001](adr/0001-monolith.md) jetzt als Kontext und nicht mehr als Hauptargument.
- Wir hosten das Frontend getrennt vom Server. So ist der Einzelspieler immer sofort da, auch wenn der Server schläft. Diese Idee kam erst durch die Diskussion zustande.
- Wir haben auch Vercel für alles betrachtet. Dafür hätten wir Socket.IO durch einen externen Echtzeit-Dienst und den Arbeitsspeicher durch Redis ersetzen müssen. Damit hätten wir beide ADRs umgeworfen, also haben wir es verworfen.

**Bewertung:** Das war für uns der wertvollste Teil von Phase 2. Die AI hat nicht nur bestätigt, sondern unsere eigene Begründung als zu schwach erkannt. Nebenbei haben wir uns den Unterschied zwischen Monolith (wie wird ausgeliefert?) und Layered Architecture (wie ist der Code innen aufgebaut?) erklären lassen. Beides trifft auf uns zu.

## 5. Hosting recherchieren

Die erste Hosting-Tabelle der AI stammte aus ihrem Trainingswissen. Danach hat sie die Angaben auf den Websites der Anbieter nachgeprüft.

**Ergebnis:** Zwei Aussagen aus der ersten Tabelle waren falsch:
- ❌ **Koyeb** war als Gratis-Alternative empfohlen. Die Preisseite zeigt aber keine kostenlose Server-Instanz mehr.
- ❌ **Vercel** sollte keine WebSockets unterstützen. Seit Oktober 2026 werden sie aber unterstützt. Vercel passt trotzdem nicht zu uns, weil Verbindungen nach einer maximalen Laufzeit getrennt werden und es keinen gemeinsamen Arbeitsspeicher gibt.

**Bewertung:** Angaben zu Preisen und Gratis-Tarifen veralten schnell. Ohne die Nachprüfung hätten wir in [hosting.md](hosting.md) mit falschen Voraussetzungen argumentiert. Nicht ganz klären ließ sich, ob die Region Frankfurt bei Render auch im Gratis-Tarif wählbar ist. Das sehen wir erst beim Anlegen des Dienstes.

## 6. ADRs, Architekturdokument und Diagramme

**Ergebnis:** Die AI hat die [ADRs](adr/), [architecture.md](architecture.md) und [hosting.md](hosting.md) formuliert, auf Basis unserer Entscheidungen:
- React, Socket.IO und TypeScript, weil wir React schon kennen
- Lobby bleibt 10 s erhalten, wenn die Verbindung abbricht
- Wer während einer Runde beitritt, spielt ab der nächsten Runde mit
- Schummeln nehmen wir in Kauf, weil die Zielgruppe Freundesgruppen sind

Die C4-Diagramme haben wir selbst in draw.io gezeichnet. Die Mermaid-Entwürfe der AI dienten dabei als Vorlage. Im Kontextdiagramm haben wir die Tastatur-Tester:innen als eigene Gruppe ergänzt. Die AI hat unsere Diagramme danach gegengelesen und fehlende Beschriftungen gefunden (die Rahmen für Shared, Server und Render).

**Bewertung:**
- ✅ Beim Ausformulieren hat die AI einen Fehler in ihrem eigenen Datenmodell gefunden. Aus den ursprünglichen Zählern ließ sich die Genauigkeit nicht korrekt berechnen, weil korrigierte Fehler nicht erfasst wurden. Mit dem neuen Zähler `errorKeystrokes` stimmt der Testfall aus NFR-05 (11 von 12 Anschlägen, also 91,7 %).
- ⚠️ Die AI hat für Sonderfälle eigene Annahmen getroffen, z. B. 5 s Puffer für Ergebnisse oder dass bei gleichem WPM die Genauigkeit entscheidet. Sie hat sie offen aufgelistet, und wir haben sie so übernommen.
- ❌ Den Unterschied zwischen dem C4-Modell und Mermaid mussten wir erst nachfragen. Die AI hatte die Mermaid-Diagramme „C4“ genannt, obwohl sie nur den C4-Ebenen folgen und nicht der offiziellen Notation.

## Fazit Phase 2

Am meisten hat uns die AI geholfen, wenn wir nachgefragt und widersprochen haben, nicht beim ersten Vorschlag. Faktenangaben wie Preise und Funktionen von Anbietern müssen wir immer prüfen. Die Architektur haben wir entschieden: Monolith, keine Datenbank, Frontend getrennt gehostet bei Render.
