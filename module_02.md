# Modul 2: Der Inner Loop — statische Analyse & Hooks

Hier lösen wir das Zitat aus deinen Notizen vollständig auf. Am Ende weißt du nicht nur, was ein Linter von einem Typechecker unterscheidet, sondern auch, warum diese Kette der Grund ist, dass Agenten überhaupt zuverlässig arbeiten können.

## 1. Die Grundunterscheidung: statisch vs. dynamisch

Alles in diesem Modul ist **statische Analyse**: Werkzeuge, die deinen Code *lesen*, ohne ihn auszuführen. Tests (Modul 3) sind das Gegenteil — **dynamische Analyse**, sie führen aus und beobachten.

Das ist keine Wortklauberei, sondern bestimmt, welche Fehlerklassen du überhaupt finden kannst. Statische Analyse findet Fehler, die aus der *Struktur* des Codes folgen, und findet sie in jedem Codepfad — auch in dem, der nur alle drei Monate durchlaufen wird. Tests finden Fehler im *Verhalten*, aber nur in den Pfaden, die der Test auch tatsächlich betritt. Die beiden ergänzen sich, sie ersetzen sich nicht.

Ein technisches Detail, das erklärt, warum diese Werkzeuge so präzise sind: Sie arbeiten nicht auf Text, sondern auf dem **AST** (Abstract Syntax Tree) — der Baumdarstellung, in die ein Parser deinen Code überführt. Ein Linter sucht nicht nach der Zeichenfolge `except:`, er sucht nach einem Try-Knoten mit einem Handler ohne Typangabe. Deshalb lässt er sich nicht durch Formatierung, Kommentare oder Zeilenumbrüche täuschen — und deshalb ist selbstgebautes „Linting" mit Regex immer Murks.

## 2. Die Werkzeugfamilie, sauber getrennt

In deinen Notizen verschwimmen diese Kategorien. Sie machen völlig verschiedene Dinge.

**Formatter** — ändert ausschließlich das Layout, niemals die Bedeutung. Einrückung, Zeilenumbrüche, Anführungszeichen, Klammersetzung. `black`, `ruff format`, `prettier`, `gofmt`.

Das entscheidende Designmerkmal guter Formatter ist, dass sie **kaum konfigurierbar** sind. `black` nennt sich selbst „uncompromising", `gofmt` hat gar keine Optionen. Das ist Absicht: Der Wert liegt nicht darin, dass die eine Formatierung besser wäre, sondern darin, dass die Diskussion darüber aufhört. Zweiter, praktischer Nutzen: Diffs enthalten kein Formatierungsrauschen mehr. Wenn im Diff nur die Zeilen stehen, deren Bedeutung sich geändert hat, wird Review um ein Vielfaches schneller — und das gilt für AI-generierten Code doppelt.

**Linter** — sucht verdächtige Muster. `ruff`, `eslint`, `pylint`, `clippy`.

Linter-Regeln zerfallen in zwei Gruppen, die man auseinanderhalten sollte:
- *Korrektheit*: ungenutzte Variable (oft ein Tippfehler), unerreichbarer Code, `except:` ohne Typ (verschluckt auch `KeyboardInterrupt`), veränderliche Default-Argumente in Python (`def f(x=[])` — der berühmte Fußschuss), `==` statt `===` in JavaScript.
- *Konvention*: Namensschemata, Importreihenfolge, Komplexitätsgrenzen.

Die erste Gruppe findet echte Bugs, die zweite erzeugt Einheitlichkeit. Beide zusammen ergeben den Eindruck, Linting sei Stilpolizei — es ist überwiegend Fehlerprävention.

Erwähnenswert: `ruff` hat in den letzten Jahren das Python-Tooling konsolidiert. Es ersetzt `flake8`, `isort`, `pyupgrade` und `black` in einem einzigen, in Rust geschriebenen Werkzeug, das typischerweise zwei Größenordnungen schneller läuft als die Vorgänger. Diese Geschwindigkeit ist kein Luxus, sondern die Voraussetzung dafür, dass man das Ding nach *jeder* Dateiänderung laufen lassen kann. Im JS-Ökosystem verfolgt `biome` denselben Ansatz.

