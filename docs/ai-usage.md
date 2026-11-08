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

## Fazit

AI ist stark darin, eine Struktur zu schaffen und Lücken zu finden. Sie neigt aber dazu, zu viel zu spezifizieren, und setzt Zahlen, die plausibel klingen. Was ins Produkt kommt, haben wir entschieden.
