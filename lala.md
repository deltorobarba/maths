# Modul 9: Das Agent-Harness bauen

Das ist das Synthese-Modul. Alles aus Phase 1 und 2 kommt hier zusammen, und die These aus deinen Notizen — *„Verifikation schlägt Modellgröße"* — lässt sich jetzt präzise begründen.

## 1. Was ein Harness ist

Der Begriff kommt aus dem Testing: Ein *Test Harness* ist die Umgebung, in der ein Prüfling läuft, angesteuert und beobachtet wird. Ein **Agent-Harness** ist entsprechend die Gesamtheit aus Instruktionen, Werkzeugen, Prüfungen, Grenzen und Gates, in denen ein Agent arbeitet.

Die technisch sauberste Beschreibung ist die eines **Regelkreises**. Ein Agent ohne Rückkopplung ist eine Steuerung: Er handelt nach bestem Wissen und erfährt nie, ob es funktioniert hat. Ein Agent mit Rückkopplung ist eine Regelung: Jede Handlung erzeugt ein messbares Signal, das die nächste Handlung korrigiert.

Damit lässt sich die These aus deinen Notizen präzisieren. Ein Harness bestimmt die **Stärke und Geschwindigkeit der Kopplung zwischen Handlung und Konsequenz**. Ein starkes Modell im offenen Regelkreis rät gut. Ein schwächeres Modell im geschlossenen Regelkreis iteriert — und Iteration mit Feedback schlägt einmaliges Raten, auch wenn das Raten besser ist. Deshalb schlägt Verifikation Modellgröße. Nicht weil Modellqualität egal wäre, sondern weil sie den falschen Freiheitsgrad optimiert, solange die Rückkopplung fehlt.

## 2. Die fünf Schichten

| Schicht | Beantwortet | Bausteine | aus Modul |
|---|---|---|---|
| **Instruktionen** | Was gilt hier? | `AGENTS.md`, Konventionen | neu |
| **Werkzeuge** | Was kann er tun? | MCP, Makefile, Skripte | 2, 8 |
| **Verifikation** | Was wird geprüft? | Hooks, Lint, Typecheck, Tests | 2, 3 |
| **Grenzen** | Was darf er nicht? | Permissions, Sandbox, Verbotsliste | 1, 5, 6 |
| **Gates** | Was ist verbindlich? | CI, Branch Protection, Review | 5 |

Die entscheidende Unterscheidung liegt zwischen Schicht 1 und den Schichten 4/5. Instruktionen wirken **probabilistisch** — sie werden meistens befolgt. Grenzen und Gates wirken **deterministisch** — sie können nicht umgangen werden. Dieser Unterschied ist der rote Faden dieses Moduls.

## 3. `AGENTS.md` richtig schreiben

Die Datei, die zu Beginn jeder Sitzung gelesen wird. `AGENTS.md` hat sich als werkzeugübergreifender offener Standard etabliert; `CLAUDE.md` und `GEMINI.md` sind toolspezifische Varianten desselben Gedankens, und die meisten Werkzeuge lesen inzwischen beides.

**Die goldene Regel: Befehle, nicht Prosa.** Der Wert einer Zeile bemisst sich daran, ob sie eine Handlung eindeutig determiniert.

```markdown
## Verifikation
Vor Abschluss jeder Aufgabe: `make lint && make test`
Beides muss Exit-Code 0 liefern. Keine Ausnahmen.

## Verboten
- Tests ändern oder mit @skip/@xfail markieren, um sie grün zu bekommen
- `# noqa` oder `# type: ignore` ohne Fehlercode und Begründung
- Änderungen an `.github/`, `Makefile`, `.pre-commit-config.yaml`
- `git add -A`, `git commit --no-verify`, `git push --force`
- Schema-Migrationen ohne Expand-Migrate-Contract (siehe docs/adr/0012)