**Type Checker** — prüft die Konsistenz von Datentypen über Funktions-, Datei- und Modulgrenzen hinweg. `mypy`, `pyright`, `tsc`. Dazu gleich mehr, weil das die größte Lücke ist.

**Security Linter / SAST** — ein Linter mit Sicherheitsregeln. `bandit` (Python), `semgrep` (sprachübergreifend), `CodeQL` (GitHub). Findet hartkodierte Passwörter, SQL-Strings mit String-Konkatenation, unsichere Deserialisierung, schwache Krypto. Die anspruchsvolleren machen **Taint Analysis**: Sie verfolgen, ob Daten aus einer unvertrauenswürdigen Quelle (HTTP-Parameter) ohne Bereinigung in eine gefährliche Senke (SQL-Query, `eval`) fließen.

**Secret Scanner** — `gitleaks`, `detect-secrets`, `trufflehog`. Sucht nach API-Keys, Tokens, privaten Schlüsseln. Erinnere dich an Modul 1: Einmal committet ist für immer in der Historie. Deshalb gehört dieser Check *vor* den Commit, nicht danach.

**Dependency Scanner (SCA)** — `pip-audit`, `npm audit`, Dependabot, Renovate. Streng genommen keine Codeanalyse, sondern Software Composition Analysis: Es prüft deine Abhängigkeiten gegen Schwachstellendatenbanken. Gehört hierher, weil in modernen Projekten der weitaus meiste Code fremder Code ist.

## 3. Type Checking — der Teil, der bei dir am meisten bringt

Zwei Achsen, die ständig verwechselt werden:

- **statisch vs. dynamisch typisiert**: Wann wird geprüft — vor dem Ausführen oder währenddessen?
- **stark vs. schwach typisiert**: Wie bereitwillig konvertiert die Sprache Typen stillschweigend?

Python ist *dynamisch und stark* typisiert (`"1" + 1` ist ein Fehler zur Laufzeit). JavaScript ist *dynamisch und schwach* (`"1" + 1` ergibt `"11"` — und ist damit die Quelle unendlicher Bugs).

**Wie Python-Typprüfung wirklich funktioniert** — das ist der Punkt, der die meisten überrascht: Type Hints sind zur Laufzeit **wirkungslos**. Der Interpreter liest `def add(a: int, b: int) -> int:` und ignoriert die Annotationen vollständig. Du kannst `add("a", "b")` aufrufen, und Python beschwert sich nicht. Die Annotation existiert ausschließlich für Werkzeuge: für `mypy`, für deine IDE, für andere Menschen.

Das nennt sich **Gradual Typing**: Du kannst Typen schrittweise einführen, Modul für Modul, Funktion für Funktion. Nicht annotierter Code bleibt einfach ungeprüft. Das macht die Einführung in bestehenden Projekten überhaupt erst machbar.

TypeScript funktioniert nach demselben Prinzip: `tsc` prüft und wirft die Typen dann weg — was rausfällt, ist reines JavaScript. Der Fluchtweg heißt dort `any` und deaktiviert die Prüfung für den betroffenen Wert vollständig; `unknown` ist die sichere Alternative, weil sie eine explizite Prüfung erzwingt, bevor du den Wert benutzen darfst.

**Was ein Typechecker nicht findet:** Logik. Das hier ist vollständig typkorrekt und vollständig falsch:

```python
def add(a: int, b: int) -> int:
    return a - b
```

Deshalb brauchst du trotzdem Tests. Typen sichern die *Form* zu, nicht die *Bedeutung*.

**Warum es sich bei einer dynamischen Sprache trotzdem lohnt** — drei Gründe, der dritte ist für dich der wichtigste:

1. Eine ganze Fehlerklasse verschwindet vor der Laufzeit: `None`, wo ein Objekt erwartet wird; ein Dict, wo eine Liste erwartet wird; ein umbenanntes Feld, das an vier Stellen noch alt heißt.
2. Typen sind Dokumentation, die nicht veralten kann — weil sie geprüft wird. Ein Docstring, der lügt, wird nie bemerkt; eine Signatur, die lügt, bricht den Build.
3. **Typen sind maschinenlesbarer Kontext für Agenten.** Ein Modell, das die Signatur einer Funktion sieht, muss nicht raten, was `config` enthält. Ein Modell, das nur `def process(data):` sieht, halluziniert die Struktur — und liegt oft daneben. Type Hints sind, wenn man so will, Context Engineering, bevor es den Begriff gab. Das ist einer der wenigen Fälle, in denen sich eine Investition in klassisches Engineering direkt und messbar in besserer AI-Ausgabe niederschlägt.

Zur Einführung in bestehendem Code: modulweise anziehen, nicht global `--strict` einschalten. In `mypy` etwa `disallow_untyped_defs` erst für die neuen Module, dann nach und nach ausweiten. Ein `# type: ignore` ist erlaubt, aber nur mit Fehlercode und Begründung — `# type: ignore[arg-type]  # lib stubs sind falsch, siehe issue #412`.

## 4. Konfiguration gehört ins Repo

Ein Prinzip, das banal klingt und häufig verletzt wird: **Sämtliche Werkzeugkonfiguration liegt versioniert im Repository**, nicht in den Einstellungen deiner IDE.

In Python bündelt `pyproject.toml` inzwischen fast alles — Projekt-Metadaten, Abhängigkeiten, ruff-, mypy- und pytest-Konfiguration in einer Datei. In JavaScript sind es `eslint.config.js`, `tsconfig.json` und `package.json`.

Der Grund ist Reproduzierbarkeit: Wenn deine IDE andere Regeln anwendet als die CI, produzierst du systematisch Änderungen, die lokal grün und in der Pipeline rot sind. Und ein Agent, der eine Datei anfasst, muss dieselben Regeln vorfinden wie du. Dasselbe gilt für Versionen — `ruff` in Version 0.4 lokal und 0.9 in der CI erzeugt genau das „läuft bei mir"-Problem, das die ganze Übung eigentlich abschaffen soll.

## 5. Git-Hooks — die Mechanik

Ein Git-Hook ist schlicht ein ausführbares Skript in `.git/hooks/` mit einem festgelegten Namen. Git ruft es an bestimmten Punkten auf. Beendet es sich mit einem Exit-Code ungleich 0, bricht Git die Operation ab.

Die naive Umsetzung scheitert an einem Detail: **`.git/hooks/` wird nicht versioniert und nicht mitgeklont.** Dein Hook existiert nur bei dir. Deshalb benutzt niemand rohe Git-Hooks, sondern ein Framework, das die Hooks aus einer versionierten Konfigurationsdatei installiert:

- `pre-commit` (Python-Ökosystem, aber sprachagnostisch) — Konfiguration in `.pre-commit-config.yaml`, jeder Entwickler führt einmal `pre-commit install` aus
- `husky` plus `lint-staged` (JavaScript) — dasselbe Muster

Die relevanten Hooks:
- **`pre-commit`** — läuft vor dem Commit. Hier gehören Formatter, Linter und Secret Scan hin.
- **`commit-msg`** — bekommt die Commit-Message und kann sie ablehnen. Damit lassen sich Conventional Commits erzwingen (`commitlint`).
- **`pre-push`** — läuft vor dem Push. Hier passen langsamere Checks hin: Typecheck, schnelle Unit-Tests.

**Das Latenzbudget ist die entscheidende Designfrage.** Ein Pre-Commit-Hook, der 40 Sekunden braucht, wird nicht dazu führen, dass Leute besseren Code schreiben. Er führt dazu, dass alle `git commit --no-verify` lernen. Deshalb prüfen gute Hooks **nur die geänderten Dateien**, nicht das ganze Repo — genau das macht `lint-staged`, und `pre-commit` tut es standardmäßig.

Und ein Punkt, der oft missverstanden wird: **Hooks sind eine Bequemlichkeit, kein Sicherheitsmechanismus.** Sie laufen auf der Maschine des Entwicklers und lassen sich mit einem Flag abschalten. Die verbindliche Durchsetzung passiert ausschließlich in der CI mit Branch Protection (Modul 5). Der Hook ist dazu da, dir den Fehler in fünf Sekunden statt in fünf Minuten zu zeigen — nicht dazu, dich zu zwingen.

