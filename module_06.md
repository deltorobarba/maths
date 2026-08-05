# Modul 6: Release Engineering

Hier räumen wir deinen ausdrücklich genannten Blindspot ab. Und wir fangen mit dem Satz an, der das ganze Modul trägt.

## 1. Deployment ist nicht Release

> **Deployment** heißt: Der Code läuft auf der Infrastruktur.
> **Release** heißt: Nutzer sehen das neue Verhalten.

Die meisten Teams behandeln das als dasselbe Ereignis. Dann ist jedes Deployment ein Risiko, Deployments werden selten, seltene Deployments werden groß, große Deployments werden gefährlich — und die Angst vor dem Deployment wird zur selbsterfüllenden Prophezeiung.

Entkoppelst du die beiden, ändert sich die Ökonomie vollständig. Code kann unbemerkt in Produktion liegen. Das Deployment wird zu einem technischen Vorgang ohne Publikum, und das Release wird zu einer Entscheidung, die sich in Sekunden zurücknehmen lässt. Das Ziel von Release Engineering ist nicht Perfektion, sondern **Langeweile**: Ein Deployment soll ein Nicht-Ereignis sein.

Alles Weitere in diesem Modul — Canary, Feature Flags, progressive Delivery — sind Techniken, um diese Entkopplung herzustellen.

## 2. Umgebungen — und warum Staging immer lügt

Die übliche Kette: **Local → Dev/Test → Staging → Production.**

Staging soll Produktion nachbilden. Das gelingt nie vollständig, und die Gründe sind strukturell, nicht behebbar: andere Datenmenge, andere Datenverteilung (echte Kundendaten haben Sonderfälle, die kein Testdatensatz hat), andere Last, andere Nachbarsysteme, andere Skalierung, andere Netzwerktopologie, kein echtes Nutzerverhalten.

Daraus folgt der Satz, der viele überrascht: **„Es lief auf Staging" ist kein Beweis.** Staging schließt eine Klasse grober Fehler aus. Es schließt nicht aus, dass eine Query bei 40 Millionen Zeilen in einen Timeout läuft, obwohl sie bei 40.000 flott war.

Und genau daraus folgt der Rest dieses Moduls. Weil die einzige Umgebung, die Produktion ist, Produktion ist, verlagert man einen Teil der Verifikation dorthin — kontrolliert, mit Netz und mit begrenzter Blastradius. „Testing in Production" ist deshalb kein Zynismus, sondern eine reife Praktik. Ohne Netz ist es Leichtsinn, mit Netz ist es das Einzige, was ehrlich ist.

Zwei ergänzende Konzepte: **Environment Parity** (die Zwölf-Faktoren-Forderung, die Unterschiede zwischen Umgebungen aktiv klein zu halten — gleiche Datenbankversion, gleiche Backing Services, nicht SQLite lokal und Postgres in Produktion) und **Ephemeral Environments** — pro Pull Request eine vollständige, kurzlebige Umgebung, die beim Merge verschwindet. Letzteres hat sich mit Container-Orchestrierung von einem Luxus zu einer realistischen Option entwickelt und ist besonders wertvoll, wenn Agenten viele PRs parallel produzieren.

## 3. Die vier Deployment-Strategien

**Recreate** — alte Version stoppen, neue starten. Es gibt ein Zeitfenster ohne Service. Für interne Werkzeuge und Nachtfenster völlig in Ordnung, und es hat den Vorteil, dass nie zwei Versionen gleichzeitig laufen (was, wie du in Abschnitt 6 sehen wirst, das Migrationsproblem stark vereinfacht).

**Rolling** — Instanzen werden nacheinander ersetzt, meist in kleinen Gruppen. Keine Downtime, kein zusätzlicher Ressourcenbedarf. Der Standard in Kubernetes. Zwei Eigenschaften, die man kennen muss: Während des Vorgangs laufen **beide Versionen gleichzeitig** — und ein Rollback ist genauso langsam wie der Rollout, weil er derselbe Vorgang rückwärts ist.

