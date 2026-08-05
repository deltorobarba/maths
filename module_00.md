Gut. Modul 0 ist das kürzeste Modul, aber alles Weitere hängt daran. Los geht's.

## Die vier Loops

Softwareentwicklung läuft nicht linear, sondern in vier ineinander verschachtelten Schleifen. Wichtig: verschachtelt, nicht nacheinander. Du durchläufst den Inner Loop 30–50 Mal, bevor du einmal den Submit Loop durchläufst.

**Design Loop** (Observe → Analyze → Design → Deliberate → Plan)
Hier wird entschieden, *was* überhaupt gebaut wird. Observe heißt: Signale sammeln — Bugreports, Metriken, Support-Tickets, Nutzerfeedback. Analyze heißt: das echte Problem vom Symptom trennen. Design heißt: Lösungsoptionen mit Trade-offs. Deliberate heißt: die Entscheidung gegen andere Menschen validieren (Design Review, RFC). Plan heißt: in umsetzbare Stücke zerlegen. Artefakte sind Design Docs, RFCs, ADRs, Tickets. Zykluszeit: Stunden bis Wochen.

**Inner Loop** (Think → Code → Build → Test)
Der Loop, in dem du gefühlt die meiste Zeit verbringst. Zykluszeit: Sekunden bis Minuten. Das entscheidende Qualitätsmerkmal hier ist nicht Korrektheit, sondern **Latenz**. Ein Build, der 4 Minuten braucht, zerstört den Loop — nicht weil 4 Minuten viel Zeit sind, sondern weil du in der Wartezeit den Kontext verlierst und Slack öffnest. (Nebenbei: „Build" gibt es auch in Python — Dependencies auflösen, Container bauen, Assets kompilieren.)

**Submit Loop** (Lint → PreSubmit → Code Review → Submit)
Der Übergang von *meinem* Code zu *unserem* Code. PreSubmit sind die automatisierten Checks, die vor dem Merge laufen müssen — in GitHub-Sprache: required status checks auf dem PR. Zykluszeit: Minuten bis Tage, wobei die Review-Latenz fast immer der Engpass ist, nicht die Maschine.

Ein Punkt, den fast alle Quereinsteiger falsch verstehen: Der Hauptzweck von Code Review ist **nicht** Fehlerfinden. Darin sind Maschinen inzwischen besser. Es geht um Wissensverteilung im Team, Design-Feedback und geteilte Verantwortung. Wer Review als Fehlersuche begreift, führt sie schlecht durch.

**Outer Loop** (Postsubmit → Staging → Canary → Production → Measure)
Postsubmit sind die Tests, die nach dem Merge auf `main` laufen — zu langsam für den PR, aber notwendig. Danach die Deployment-Kette. Zykluszeit: Stunden bis Wochen. Und **Measure schließt den Kreis** zurück zu Observe im Design Loop. Das ist der eigentliche Grund, warum man das als Loops zeichnet und nicht als Pipeline.## Das Grundgesetz

Die Kosten eines Fehlers wachsen ungefähr exponentiell mit der Distanz zwischen Entstehung und Entdeckung. Die alte Faustzahl aus der Literatur ist „Faktor 10 pro Stufe" — nimm sie nicht als Messwert, die empirische Basis dafür ist dünn. Als Denkmodell ist die Richtung aber unbestritten:

- Typfehler im Editor: 2 Sekunden
- Derselbe Fehler im PR: 20 Minuten Rework plus Review-Runde
- Derselbe Fehler in Produktion: Incident, Rollback, Postmortem, verlorenes Vertrauen

Daraus folgt der Begriff, der dir überall begegnen wird: **shift left**. Praktisch jede Engineering-Praktik, die du in den nächsten Modulen lernst, ist ein Versuch, eine bestimmte Fehlerklasse eine Stufe nach links zu ziehen. Typechecker zieht Laufzeitfehler in den Editor. Pre-Commit-Hook zieht CI-Fehler auf deine Maschine. Canary zieht einen Produktionsausfall auf 1 % der Nutzer. Feature Flag entkoppelt Deployment von Release, damit ein Fehler nicht gleichzeitig sichtbar wird.

Der Design Loop steht im Diagramm oben und nicht in der Kostenreihe, weil seine Fehler anders funktionieren: Sie werden nicht mit der Zeit teurer, sie **entwerten von Anfang an alles darunter**. Wenn du das Falsche baust, ist 100 % der Implementierung Verschwendung, egal wie sauber sie war. Deine eigenen Notizen formulieren das als „Fehler in der Planung sind im Nachhinein extrem teuer."

## Wo AI ansetzt

Damit wird das Overlay aus Addy Osmanis Diagramm lesbar — es markiert pro Loop, wo AI Toil reduziert:

| Loop | AI-Ansatzpunkte |
|---|---|
| Design | Chatbot für Entwickler, AI Auto-triage |
| Inner | Code completion & intent-to-code, AI refactorings, automated testing, AI-assisted debug |
| Submit | AI-assisted code review |
| Outer | Resource efficiency, AI performance refactorings |

## Der Punkt, den fast alle übersehen

Das ist für dich beruflich der wertvollste Teil des Moduls, deshalb steht er am Ende und nicht nebenbei.

AI beschleunigt den **Inner Loop** massiv. Sie beschleunigt Submit und Outer Loop kaum. Review braucht weiterhin menschliche Aufmerksamkeit, Testsuiten laufen weiterhin gleich lang, Deployments tragen weiterhin dasselbe Risiko.

Konsequenz: **Der Engpass wandert nach rechts.** Ein Team, das 5× so viele Änderungen produziert, aber dieselbe Review-Kapazität, dieselbe Testlaufzeit und dieselbe Deployment-Frequenz hat, wird nicht 5× schneller — es staut sich vor dem Submit Loop. Das ist der strukturelle Grund für Verification Debt (Modul 10) und die eigentliche Antwort auf die Frage „Warum bringt AI uns im Team weniger, als versprochen wurde?" Meist ist der Engpass eben nicht das Tippen.

Und genau deshalb existiert das Harness-Konzept aus deinen Notizen: Es automatisiert die rechten Loops mit, damit sie mit dem Inner Loop Schritt halten können.

## Kernbegriffe für dein Glossar

Inner/Submit/Outer/Design Loop · Zykluszeit · Shift left · Toil · PreSubmit vs. Postsubmit · Design Doc · RFC · ADR · Rework · Review-Latenz

## Hands-on (~30 Minuten)

Nimm eine echte Aufgabe, die du in den letzten Wochen gemacht hast, und ordne sie den Loops zu — grob geschätzt, wie viel Zeit steckte wo? Fast alle unterschätzen den Design Loop dramatisch.

Dann notiere für dein Projekt vier Zahlen: Build-Zeit, Testlaufzeit, Review-Latenz, Deploy-Zeit. Das ist deine Baseline. Wir kommen in Modul 11 darauf zurück, wenn wir über Wirkungsmessung reden — und du wirst sehen, dass sich diese vier Zahlen über die Agenda hinweg messbar verändern.

## Checkpoint

Vier Fragen. Antworte in eigenen Worten, ich korrigiere ehrlich:

1. Ein Pre-Commit-Hook, der `mypy` ausführt — in welchem Loop sitzt er, welche Fehlerklasse zieht er nach links, und wo wäre der Fehler sonst aufgetaucht?
2. Warum ist eine Testsuite, die 25 Minuten läuft, ein *Design*-Problem und nicht nur eine Unbequemlichkeit?
3. Ein Team führt AI-Coding-Tools ein. Die Anzahl der PRs pro Woche verdoppelt sich, die durchschnittliche Zeit von PR-Erstellung bis Merge steigt von 6 auf 30 Stunden. Was ist passiert, und in welchem Loop liegt das Problem?
4. Warum habe ich den Design Loop im Diagramm über die anderen drei gesetzt statt links daneben?