## Struktur
src/domain/    reine Geschäftslogik, importiert nichts aus infra/
src/infra/     Adapter für DB, HTTP, externe APIs
tests/unit/    ohne I/O, laufen in Millisekunden
```

**Was nicht hineingehört:**

- Alles, was der Agent im Code sehen kann. Die Ordnerstruktur zu beschreiben, die er auflisten kann, kostet Kontext und liefert nichts.
- Generische Qualitätsprosa. „Schreibe sauberen, wartbaren Code" verändert das Verhalten nicht messbar.
- Wissen, das veraltet. Jede Zeile, die nicht gepflegt wird, wird irgendwann zu einer Falschaussage, die aktiv schadet.

**Und der Punkt, den fast niemand ausspricht: `AGENTS.md` ist teuer.** Es liegt in *jedem* Kontext, in *jeder* Sitzung. Eine 400-Zeilen-Datei konkurriert dauerhaft mit dem eigentlichen Arbeitskontext und trifft direkt den Effekt aus Modul 8 — Signalverdünnung. Halte sie klein. Wenn sie wächst, ist das ein Signal, dass Regeln zu Checks werden sollten (Abschnitt 9).

Zwei praktische Ergänzungen: In Monorepos funktioniert eine **Hierarchie** — eine Wurzeldatei mit dem Gemeinsamen, pro Paket eine kleine mit den Besonderheiten. Und: **`AGENTS.md` ist Code.** Versioniert, im Review, in den CODEOWNERS. Eine Regel, die ein Agent selbst hinzufügen darf, ist keine Regel.

**Die wichtigste Heuristik:** Jede Regel sollte aus einem *beobachteten* Fehler entstanden sein, nicht aus einer Vorstellung davon, was schiefgehen könnte. Ausgedachte Regeln füllen Kontext und schützen vor nichts.

## 4. Werkzeuge

**MCP** (Model Context Protocol) hat sich als Standard für die Anbindung von Werkzeugen und Datenquellen durchgesetzt — ein Adapter-Protokoll, das dieselbe Rolle spielt wie Ports and Adapters in Modul 4: Der Agent kennt die Schnittstelle, nicht die Implementierung.

Drei Gestaltungsregeln, die aus Modul 8 folgen:

- **Knappe Ausgaben.** Ein Werkzeug, das 3000 Zeilen zurückgibt, ist ein Kontextzerstörer.
- **Wenige, klar abgegrenzte Werkzeuge.** Die Auswahlqualität sinkt mit der Anzahl ähnlich klingender Optionen.
- **Ein Skript schlägt oft einen Server.** Ein `make check-migrations` ist deterministisch, versioniert, lokal ausführbar und für Mensch und Agent identisch. Der reflexhafte Griff zum MCP-Server ist häufig die aufwändigere Lösung für ein Problem, das ein Makefile-Ziel löst.

Zur Sicherheit nur der Hinweis, den Modul 10 vertieft: Ein MCP-Server ist fremder Code mit Zugriff auf deine Daten und deine Werkzeuge. Die Auswahl ist eine Lieferkettenentscheidung.

## 5. Verifikation

Die Reibungsleiter aus Modul 2, jetzt vollständig — mit der entscheidenden Spalte ganz rechts:

| Station | Budget | Was läuft | Wirkung |
|---|---|---|---|
| Agent-Hook | 1–10 s | Format, Lint, Typecheck auf geänderte Dateien | probabilistisch |
| Agent-Testlauf | 10–60 s | schnelle Unit-Tests | probabilistisch |
| Pre-Commit | unter 5 s | Format, Lint, Secret Scan | umgehbar |
| CI-Pipeline | Minuten | alles, vollständig | **verbindlich** |
| Review | Stunden | Design, Fachlogik, Test-Diff | **verbindlich** |

Der eine Befehl aus Modul 2 gilt hier verstärkt: `make lint && make test` — von dir, vom Hook, vom Agenten und von der CI aufgerufen. Eine Wahrheit, kein Drift.

Und Geschwindigkeit zählt hier härter als bei Menschen. Ein Mensch führt die Testsuite fünfmal am Tag aus. Ein Agent führt sie dreißigmal in einer Sitzung aus. Zwei Minuten Testlaufzeit sind für dich eine Kaffeepause und für den Agenten eine Stunde.

## 6. Grenzen — die unterschätzte Schicht

Die Verbotsliste ist die Ernte aus allen vorherigen Modulen. Jeder Eintrag steht für ein Ausweichmanöver, das aus Sicht des Agenten eine Lösung ist:

| Verbot | Warum | Modul |
|---|---|---|
| Tests anpassen, um sie grün zu bekommen | zerstört das Abbruchkriterium | 3 |
| Lint-Regeln lockern, `noqa` streuen | zerstört das Signal | 2 |
| `.github/` oder `Makefile` ändern | löscht das Gate selbst | 5 |
| `git add -A`, `--no-verify`, `--force` | committet Secrets, umgeht Prüfungen | 1 |
| Migration ohne Expand-Migrate-Contract | verhindert Rollback, erzeugt Downtime | 6 |
| Flags anlegen ohne Ablaufdatum | Zustandsraum wächst still | 6 |

Und jetzt der Punkt, der zählt: **Diese Verbote gehören nicht nur in `AGENTS.md`, sondern in Mechanismen.**

Eine Instruktion konkurriert mit dem Ziel des Agenten. Wenn die Aufgabe „mach die Tests grün" lautet und der ehrliche Weg schwierig ist, ist das Anpassen des Tests der naheliegende Pfad — nicht aus Böswilligkeit, sondern weil das formulierte Ziel es belohnt. Ein Agent unter genügend Druck umgeht jede Regel, die nur eine Regel ist.

Die Durchsetzung sieht deshalb so aus: CODEOWNERS auf `tests/`, `.github/` und Migrationsverzeichnisse. Branch Protection, die auch für Bot-Tokens gilt. Werkzeugberechtigungen, die bestimmte Befehle gar nicht erst anbieten. Und, für autonome Läufe, **Sandboxing** — Container mit begrenztem Dateisystemzugriff, eigener Worktree (Modul 1), eingeschränkter Netzwerkzugang. Die Faustregel: *Je mehr Autonomie du gibst, desto mehr Isolation musst du bauen.*

## 7. Sub-Agenten und das Orchestrator-Worker-Muster

Deine Notizen beschreiben das Muster; hier ist es mit einer Ergänzung, die dort fehlt.**Die Ergänzung, die in deinen Notizen fehlt:** Der oberste Kasten. Das Orchestrator-Modell plant *innerhalb* eines Rahmens, den ein Mensch gesetzt hat — Spezifikation, Nicht-Ziele, Constraints (Modul 8), Freigabe. Die Design-Loop-Entscheidungen aus Modul 4 sind genau die, für die kein Modell die Verantwortung übernehmen kann, weil es die Zukunft deiner Fachdomäne nicht kennt. Ein Orchestrator-Modell ohne diesen Rahmen plant plausibel und in die falsche Richtung.

Drei praktische Punkte zu Sub-Agenten:

**Der Nutzen ist primär Kontextisolation.** Ein Unteragent, der 40.000 Tokens Recherche betreibt und drei Absätze zurückgibt, hält den Hauptkontext sauber. Das ist der Mechanismus aus Modul 8, in Systemform.

**Die Übergabe ist verlustbehaftet.** Der Sub-Agent weiß nicht, was der Hauptagent weiß. „Mach da weiter" funktioniert nicht — die Aufgabenbeschreibung muss vollständig und schriftlich sein, so als schriebe man sie einem neuen Kollegen. Das ist derselbe Anspruch wie an eine Spec.

**Parallelität hat Folgekosten.** Drei Agenten in drei Worktrees (Modul 1) kollidieren nicht auf der Dateiebene — aber semantische Merge-Konflikte (Modul 5) werden häufiger, nicht seltener. Ab hier ist eine Merge Queue kein Luxus mehr.

## 8. Multi-Modell-Routing und Tokenomics

Deine Notizen argumentieren: großes Modell für Planung und Architektur, schnelles Modell für die laufende Implementierung. Das ist ökonomisch richtig, und zwar aus einem Grund, den man explizit machen sollte.

**Die richtige Metrik sind Kosten pro erfolgreich abgeschlossener Aufgabe — nicht Kosten pro Token.** Ein billiges Modell, das fünf Iterationen braucht und dann doch am Review scheitert, ist teurer als ein teures, das in einem Durchgang liefert. Die Token-Preisliste ist die verführerischste falsche Metrik in diesem Feld.

Daraus die Aufteilung:

- **Großes Modell lohnt** bei Planung, Architekturentscheidungen, schwierigen Bugs mit unklarer Ursache und Code-Review. Gemeinsames Merkmal: wenige Tokens, hohe Konsequenz eines Fehlers. Das ist Modul 0 — links im Loop ist ein Fehler am teuersten.
- **Kleines Modell reicht** bei mechanischer Umsetzung nach klarer Spec, Boilerplate, Testschreiben nach Vorlage, Formatierungs- und Migrationsarbeit über viele Dateien. Gemeinsames Merkmal: viele Tokens, sofort verifizierbar.

**Im Inner Loop zählt Latenz mehr als Kosten.** Ein Modell, das 40 Sekunden für eine kleine Änderung braucht, zerstört den Loop aus denselben Gründen wie ein langsamer Build — der Kontextwechsel ist das Teure, nicht die Rechenzeit.

**Provisioned Throughput** ist reservierte Kapazität gegen Fixpreis statt Abrechnung pro Token. Die Logik deiner Notizen ist plausibel: Das stetige, hohe Volumen entsteht im Implementierungs-, Test- und Debugging-Loop, also beim Worker-Modell — dort liefert reservierte Kapazität Kostenkontrolle und stabile Latenz. Das Planungsmodell wird punktuell aufgerufen, dort reicht die normale Abrechnung. Die konkreten Modellnamen und Versionsnummern in deinem Dokument kann ich nicht verifizieren; das Argument steht unabhängig davon.

Der billigste Hebel bleibt aber **Prefix-Caching** (Modul 8): Statisches nach vorn, Variables nach hinten. Das kostet einen Nachmittag und wirkt dauerhaft.

## 9. Der Feedback-Loop auf das Harness selbst

Deine Notizen nennen es „Aus den Workflows lernen". Das ist der Punkt, der aus einem Setup ein lebendes System macht.

Die Praxis: Regelmäßig — nach jedem Sprint, jede Woche — die Frage stellen, welche Fehler *wiederholt* aufgetreten sind. Für jeden wiederholten Fehler dann die entscheidende Frage: Ist das eine Regel, ein Check oder eine Grenze?

Und dazu die Rangordnung, die selten ausgesprochen wird:

1. **Am besten: ein automatischer Check**, der den Fehler unmöglich macht. Ein Lint-Regel, ein Test, ein Skript in der Pipeline.
2. **Zweitbeste: eine Grenze**, die ihn verhindert. Permission, CODEOWNERS, Sandbox.
3. **Schlechteste: eine Regel** in `AGENTS.md`.

Der Grund folgt aus Abschnitt 2: Regeln wirken probabilistisch, Checks deterministisch. Und Regeln kosten dauerhaft Kontext, während ein Check nur dann etwas kostet, wenn er anschlägt.

Daraus folgt ein Satz, der dein Harness-Design leiten sollte:

> **Jede Regel in `AGENTS.md` ist ein Eingeständnis, dass du keinen Check bauen konntest.**

Manchmal stimmt das — nicht alles ist prüfbar, und für Designkonventionen bleibt die Regel die einzige Option. Aber der Reflex sollte umgekehrt laufen als üblich: Wenn eine Regel dreimal verletzt wurde, ist die richtige Reaktion nicht, sie fetter zu schreiben, sondern zu fragen, wie man sie automatisiert prüfbar macht.

## 10. Wann das Harness zu viel ist

Ein Harness hat Kosten: Wartung, Latenz in jedem Loop, Kontextverbrauch, kognitive Last beim Onboarding. Für einen Prototyp, ein Wegwerf-Skript oder eine explorative Analyse ist der volle Aufbau Overkill und verlangsamt dich.

Die Entscheidungsfrage ist dieselbe wie in Modul 0: **Wie oft läuft dieser Loop, und was kostet ein Fehler?** Ein Skript, das einmal läuft und dessen Ausgabe du selbst prüfst, braucht kein Gate. Ein Dienst mit Nutzern braucht alles aus Abschnitt 2.

Ein pragmatischer Einstieg in dieser Reihenfolge, wenn du bei null anfängst: Formatter und Linter im Hook → schnelle Testsuite mit einem Einstiegsbefehl → `AGENTS.md` mit Prüfbefehlen und Verbotsliste → CI mit Branch Protection → Rest nach Bedarf. Die ersten drei Schritte liefern den größten Teil des Effekts.

## 11. Anti-Patterns

- `AGENTS.md` als Aufsatz über Codequalität statt als Befehlsliste
- Eine `AGENTS.md`, die über die Zeit auf 500 Zeilen wächst, statt dass Regeln zu Checks werden
- Verbote nur als Instruktion, ohne Mechanismus dahinter
- Ein Bot-Token mit mehr Rechten als ein Mensch
- Der Agent darf die eigene Prüfinfrastruktur ändern
- Regeln, die aus Vorstellung entstanden sind statt aus beobachteten Fehlern
- Kosten pro Token optimieren statt Kosten pro abgeschlossener Aufgabe
- Sub-Agenten mit vager Aufgabenbeschreibung („mach weiter")
- Parallele Agenten ohne Worktrees und ohne Merge Queue
- Autonomie ohne entsprechende Isolation
- Ein Harness, das nie überarbeitet wird, obwohl dieselben Fehler wiederkehren

## 12. Glossar

Agent-Harness · Offener vs. geschlossener Regelkreis · `AGENTS.md` / `CLAUDE.md` / `GEMINI.md` · Hierarchische Instruktionsdateien · MCP · Werkzeugausgabe-Disziplin · Probabilistische Regel vs. deterministischer Check · Verbotsliste · Sandboxing · Worktree-Isolation · Sub-Agent · Kontextübergabe · Orchestrator-Worker · Multi-Modell-Routing · Kosten pro abgeschlossener Aufgabe · Provisioned Throughput · Prefix-Caching · Harness-Retrospektive

## 13. Übungen

**Ü1 — Die minimale `AGENTS.md`.** Schreibe eine Version mit maximal 40 Zeilen: Prüfbefehle, Struktur in drei Zeilen, Verbotsliste. Nichts, was der Agent im Code sehen kann. Miss die Tokenzahl.

**Ü2 — Regeln aus Beobachtung ableiten.** Nimm die Notizen aus Ü6 und Ü7 von Modul 4 und Ü7 aus Modul 3. Formuliere daraus drei Regeln, die jeweils auf einen von dir tatsächlich beobachteten Fehler zurückgehen. Prüfe für jede: Könnte das auch ein Check sein?

**Ü3 — Vom Regel zum Check.** Nimm eine deiner Regeln und baue sie als automatische Prüfung — ein Lint-Plugin, ein Test, ein Skript im Makefile. Entferne dann die Regel aus `AGENTS.md` und prüfe, ob der Fehler weiterhin abgefangen wird.

**Ü4 — Grenzen scharfschalten.** Richte CODEOWNERS auf `tests/`, `.github/` und dein Migrationsverzeichnis ein. Lass dann einen Agenten eine Aufgabe bearbeiten, bei der die einfachste Lösung wäre, einen Test zu ändern — und beobachte, was der PR-Prozess damit macht.

**Ü5 — Werkzeugausgaben kürzen.** Baue ein `make test-agent`, das nur fehlgeschlagene Tests mit gekürztem Traceback ausgibt. Vergleiche die Kontextlänge über einen kompletten Agentenlauf mit und ohne.

**Ü6 — Orchestrator-Worker aufsetzen.** Zerlege ein Feature per Planungsmodell in drei Aufgaben, lass sie von einem schnellen Modell in drei Worktrees parallel umsetzen, und führe sie zusammen. Notiere, wo es hakt — insbesondere bei der Zusammenführung.

**Ü7 — Kosten pro Aufgabe messen.** Dieselbe Aufgabe einmal mit einem großen und einmal mit einem kleinen Modell. Erfasse Tokenverbrauch, Anzahl der Iterationen, Wanduhrzeit und ob das Ergebnis den Review bestanden hätte. Rechne auf Kosten pro *erfolgreicher* Aufgabe um.

**Ü8 — Harness-Retrospektive.** Führe eine Woche lang eine Liste der Fehler, die Agenten in deinem Projekt gemacht haben. Ordne am Ende jeden Eintrag der Rangordnung aus Abschnitt 9 zu und setze die drei häufigsten um. Das ist die Übung, die dein Harness von einem Setup zu einem System macht.

Modul 10 wäre als Nächstes dran: AI-Code kritisch prüfen. Dort geht es um die Fehlerklassen, die dein Harness *nicht* abfängt — Sicherheit, Lieferkette, Prompt Injection in agentischen Workflows und die Verification Debt, die still mitwächst.