**Blue-Green** — zwei vollständige, parallele Umgebungen. Grün läuft, Blau wird mit der neuen Version bespielt und geprüft, dann schaltet der Load Balancer um. Der Vorteil ist der Rollback: zurückschalten, Sekunden. Der Preis: zeitweise doppelte Infrastruktur — und alles, was Zustand hält (Datenbank, Sessions, Caches), wird geteilt und muss deshalb zu beiden Versionen passen.

**Canary** — der Rest dieses Abschnitts.

Drei Mechanik-Details, die bei allen zustandsbehafteten Varianten mitgedacht werden müssen: **Connection Draining** (eine Instanz, die heruntergefahren wird, bekommt `SIGTERM`, nimmt keine neuen Anfragen mehr an, beendet die laufenden und geht dann), **Session-Affinität** (wenn Nutzer an eine Instanz gebunden sind, verhält sich ein Rollout ganz anders) und **Warm-up** (frische Instanzen haben kalte Caches, leere Connection Pools und, bei JVM-Sprachen, nicht optimierten Code — sie sind für einige Minuten messbar langsamer).

## 4. Canary in der Praxis

Der Name kommt aus dem Bergbau: Der Kanarienvogel im Käfig zeigt giftige Luft an, bevor sie die Bergleute umbringt. Die Idee ist dieselbe — eine kleine Menge realer Nutzer trifft die neue Version zuerst, und ihre Erfahrung ist das Signal.Der Ablauf konkret:

**Schritt 1 — Routing.** Die neue Version wird deployt und bekommt einen kleinen Traffic-Anteil. Die Aufteilung passiert im Load Balancer, im Ingress oder im Service Mesh — nach Prozent, nach Header, nach Nutzerkohorte oder nach Region.

**Schritt 2 — Automated Canary Analysis.** Hier steckt die eigentliche Intelligenz, und hier machen die meisten den ersten Fehler.

Man vergleicht **nicht gegen absolute Schwellen**, sondern gegen eine **Baseline**. Eine Regel wie „Fehlerrate unter 0,5 %" ist wertlos, weil die Fehlerrate von Tageszeit, Lastspitzen, einem hakenden Nachbarsystem und dem Wetter abhängt. Du willst den Effekt *deiner Änderung* isolieren — also vergleichst du Canary gegen die parallel laufende alte Version, im selben Zeitfenster, unter derselben Last.

Und die Feinheit, an der man Fachleute erkennt: **Die Baseline sollte ebenfalls frisch deployt sein.** Vergleichst du deine neue Instanz gegen die seit drei Wochen laufende Produktion, misst du zum großen Teil den Unterschied zwischen kalt und eingelaufen — leere Caches, kalte Connection Pools, nicht optimierter Code. Der saubere Aufbau deployt deshalb *beide* Versionen frisch nebeneinander und vergleicht die beiden. Netflix' Kayenta hat das populär gemacht.

**Schritt 3 — die Metriken.** In der Reihenfolge, in der sie relevant werden:

- **Fehlerrate** — HTTP 5xx, Exception-Rate, fehlgeschlagene Jobs. Reagiert am schnellsten.
- **Latenz als Perzentile**, nicht als Mittelwert. Das ist wichtig: Wenn 1 % deiner Anfragen zehn Sekunden brauchen und der Rest 50 Millisekunden, sieht der Mittelwert harmlos aus — aber 1 % deiner Nutzer sind weg. Deshalb p50, p95, p99. Der p99 ist bei hohem Volumen der ehrlichste Indikator.
- **Sättigung** — CPU, Speicher, Verbindungen im Pool, Queue-Tiefe. Fängt Ressourcenlecks, die sich noch nicht in Fehlern zeigen.
- **Business-Kennzahlen** — Conversion, abgeschlossene Bestellungen, Anmeldungen. Die einzige Kategorie, die den Fall abfängt, in dem technisch alles grün ist und der Kaufen-Button trotzdem nicht mehr sichtbar ist. Sie brauchen mehr Traffic für Aussagekraft, kommen deshalb erst auf höheren Stufen ins Spiel.