## 6. Agent-Hooks — die neue Kategorie

Hier wird dein Zitat konkret. Ein Agent-Hook funktioniert nach demselben Prinzip wie ein Git-Hook, aber der Auslöser ist kein Git-Ereignis, sondern ein *Agenten*-Ereignis: nach jeder Dateiänderung, vor einem Werkzeugaufruf, am Ende einer Antwort.

Claude Code kennt dafür unter anderem `PreToolUse`, `PostToolUse` und `Stop` — konfiguriert in `settings.json`. Ein `PostToolUse`-Hook auf dem Edit-Werkzeug ist genau die Konstruktion aus deinen Notizen: Der Agent schreibt eine Datei, sofort läuft `ruff check --fix` und `mypy` darauf, und die Ausgabe geht zurück in seinen Kontext.

*(Die konkrete Pfadangabe `.agy/hooks` aus deinem Dokument kann ich nicht bestätigen — solche Details ändern sich schnell und AI-generierte Notizen sind da nicht immer zuverlässig. Wenn wir in Modul 9 das Harness bauen, prüfen wir das gegen die aktuelle Antigravity-Dokumentation.)*

Der Mechanismus, um den es geht:Bei Exit-Code 0 passiert schlicht nichts und der Agent arbeitet weiter — der interessante Zweig ist der andere.

Warum das so viel bewirkt: Der Rückkanal ist der **Terminal-Output**. Ein Agent ist ein System, das Text liest und Text schreibt. Eine Fehlermeldung von `mypy` ist präzise, maschinenlesbar und enthält Datei, Zeile und Ursache — für ein Modell ist das ein besseres Signal als fast jede menschliche Beschreibung. Ohne diesen Kanal behauptet der Agent „fertig", weil er keinen Anlass hat, etwas anderes zu behaupten.

Und die Ökonomie: Derselbe Tippfehler kostet im Agent-Loop 20 Sekunden Modellzeit. In der CI-Pipeline kostet er fünf Minuten Wartezeit *plus* einen Kontextwechsel bei dir — und der Kontextwechsel ist das Teure. Das ist Shift Left aus Modul 0, nur auf der Sekundenskala.

## 7. Exit-Codes — die gemeinsame Sprache

Diese ganze Architektur hängt an einer uralten Unix-Konvention, die niemand erklärt, weil sie alle für selbstverständlich halten: **Ein Prozess signalisiert Erfolg mit Exit-Code 0 und Misserfolg mit allem anderen.**

Das ist die Sprache, in der Hooks, Pipelines und Agenten miteinander reden. `ruff check` gibt 0 zurück, wenn nichts zu beanstanden ist. `pytest` gibt 0 zurück, wenn alle Tests grün sind. `mypy` gibt 0 zurück, wenn die Typen stimmen.

Deshalb funktioniert die Beispielregel aus deinen Notizen — `make lint && make test` — mechanisch: `&&` in der Shell führt den zweiten Befehl nur aus, wenn der erste 0 zurückgibt, und der Gesamtausdruck gibt selbst 0 zurück, wenn beide es taten. Ein einziger Ausdruck, der die Gesamtaussage „alles in Ordnung" transportiert. (Mit `;` statt `&&` würden beide immer laufen und der Exit-Code des letzten Befehls gewinnen — ein klassischer stiller Fehler in Skripten.)

Wenn du in Modul 3 die Formulierung „der Task gilt erst als DONE, wenn Exit-Code 0 zurückkommt" wiedersiehst, weißt du jetzt, dass das keine Metapher ist.

## 8. Die Reibungsleiter: wo läuft welcher Check?

Das ist die praktische Entwurfsentscheidung. Jede Station hat ein anderes Latenzbudget und eine andere Verbindlichkeit:

| Station | Budget | Was dort läuft | Durchsetzung |
|---|---|---|---|
| Editor (LSP) | Millisekunden | Formatter beim Speichern, Linter und Typechecker inkrementell | keine |
| Agent-Hook | 1–10 s | Formatter, Linter, Typecheck auf die geänderten Dateien | keine |
| Pre-Commit | unter 5 s | Formatter, Linter, Secret Scan — nur staged files | umgehbar |
| Pre-Push | unter 30 s | Typecheck über das Projekt, schnelle Unit-Tests | umgehbar |
| CI | Minuten | alles, ganzes Repo, mehrere Versionen, Security-Scans | verbindlich |