**Schritt 4 — das Beobachtungsfenster.** Hier steckt der zweite Fehler, den viele machen. Bei 1 % Traffic hast du wenig Daten. Entscheidest du nach zwei Minuten, ist dein Ergebnis Rauschen — in beide Richtungen: Du rollst grundlos zurück, oder du gibst frei, obwohl du gar nichts gesehen hast. Jede Stufe braucht ein Fenster, das lang genug für statistische Aussagekraft ist. Die Faustregel geht über Ereigniszahlen, nicht über Minuten: Bei einer normalen Fehlerrate von 0,1 % brauchst du eine erhebliche Menge Anfragen, um eine Verdopplung überhaupt erkennen zu können.

**Schritt 5 — Freigabe oder Rollback.** Innerhalb der Schwellen: Anteil erhöhen. Außerhalb: automatisch zurück auf null. Der Punkt am automatischen Rollback ist, dass kein Mensch reagieren muss — Reaktionszeit ist genau das, was um 3 Uhr morgens fehlt.

**Und wann Canary nicht funktioniert** — das gehört zur Expertise dazu:

- **Batch-Jobs und asynchrone Verarbeitung.** Es gibt keinen Traffic zum Aufteilen.
- **Seltene Codepfade.** Ein Fehler im Jahresabschluss-Report wird von 1 % Traffic über zwanzig Minuten nicht getroffen.
- **Langsam auftretende Fehler.** Ein Speicherleck zeigt sich nach vier Stunden, nicht nach zwanzig Minuten. Canary schützt gegen akute Regressionen, nicht gegen schleichende.
- **Datenbankänderungen.** Dazu Abschnitt 6 — das ist der harte Teil.
- **Kleine Systeme.** Bei 200 Anfragen pro Stunde ergibt 1 % keine Aussage. Dann ist Blue-Green mit schnellem Rollback die ehrlichere Wahl.

## 5. Feature Flags — das Werkzeug der Entkopplung

Ein Feature Flag ist eine Verzweigung im Code, deren Zustand von außen gesteuert wird. Damit wird Abschnitt 1 mechanisch umsetzbar: Der Code ist deployt, das Verhalten ist aus.

Die Typen unterscheiden sich in Lebensdauer und Dynamik, und das Verwechseln ist die häufigste Ursache für Flag-Chaos:

| Typ | Zweck | Lebensdauer |
|---|---|---|
| **Release Toggle** | schrittweiser Rollout eines neuen Features | Tage bis Wochen — dann weg |
| **Experiment Toggle** | A/B-Test, Varianten gegeneinander messen | Dauer des Experiments |
| **Ops Toggle** | Notschalter, Lastabwurf, Circuit Breaker | dauerhaft |
| **Permission Toggle** | Feature nur für bestimmte Kunden oder Pläne | dauerhaft, fachlich |

Nur die ersten beiden sind temporär. Die letzten beiden sind dauerhafte Bestandteile deines Systems und dürfen es sein.

**Der Preis ist real.** Jedes Flag verdoppelt den Zustandsraum. Zehn gleichzeitig aktive Flags ergeben 1024 mögliche Kombinationen, von denen du eine Handvoll testest. Flags sind technische Schuld mit eingebautem Verfallsdatum — im Sinne von Modul 12: bewusst, besonnen, und mit Tilgungsplan.

Praktische Flag-Hygiene: jedes Flag mit Eigentümer und Ablaufdatum, ein Ticket zum Aufräumen wird gleich mit angelegt, und eine regelmäßige Durchsicht. Ein Release Toggle, das seit acht Monaten fest auf „an" steht, ist toter Code plus ein Schalter, den jemand versehentlich umlegen kann. Die Testfrage bleibt beantwortbar, wenn du dich auf den Zustand konzentrierst, der als Nächstes Produktion sein wird — nicht auf alle Kombinationen.

Zwei verwandte Techniken:

**Progressive Delivery** — Flags plus Kohorten statt Prozente: interne Mitarbeiter → Beta-Nutzer → 5 % → alle. Der Vorteil gegenüber zufälligen Prozenten ist, dass die frühen Kohorten Rückmeldung geben statt nur Metriken.