Ein Check gehört auf die früheste Station, deren Budget er einhält. Ein Typecheck über 500 Dateien braucht zu lange für den Pre-Commit-Hook, passt aber gut in Pre-Push. Ein vollständiger E2E-Testlauf gehört ausschließlich in die CI.

## 9. Ein Einstiegspunkt für alle

Ein Muster, das mehr bringt, als es aussieht: **Definiere genau einen Befehl pro Prüfung — und alle rufen denselben auf.**

```makefile
lint:
	ruff format --check .
	ruff check .
	mypy src/

test:
	pytest -q
```

Du tippst `make lint`. Der Pre-Commit-Hook ruft `make lint` auf. Die CI ruft `make lint` auf. Und in deiner `AGENTS.md` steht: „Führe vor Abschluss einer Aufgabe immer `make lint && make test` aus."

Der Grund ist Driftvermeidung. Sobald der Agent einen anderen Befehl aufruft als die Pipeline, laufen die beiden Wahrheiten auseinander, und du bekommst rote Builds für Dinge, die lokal grün waren. Ein Kommando, eine Wahrheit. Das ist auch der Kern dessen, was später in deiner Agent-Konfiguration steht: nicht Prosa über Codequalität, sondern der exakte Befehl.

## 10. Einführung in bestehendem Code

Wenn du einen Linter das erste Mal auf ein gewachsenes Projekt loslässt, bekommst du 3000 Fehler und niemand fasst sie an. Vier Strategien:

- **Formatierung separat committen.** Ein einziger großer Commit, der nur formatiert, mit Conventional-Commit-Präfix `style:`. Trage seinen Hash in `.git-blame-ignore-revs` ein, dann überspringt `git blame` ihn — sonst zeigt jede Zeile im Projekt auf diesen einen Commit und Modul 1 war umsonst.
- **Ratchet / Baseline.** Der bestehende Fehlerstand wird eingefroren; neue Verstöße werden abgelehnt. Der Zähler darf nur sinken.
- **Nur geänderte Zeilen prüfen.** Werkzeuge wie `darker` oder `lint-staged` beschränken die Prüfung auf den Diff. Damit wird der Code genau dort besser, wo ohnehin gearbeitet wird.
- **Regeln stufenweise scharfschalten.** Erst die Korrektheitsregeln, dann die Konventionen, dann Strictness beim Typechecker.

## 11. AI-spezifische Punkte

**AI-Code besteht Linter mühelos — das ist eine Falle.** Modelle schreiben stilistisch sauberen Code: konsistente Benennung, ordentliche Docstrings, aufgeräumte Struktur. Ein grüner Linter erzeugt dadurch ein Gefühl von Qualität, das nichts über Korrektheit aussagt. Sei dir bewusst, dass dieses Signal bei AI-Code schwächer ist als bei menschlichem Code, wo Schlampigkeit im Stil oft mit Schlampigkeit im Denken korreliert.

**Typechecker sind bei AI-Code überproportional wertvoll.** Die charakteristischen Fehler von Modellen sind genau die, die ein Typechecker findet: eine Methode, die es auf dem Objekt nicht gibt; eine Bibliotheksfunktion mit der Signatur der Vorgängerversion; ein Rückgabewert, der mal ein Dict und mal ein Objekt ist. Das ist die direkteste Waffe gegen Halluzination im Code.

**Halluzinierte Abhängigkeiten.** Modelle erfinden gelegentlich Paketnamen. Ein `pip install` auf ein erfundenes Paket schlägt normalerweise fehl — es sei denn, jemand hat den Namen vorsorglich registriert und Schadcode hinterlegt. Dieses Angriffsmuster hat einen Namen bekommen („Slopsquatting"), und die Gegenmaßnahme ist banal: Abhängigkeiten gehören gepinnt in eine Lock-Datei, und neue Pakete werden angeschaut, bevor sie hereinkommen.

**Die Kardinalregel.** Der Agent darf niemals Regeln lockern, um grün zu werden — keine Regel deaktivieren, kein `# noqa` streuen, kein `# type: ignore` verteilen, kein `any` einsetzen. Das ist strukturell dasselbe Versagen wie das Anpassen von Tests, das deine Notizen bereits verbieten, und es ist tückischer, weil es harmloser aussieht. Ein Harness, dessen Regeln der Prüfling ändern darf, misst nichts. Diese Verbote gehören explizit in die Agent-Instruktionen.

## 12. Anti-Patterns

- Regel deaktivieren statt Code korrigieren
- `# type: ignore` oder `# noqa` ohne Fehlercode und ohne Begründung
- Formatter und Linter mit widersprüchlichen Regeln, die sich gegenseitig überschreiben
- Ein Pre-Commit-Hook, der so langsam ist, dass alle `--no-verify` benutzen
- Werkzeugversionen lokal und in der CI unterschiedlich
- Linter-Konfiguration in den IDE-Einstellungen statt im Repo
- Verlassen auf Hooks als Durchsetzungsmechanismus statt auf CI

## 13. Glossar

Statische vs. dynamische Analyse · AST · Formatter · Linter · Type Checker · SAST · Taint Analysis · SCA · Secret Scanner · Gradual Typing · statisch/dynamisch vs. stark/schwach typisiert · Type Hint · `any` vs. `unknown` · Strictness · Git Hook · `pre-commit` / `commit-msg` / `pre-push` · `lint-staged` · `--no-verify` · Agent-Hook (`PostToolUse`) · Exit-Code · `&&`-Verkettung · Ratchet/Baseline · `.git-blame-ignore-revs` · Lock-Datei · Slopsquatting

## 14. Übungen

**Ü1 — Die drei Werkzeuge auseinanderhalten.** Schreibe eine Python-Datei, die genau drei Fehler enthält: einen reinen Formatierungsverstoß, einen, den nur der Linter findet (z. B. ein mutables Default-Argument), und einen, den nur der Typechecker findet. Lass alle drei Werkzeuge darauf los und beobachte, welches was meldet — und was keines von ihnen meldet.

**Ü2 — Grundausstattung einrichten.** In deinem Projekt: `ruff` und `mypy` über `pyproject.toml` konfigurieren, ein `Makefile` mit `lint` und `test` anlegen. Prüfe die Exit-Codes von Hand mit `echo $?`.

**Ü3 — Pre-Commit installieren.** `.pre-commit-config.yaml` mit ruff, mypy und `gitleaks`. Dann versuche bewusst, eine Datei mit einem fingierten API-Key zu committen. Miss danach, wie lange dein Hook läuft — über fünf Sekunden ist ein Designfehler.

**Ü4 — Typen als Kontext.** Nimm eine untypisierte Funktion mittlerer Komplexität. Lass einen Agenten sie erweitern. Dann annotiere sie vollständig, setze zurück und lass denselben Agenten dieselbe Erweiterung noch einmal machen. Vergleiche die Ergebnisse — das ist der überzeugendste Beleg für Punkt 3 aus Abschnitt 3.

**Ü5 — Agent-Hook bauen.** Richte in Claude Code einen `PostToolUse`-Hook ein, der nach jeder Dateiänderung `ruff check` auf die geänderte Datei laufen lässt. Gib dem Agenten dann eine Aufgabe, bei der er absehbar einen Lint-Verstoß produziert, und beobachte, ob er ihn ohne dein Zutun korrigiert.

**Ü6 — Das Failure-Pattern provozieren.** Gib einem Agenten eine Aufgabe, die einen echten Typfehler erzeugt, und formuliere den Auftrag bewusst als „mach mypy grün". Schau, ob er den Fehler behebt oder ein `# type: ignore` setzt. Das ist die wichtigste Übung des Moduls, weil sie dir zeigt, wogegen dein Harness abgesichert sein muss.

Weiter zu Modul 3? Das ist der Kern der ganzen Agenda — dort wird aus „der Code läuft" die maschinenprüfbare Aussage „das Verhalten stimmt", und damit die eigentliche Definition of Done für Agenten.