**Dark Launch / Shadow Traffic** — der neue Codepfad läuft parallel mit, seine Ausgabe wird verworfen oder gegen die alte verglichen, aber nie ausgeliefert. Das ist das mit Abstand beste Werkzeug für riskante Migrationen: Du bekommst echte Produktionslast auf den neuen Pfad, ohne dass ein Nutzer je betroffen ist. Wenn du ein Empfehlungsmodell, eine neue Suchimplementierung oder einen neuen Berechnungskern ablösen willst, ist das der Weg.

## 6. Datenbank-Migrationen — der eigentlich schwere Teil

Diesen Abschnitt lassen die meisten Übersichten aus, und er ist derjenige, der in der Praxis am meisten weh tut.

Der Grund: **Code kannst du zurückrollen. Daten nicht.**

Und daraus folgt das Kernproblem: Bei Rolling- und Canary-Deployments laufen **alte und neue Codeversion gleichzeitig gegen dieselbe Datenbank**. Also muss das Schema zu beiden passen. Eine Migration, die die alte Version bricht, macht dein Rollback unmöglich — genau in dem Moment, in dem du es brauchst.

Die Lösung heißt **Expand–Migrate–Contract** (auch: Parallel Change), und sie besteht darin, jede schemaverändernde Operation in mehrere, jeweils rückwärtskompatible Deployments zu zerlegen.

Beispiel: Du willst die Spalte `name` in `full_name` umbenennen. Naiv wäre das ein `ALTER TABLE ... RENAME COLUMN` — und in dem Moment ist jede noch laufende alte Instanz kaputt.

Richtig sind fünf getrennte Deployments:

1. **Expand** — neue Spalte `full_name` hinzufügen, nullable. Alter Code ignoriert sie, nichts bricht.
2. **Doppelt schreiben** — neuer Code schreibt in beide Spalten, liest aber weiter aus `name`.
3. **Backfill** — bestehende Zeilen befüllen, in Stapeln, nicht in einer Transaktion über zehn Millionen Zeilen.
4. **Umschalten** — Code liest aus `full_name`, schreibt weiter in beide.
5. **Contract** — Schreiben auf `name` einstellen, dann die Spalte entfernen. Erst wenn sicher ist, dass kein laufender Code sie mehr braucht.

Umständlich? Ja. Aber jeder einzelne Schritt ist gefahrlos zurückrollbar, und das ist der Punkt.

Zwei weitere Regeln, die daraus folgen: **Niemals eine neue Spalte direkt als `NOT NULL` anlegen** (das bricht jeden alten `INSERT`) — erst nullable, befüllen, dann die Bedingung nachziehen. Und **Vorsicht mit Locks**: Ein `ALTER TABLE` auf einer großen Tabelle kann je nach Datenbank und Operation minutenlang sperren und damit den gesamten Service anhalten. In Postgres brauchen manche Operationen deshalb `CONCURRENTLY`, und Indexerstellung gehört grundsätzlich in diese Kategorie.

Dasselbe Prinzip gilt übrigens für alles Versionierte an Systemgrenzen: API-Antworten, Message-Formate in Queues, Event-Schemata. Erst additiv erweitern, dann umstellen, dann alte Felder entfernen — nie in einem Schritt.

## 7. Rollback oder Roll-forward

**Rollback** — zurück auf die vorherige Version. Schnell und vorhersagbar, aber nur möglich, wenn keine irreversible Zustandsänderung passiert ist. Genau deshalb der Aufwand in Abschnitt 6.

**Roll-forward** — den Fehler nach vorn beheben und neu deployen. Nötig, wenn Rollback nicht geht. Setzt eine schnelle Pipeline voraus, sonst dauert es zu lange.

Die Realität ist meist eine Mischung, und die eigentliche Arbeit liegt vorher: **auf Reversibilität hin zu entwerfen**. Bei jeder Änderung die Frage stellen — kann ich das zurücknehmen, und wie lange dauert es?

Ein Punkt aus der Praxis, der oft übersehen wird: **Rollback ist eine Fähigkeit, keine Konfiguration.** Ein Rollback, den man nie geübt hat, funktioniert im Ernstfall nicht — weil das Artefakt nicht mehr existiert, weil die Migration nicht rückwärtskompatibel ist, weil niemand die Berechtigung hat, weil es nirgends dokumentiert ist. Übe es an einem Dienstagvormittag, nicht um 3 Uhr nachts.

## 8. Health Checks — ein Detail mit großer Wirkung

Zwei verschiedene Fragen, die häufig verwechselt werden:

- **Liveness** — „Lebt der Prozess noch?" Bei Fehlschlag: **neu starten**.
- **Readiness** — „Kann er gerade Anfragen annehmen?" Bei Fehlschlag: **aus dem Load Balancer nehmen**, aber nicht neu starten.

Der Unterschied ist praktisch bedeutsam. Eine Instanz, die gerade ihren Cache aufwärmt oder auf eine überlastete Datenbank wartet, ist *nicht bereit*, aber vollkommen *lebendig*. Prüft dein Liveness-Check die Datenbankverbindung mit, startet der Orchestrator bei einem Datenbankproblem sämtliche Instanzen gleichzeitig neu — und macht aus einer Störung einen Totalausfall. Diese Fehlkonfiguration ist erstaunlich verbreitet.

Damit ist der Outer Loop aus Modul 0 vollständig: Postsubmit → Staging → Canary → Production, dort Experiment, Flag Flip, Release → Measure. Und Measure führt zurück zu Observe im Design Loop.

## 9. Die AI-Verbindung

**Die Frage hat sich verschoben.** Mit AI-gestützter Entwicklung lautet der Engpass nicht mehr „können wir schnell bauen", sondern „können wir schnell und sicher ausrollen". Das Sicherheitsnetz aus diesem Modul ist das, was eine hohe Änderungsrate überhaupt erst nutzbar macht — ohne es ist mehr Durchsatz nur mehr Risiko pro Woche. In Modul 11 bekommt das eine Zahl: Deployment Frequency ohne Change Failure Rate zu betrachten ist bedeutungslos.

**Feature Flags werden von einer Option zur Notwendigkeit.** Trunk-Based Development plus Agenten, die viele Änderungen schnell auf `main` bringen, funktioniert nur, wenn Unfertiges deployt, aber inaktiv sein kann. Die beiden Praktiken hängen zusammen — man kann nicht die eine ohne die andere haben.

**Migrationen sind die gefährlichste Kategorie AI-generierten Codes.** Ein Agent, der eine Migration schreibt, kennt Expand–Migrate–Contract in aller Regel nicht — jedenfalls nicht ungefragt. Er schreibt das direkte `RENAME COLUMN`, weil das die naheliegende Antwort auf „benenne die Spalte um" ist. Im Review sieht das harmlos aus: eine Zeile, klar formuliert, korrekte Syntax. In Produktion bedeutet es Downtime und ein unmögliches Rollback. Das ist ein Fall, in dem die Regel explizit in die Agent-Instruktionen gehört und in dem Migrationen einen menschlichen Pflicht-Review über CODEOWNERS brauchen.

**Flag-Wildwuchs.** Agenten legen bereitwillig Flags an und räumen sie nie auf — Aufräumen ist keine Aufgabe, die sich jemand stellt. Ohne Ablaufdatum und regelmäßige Durchsicht wächst der Zustandsraum aus Abschnitt 5 still weiter.

**Wo AI hier tatsächlich hilft:** Anomalieerkennung in der Canary-Analyse, Zusammenfassung von Deployment-Diffs für die Freigabeentscheidung, Erkennung riskanter Migrationsmuster im PR. Die Schwellen und die Freigabekriterien bleiben dabei bei dir — ein Modell, das entscheidet, wann ein Rollout durchgewunken wird, ist ein Modell, das den Blastradius kontrolliert.

## 10. Anti-Patterns

- Deployment und Release als dasselbe Ereignis behandeln
- Canary gegen absolute Schwellen statt gegen eine Baseline prüfen
- Canary gegen eine lange eingelaufene Baseline vergleichen
- Latenz als Mittelwert statt als Perzentil beobachten
- Beobachtungsfenster zu kurz für statistische Aussagekraft
- Schemaänderungen, die die vorherige Codeversion brechen
- `NOT NULL` auf eine neue Spalte in einem Schritt
- Release Toggles ohne Ablaufdatum
- Rollback nie geübt
- Liveness-Check, der externe Abhängigkeiten mitprüft
- Freitags um 17 Uhr deployen, ohne dass jemand die Metriken anschaut

## 11. Glossar

Deployment vs. Release · Environment Parity · Ephemeral / Preview Environment · Recreate / Rolling / Blue-Green / Canary · Connection Draining · Graceful Shutdown · Session-Affinität · Warm-up · Automated Canary Analysis · Baseline · Perzentile (p50/p95/p99) · Sättigung · Beobachtungsfenster · Blastradius · Feature Flag · Release / Experiment / Ops / Permission Toggle · Kill Switch · Circuit Breaker · Progressive Delivery · Kohorte · Dark Launch · Shadow Traffic · Expand–Migrate–Contract / Parallel Change · Backfill · Rückwärtskompatibilität · `CONCURRENTLY` · Rollback vs. Roll-forward · Liveness vs. Readiness

## 12. Übungen

**Ü1 — Entkopplung herstellen.** Baue ein Feature Flag in dein Projekt ein, das von einer Umgebungsvariablen gesteuert wird. Deploye ein Feature im ausgeschalteten Zustand, prüfe, dass Nutzer nichts sehen, und schalte es dann ohne neues Deployment an.

**Ü2 — Rolling nachvollziehen.** Starte drei Instanzen deines Dienstes und ersetze sie einzeln durch eine neue Version, während ein Skript im Sekundentakt Anfragen schickt. Protokolliere, welche Antworten von welcher Version kamen — und beobachte, dass beide gleichzeitig live sind.

**Ü3 — Perzentile verstehen.** Erzeuge einen Datensatz mit 1000 Latenzwerten, davon zehn bei zehn Sekunden. Berechne Mittelwert, p50, p95 und p99. Der Vergleich ist das Argument aus Abschnitt 4 in einer Zeile.

**Ü4 — Canary-Gate definieren.** Schreibe für dein Projekt die Freigabekriterien auf: welche Metriken, welche Schwellen relativ zur Baseline, welches Beobachtungsfenster pro Stufe, was den automatischen Rollback auslöst. Eine Seite. Das ist ein echtes Artefakt, das du in einem Team vorlegen könntest.

**Ü5 — Expand–Migrate–Contract durchspielen.** Benenne eine Spalte in fünf getrennten Deployments um, wie in Abschnitt 6. Prüfe nach jedem Schritt, dass die *vorherige* Codeversion noch funktionieren würde. Das ist die lehrreichste Übung des Moduls.

**Ü6 — Rollback üben.** Deploye eine bewusst kaputte Version und rolle zurück. Stoppe die Zeit vom Erkennen bis zum wiederhergestellten Zustand. Notiere, woran es gehakt hat — bei den meisten hakt es beim ersten Mal.

**Ü7 — Health Checks trennen.** Implementiere `/healthz` (Liveness, prüft nur den Prozess) und `/readyz` (Readiness, prüft Abhängigkeiten). Simuliere dann einen Datenbankausfall und beobachte, dass die Instanz aus dem Load Balancer fällt, aber nicht neu gestartet wird.

**Ü8 — Der Migrationstest für Agenten.** Bitte einen Agenten, eine Spalte umzubenennen, ohne weitere Vorgaben. Sieh dir an, was er produziert. Formuliere dann die Regel, die das verhindert hätte, gib sie ihm mit und wiederhole. Notiere die Formulierung — sie geht später in deine `AGENTS.md`.

Modul 7 wäre als Nächstes dran: Observability und Betrieb. Das ist die Voraussetzung, ohne die Abschnitt 4 dieses Moduls nur Theater ist — denn du kannst nur auf Signale reagieren, die du auch misst.
