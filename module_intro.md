# Lernagenda: Vom AI Engineer zum AI-gestützten Software Engineer

**Ziel:** Nicht "Tools bedienen können", sondern *wie ein Software Engineer denken* — und AI dort einbauen, wo sie belastbar Wert liefert. Am Ende sollst du eine Codebase, eine Pipeline und ein Agent-Harness so aufsetzen und beurteilen können, dass die Frage nach deinem Studienhintergrund gar nicht erst aufkommt.

---

## Die Leitidee der ganzen Agenda

Deine eigenen Notizen enthalten den wichtigsten Satz schon:

> **„Verifikation schlägt Modellgröße."**

Nicht das Modell garantiert guten Code, sondern das *Harness* drumherum — Linter, Typechecker, Tests, Hooks, CI, Review-Gates, Feature Flags, Monitoring. Genau dieses Harness ist klassisches Software Engineering.

Das heißt: **Die Lücke, die du schließt, ist nicht „AI", sondern „Engineering".** Du lernst AI-Tooling hier nicht isoliert, sondern immer als Schicht *auf* einem soliden Handwerk. Deshalb kommen Module 1–7 vor den AI-Modulen — nicht weil AI unwichtig wäre, sondern weil du sonst nur die 70 % erreichst, die Addy Osmani als „The 70% Problem" beschreibt: schnell zu einem funktionierenden Prototyp, aber die letzten 30 % kosten dich ohne Engineering-Wissen unverhältnismäßig viel.

---

## Wie wir das durchgehen

Pro Modul liefere ich dir:

1. **Mentales Modell** — warum das Konzept überhaupt existiert, welches Problem es löst
2. **Kernbegriffe** — sauber definiert, inkl. der Begriffe, die dir in Doku/Meetings begegnen
3. **Hands-on** — eine konkrete Übung an *deinem* durchlaufenden Projekt
4. **AI-Verbindung** — wo Agenten hier ansetzen und wie das Konzept zum Harness-Baustein wird
5. **Checkpoint** — Fragen, die du beantworten können musst, bevor wir weitergehen. Ich frage dich ab, ehrlich und ohne Schonung.

**Durchlaufendes Projekt:** Ein kleiner, aber echter Service (z. B. eine API mit Datenbank, Auth und Frontend-Anbindung). Der wächst über alle Module mit — Modul 3 bekommt Tests, Modul 5 eine Pipeline, Modul 6 ein Canary-Deployment usw. Am Ende hast du ein Repo, das du herzeigen kannst.

**Annahmen, die ich getroffen habe** (sag Bescheid, wenn falsch — dann passe ich an):
- Python als Primärsprache, TypeScript als Zweitsprache (deine Notizen erwähnen ruff/mypy/pytest *und* eslint/tsc)
- Du willst Tiefe, nicht Überblick — ich erkläre nichts weg, sondern gehe auf Mechanik
- Realistisches Tempo: 1 Modul pro Woche bei ~4–6 h; das macht ca. 3 Monate

---

# PHASE 0 — Landkarte

## Modul 0: Der Entwicklungs-Loop als Denkrahmen

Ohne diese Karte sind alle folgenden Begriffe zusammenhanglose Vokabeln.

**Kernkonzept:** Softwareentwicklung läuft in vier ineinanderliegenden Schleifen, jede mit eigener Zykluszeit:

| Loop | Zykluszeit | Was passiert | Kosten eines Fehlers |
|---|---|---|---|
| **Design Loop** | Stunden–Tage | Observe → Analyze → Design → Deliberate → Plan | Am höchsten |
| **Inner Loop** | Sekunden–Minuten | Think → Code → Build → Test | Am niedrigsten |
| **Submit Loop** | Minuten–Stunden | Lint → PreSubmit → Code Review → Submit | Mittel |
| **Outer Loop** | Stunden–Wochen | Postsubmit → Staging → Canary → Production → Measure | Hoch (echte Nutzer) |

**Die zentrale Ökonomie:** Ein Fehler kostet umso mehr, je später er entdeckt wird. Ein Typfehler im Editor: 2 Sekunden. Derselbe Fehler in Produktion: ein Incident, ein Postmortem, verlorenes Vertrauen. **Alle Engineering-Praktiken, die du lernen wirst, sind Versuche, Fehler nach links zu schieben** ("shift left").

**AI-Verbindung:** Das ist auch die Landkarte dafür, wo AI Wert schafft — der Diagramm-Slide von Addy Osmani markiert genau diese Punkte: Chatbot for SWEs und AI Auto-triage im Design Loop, Code Completion und Automated Testing im Inner Loop, AI-assisted Code Review im Submit Loop, AI Performance Refactorings im Outer Loop.

**Checkpoint:** Du kannst für eine beliebige Praktik (z. B. "Pre-Commit-Hook") sagen, in welchem Loop sie sitzt, welchen Fehler sie nach links schiebt und was ihre Alternative gekostet hätte.

---

# PHASE 1 — Das Handwerk

*Diese Phase ist der eigentliche Hebel. Sie ist unspektakulär und genau deshalb der Unterschied zwischen "kann Prompten" und "kann Engineering".*

## Modul 1: Version Control als Denkwerkzeug

Nicht `git commit`, sondern: Warum Git die Grundlage für alles ist, was danach kommt — inklusive paralleler Agenten.

**Kernbegriffe:** Commit als atomare Änderung · Branch · Merge vs. Rebase · Trunk-Based Development vs. GitFlow · Pull Request · `git bisect` · Worktrees · Conventional Commits · `.gitignore` und warum Secrets nie ins Repo gehören

**Mentales Modell:** Ein Commit ist keine Speicherung, sondern eine *Behauptung*: "Hier ist eine in sich geschlossene, funktionierende Änderung mit einer Begründung." Wer das ernst nimmt, bekommt geschenkt: nachvollziehbare Historie, funktionierendes `git bisect`, reviewbare PRs, saubere Rollbacks.

**Hands-on:** Repo aufsetzen. Eine Feature-Branch, drei atomare Commits, ein PR gegen dich selbst. Dann absichtlich einen Bug einbauen und ihn mit `git bisect` finden.

**AI-Verbindung:** Git ist der Undo-Button für Agenten. Checkpointing, `git worktree` für parallele Agenten in isolierten Arbeitsverzeichnissen, `git diff` als Review-Oberfläche für AI-Output. Ohne saubere Commit-Hygiene kannst du nicht mehr unterscheiden, was du und was der Agent geändert hat.

**Checkpoint:** Du erklärst, warum Trunk-Based Development mit kurzlebigen Branches besser zu AI-Agenten passt als langlebige Feature-Branches.

---

## Modul 2: Der Inner Loop — Statische Analyse & Hooks

**Hier lösen wir direkt das Zitat auf, das du geschickt hast.**

**Kernbegriffe:**
- **Formatter** (prettier, `ruff format`, black) — verändert *nur Layout*, nie Bedeutung. Beendet Stil-Diskussionen per Dekret.
- **Linter** (eslint, ruff, pylint) — sucht *verdächtige Muster*: ungenutzte Variablen, gefährliche Konstrukte, Verstöße gegen Konventionen. Kennt keine Typen.
- **Type Checker** (tsc, mypy, pyright) — prüft *Konsistenz von Datentypen* über Funktions- und Dateigrenzen hinweg. Findet die Klasse Fehler, die dich sonst zur Laufzeit trifft.
- **Static Application Security Testing (SAST)** — Linter mit Sicherheitsregeln (bandit, semgrep, CodeQL)
- **Pre-Commit-Hook** — Skript, das Git *vor* dem Commit ausführt und ihn bei Fehlschlag blockiert
- **Agent-Hook** — dasselbe Prinzip, aber getriggert durch den AI-Agenten nach jeder Dateiänderung

**Das Zitat aus deinen Notizen, entschlüsselt:**
> "Konfiguriert in eurem Agent-Harness (z. B. via .agy/hooks oder Pre-Commit-Skripte), dass nach jeder Code-Änderung automatisch ein Linter (z. B. eslint, ruff) oder Type-Checker (tsc, mypy) läuft."

Übersetzt: *Baue eine Rückkopplungsschleife, die dem Agenten sofort und maschinenlesbar sagt, ob er Mist gebaut hat.* Der Agent bekommt den Fehlertext im Terminal-Output zurück und kann ohne dich korrigieren. Das ist der Unterschied zwischen einem Agenten, der nach 20 Minuten kaputten Code abliefert, und einem, der in 20 Sekunden-Zyklen konvergiert.

*(Anmerkung: Die genaue Pfad-Syntax `.agy/hooks` in deinen Notizen prüfen wir gegen die aktuelle Antigravity-Doku, wenn wir dort sind — solche Details ändern sich schnell und AI-generierte Notizen sind da nicht immer präzise.)*

**Hands-on:** ruff + mypy + pre-commit im Projekt einrichten. Bewusst kaputten Code committen und sehen, wie der Hook blockt. Dann `pre-commit run --all-files` als einheitlichen Einstiegspunkt bauen.

**Checkpoint:** Du erklärst den Unterschied Formatter/Linter/Typechecker an je einem Beispiel und begründest, warum ein Typechecker in einer dynamisch typisierten Sprache trotzdem Sinn ergibt.

---

## Modul 3: Testen — die Sprache, in der du mit Agenten über "fertig" redest

**Das ist das wichtigste Modul der ganzen Agenda.** Ohne Tests hast du keine Definition of Done, die eine Maschine prüfen kann.

*(Zur Klärung: Du hattest "unit / root tests" erwähnt — gemeint sind vermutlich Unit-Tests und Smoke- oder E2E-Tests. Wir gehen die komplette Pyramide durch, dann sitzt die Terminologie.)*

**Die Testpyramide:**

| Ebene | Was wird getestet | Laufzeit | Anzahl | Fragilität |
|---|---|---|---|---|
| **Unit** | Eine Funktion/Klasse isoliert | ms | viele (100e) | niedrig |
| **Integration** | Zusammenspiel mehrerer Komponenten (Code + DB, Code + API) | s | mittel | mittel |
| **End-to-End (E2E)** | Kompletter Nutzerpfad durch das echte System | min | wenige | hoch |
| **Smoke Test** | "Läuft es überhaupt?" — Minimalcheck nach Deployment | s | 1–5 | niedrig |

**Kernbegriffe:** Arrange-Act-Assert · Fixture · Parametrisierung · Test Double (Mock vs. Stub vs. Fake vs. Spy) · Coverage (und warum 100 % ein Anti-Ziel ist) · Flaky Test · Regressionstest · Test-Driven Development (Red-Green-Refactor) · Property-Based Testing · Contract Testing

**Mentales Modell:** Ein Test ist eine *ausführbare Spezifikation*. Er sagt nicht "der Code funktioniert", sondern "dieses konkrete Verhalten ist zugesichert und darf nicht kaputtgehen". Deshalb ist der Wert eines Tests proportional zu dem, was er *verhindert*, nicht zu dem, was er abdeckt.

**AI-Verbindung — der Kern des Ganzen:** Exit-Code 0 ist die Sprache, in der ein Agent "fertig" versteht. Deine Notizen beschreiben das als Test-Driven Loop: Der Task gilt erst als DONE, wenn die lokale Test-Suite grün ist; bei Fehlern liest das Modell den Stacktrace und macht den nächsten Fix-Versuch. Das funktioniert nur, wenn deine Tests (a) schnell genug für viele Iterationen und (b) aussagekräftig genug sind, dass "grün" wirklich "korrekt" bedeutet.

**Die kritische Falle:** Ein Agent, der Tests anpassen darf, um sie grün zu bekommen, hat kein Harness, sondern ein Placebo. Deshalb steht in deinen Notizen die Regel: *"Ignoriere keine Warnungen und passe keinen Test an, nur damit er grün wird."* Wir werden genau diese Failure Modes provozieren und absichern.

**Hands-on:** pytest-Suite für das Projekt. Unit-Tests mit Fixtures und `parametrize`, ein Integrationstest gegen eine Test-DB, ein E2E-Test. Dann: Agent einen Bug fixen lassen und beobachten, ob er die Tests manipuliert.

**Checkpoint:** Du entscheidest für drei gegebene Szenarien, welche Testebene angemessen ist, und begründest, warum mehr E2E-Tests nicht besser sind.

---

## Modul 4: Code-Design & Architektur — hier trennt sich Denken vom Tippen

Der Teil, der sich am wenigsten automatisieren lässt und deshalb am meisten wert ist.

**Kernbegriffe:** Kopplung & Kohäsion · Separation of Concerns · Schichtenarchitektur · Dependency Injection · Interface/Protocol vs. Implementierung · Idempotenz · Pure Function vs. Side Effect · SOLID (pragmatisch, nicht dogmatisch) · YAGNI · Technische Schuld · Refactoring als eigenständige Disziplin · Architecture Decision Record (ADR)

**Mentales Modell:** Architektur ist die Kunst, Entscheidungen so zu treffen, dass spätere Entscheidungen billig bleiben. Die Frage ist nie "ist das schön?", sondern **"was kostet es, das in sechs Monaten zu ändern?"**

**Warum das für dich AI-relevant ist:** Ein Agent optimiert lokal — er löst die Aufgabe, die vor ihm liegt. Er sieht nicht, dass die dritte ähnliche Funktion ein Signal für eine fehlende Abstraktion ist. Diese Urteilskraft ist genau das, was von dir erwartet wird und was niemand automatisiert. Deine Notizen bringen das auf den Punkt: *"Fehler in der Planung sind im Nachhinein extrem teuer."*

**Hands-on:** Bewusst schlecht geschnittenen Code schreiben lassen, dann selbst refactoren — mit Tests als Sicherheitsnetz. Ein ADR schreiben für eine echte Entscheidung im Projekt.

**Checkpoint:** Du liest fremden Code und benennst drei konkrete Design-Probleme mit Begründung, was sie in Zukunft kosten.

---

# PHASE 2 — Vom Repo in die Produktion

## Modul 5: CI/CD & Quality Gates

**Kernbegriffe:** Continuous Integration vs. Continuous Delivery vs. Continuous Deployment (drei verschiedene Dinge!) · Pipeline · Job/Stage/Runner · Build Artifact · Caching · Matrix Build · Branch Protection · Required Checks · Dependency Scanning · Secret Scanning · SBOM · Reproducible Build

**Mentales Modell:** Die Pipeline ist die einzige Instanz, die nicht müde wird, keine Ausnahme macht und niemanden mag. Sie ist deine institutionalisierte Disziplin.

**AI-Verbindung:** Genau das, was deine Notizen als *"CI/CD als automatisches Reviewer-Gate"* beschreiben — schlägt die Pipeline fehl, wird das Log automatisch als neuer Prompt an das Worker-Modell zurückgespielt, ohne dass ein Mensch manuell debuggen muss. Damit wird CI vom Kontrollinstrument zum Feedback-Kanal für Agenten.

**Hands-on:** GitHub Actions Pipeline: Lint → Typecheck → Test → Security Scan → Build. Branch Protection aktivieren. Bewusst einen fehlschlagenden PR erzeugen und das Gate erleben.

**Checkpoint:** Du erklärst den Unterschied Delivery/Deployment und entscheidest begründet, welches für ein gegebenes Produkt angemessen ist.

---

## Modul 6: Release Engineering — **hier kommt Canary**

Dein explizit genannter Blindspot, gründlich abgeräumt.

**Umgebungen:** Local → Dev → Staging → Production. Was jede leistet, warum "es lief auf meiner Maschine" ein Architekturproblem und keine Ausrede ist.

**Deployment-Strategien:**

| Strategie | Mechanik | Rollback | Kosten | Wann |
|---|---|---|---|---|
| **Recreate** | Alt aus, neu an | Redeploy alt | Downtime | Interne Tools |
| **Rolling** | Instanz für Instanz ersetzen | Langsam zurückrollen | Beide Versionen parallel live | Standard |
| **Blue-Green** | Zwei komplette Umgebungen, Traffic umschalten | Sofort (zurückschalten) | Doppelte Infrastruktur | Kritische Systeme |
| **Canary** | 1 % → 5 % → 25 % → 100 % des Traffics, an Metriken gekoppelt | Sofort, meist automatisch | Komplexes Routing + Monitoring | Hohes Risiko, viele Nutzer |

**Canary in der Praxis** — so läuft es wirklich ab:
1. Neue Version wird deployed, bekommt aber zunächst nur ~1 % des Traffics
2. **Metriken laufen automatisiert gegeneinander**: Fehlerrate, Latenz (p50/p95/p99), Business-KPIs — Canary vs. Baseline
3. Bleiben die Werte innerhalb definierter Schwellen für ein Zeitfenster, wird der Anteil erhöht
4. Überschreitet irgendetwas die Schwelle: **automatischer Rollback**, ohne dass ein Mensch reagieren muss
5. Erst bei 100 % gilt das Release als durch

**Der entscheidende Punkt, den viele nicht sehen:** Canary ohne belastbares Monitoring ist Theater. Du kannst nur auf Signale reagieren, die du auch misst. Deshalb hängen Modul 6 und 7 zusammen.

**Weitere Kernbegriffe:** Feature Flag (Deployment ≠ Release — Code kann live sein, ohne aktiv zu sein) · Flag Flip · Progressive Delivery · Kill Switch · Dark Launch · Shadow Traffic · A/B-Test/Experiment · Rollback vs. Roll-Forward · Migrations-Kompatibilität (warum Datenbank-Änderungen die schwierigsten Deployments sind)

**Das erklärt auch den Outer Loop im Diagramm:** Staging → Canary → Production (Experiment → Flag Flip → Release) → Measure. Genau diese Kette.

**Hands-on:** Feature Flag ins Projekt einbauen, ein Feature hinter dem Flag deployen und live umschalten. Rollback durchspielen.

**Checkpoint:** Du entscheidest für drei Szenarien die passende Deployment-Strategie und benennst, welche Metriken dein Canary-Gate prüfen müsste.

---

## Modul 7: Observability & Betrieb

**Kernbegriffe:** Die drei Säulen — Logs, Metrics, Traces · Strukturiertes Logging · Log-Level · Kardinalität · Distributed Tracing (Span, Trace ID) · SLI / SLO / SLA · Error Budget · Alerting vs. Monitoring · On-Call · Incident Response · Blameless Postmortem · MTTR

**Mentales Modell:** *Monitoring* beantwortet Fragen, die du vorher kanntest. *Observability* erlaubt dir, Fragen zu stellen, an die du vorher nicht gedacht hast. Der Unterschied entscheidet, ob du einen Incident in 5 Minuten oder in 5 Stunden verstehst.

**AI-Verbindung:** Produktionssignale sind der Kontext, den Agenten für sinnvolles Debugging brauchen. "Der Endpoint ist langsam" ist wertlos; ein Trace mit Span-Aufschlüsselung ist ein lösbares Problem. Das ist auch das *Resource efficiency* / *AI performance refactorings* am Measure-Knoten im Diagramm.

**Hands-on:** Strukturiertes Logging + einfache Metriken ins Projekt. Ein SLO definieren. Einen künstlichen Incident auslösen und ein Postmortem schreiben.

**Checkpoint:** Du formulierst ein SLO für dein Projekt inkl. Error Budget und begründest, warum 100 % Verfügbarkeit ein falsches Ziel ist.

---

# PHASE 3 — AI systematisch einbauen

*Ab hier hast du das Fundament. Jetzt wird AI nicht mehr aufgesetzt, sondern integriert.*

## Modul 8: Context Engineering & Spec-Driven Development

**Kernkonzept:** Der Sprung von "cleveren Prompts" zu *Informationsumgebungen*. Denk an das Modell als CPU und das Kontextfenster als RAM — du betreibst Memory Management: laden bei Bedarf, häufig Genutztes cachen, Veraltetes aussortieren, Wichtigstes zuerst.

**Kernbegriffe:** SPEC.md / mini-PRD · Plan Mode · Context Rot · Progressive Context Building · Few-Shot · Chain-of-Thought · RAG im Coding-Kontext · Schema-Driven Development · Acceptance Criteria

**Die eine Regel, die am meisten bringt:** *Plan first, code second.* In 9 von 10 Fällen schlägt AI einen zu komplizierten Ansatz vor, den du vereinfachen lassen solltest. Verlange den Plan, bevor Code entsteht.

**Hands-on:** Ein Feature einmal per Vibe-Prompt und einmal spec-driven bauen lassen. Diff, Testbarkeit und Review-Aufwand vergleichen. Das ist ein Aha-Moment.

**Checkpoint:** Du schreibst eine SPEC.md, aus der ein Agent ohne Rückfragen ein testbares Feature bauen kann.

---

## Modul 9: Das Agent-Harness bauen

Jetzt setzt du alles aus Phase 1 & 2 zu dem zusammen, was deine Notizen beschreiben.

**Bausteine:**
- **Projektinstruktionen** (AGENTS.md / CLAUDE.md / GEMINI.md) — Prüf-Kommandos, Build-Regeln, Verifikations-Standard. Der Agent liest das zu Beginn jeder Sitzung.
- **Hooks** — automatische Verifikation nach jeder Änderung (Modul 2)
- **Test-Driven Loop** — Exit-Code 0 als Definition of Done (Modul 3)
- **CI als Reviewer-Gate** — Pipeline-Log als Prompt zurück (Modul 5)
- **Sandboxing & Isolation** — Container, Worktrees, Branches
- **MCP** — Model Context Protocol als Adapter zu Tools, Daten und Aktionen
- **Subagents & Spezialisierung** — Security-, Performance-, Review-Agenten
- **Orchestrator-Worker-Muster** — großes Modell plant, schnelles Modell implementiert
- **Tokenomics & Latenz** — Kosten pro Task statt Kosten pro Token; wann Provisioned Throughput sinnvoll ist
- **Feedback-Loop auf die Instruktionen selbst** — AGENTS.md nach jedem Sprint verfeinern

**Hands-on:** Vollständiges Harness für dein Projekt bauen. Dann ein absichtlich mittelgroßes Feature vollautonom durchlaufen lassen und protokollieren, wo es hakt.

**Checkpoint:** Du kannst dein Harness so beschreiben, dass ein Kollege es in einem fremden Repo nachbaut — und begründen, welcher Baustein welche Fehlerklasse abfängt.

---

## Modul 10: AI-Code kritisch prüfen — Security & Verification Debt

Der Teil, der dich von "produktiv" zu "vertrauenswürdig" bringt.

**Kernbegriffe:** OWASP Top 10 · Injection · Broken Auth · Secrets Management (und warum AI-Code gern Credentials ins Frontend legt) · Supply-Chain-Risiken · Slopsquatting/halluzinierte Dependencies · Lizenz- und IP-Fragen · Prompt Injection in agentischen Workflows · Verification Debt · Der Review-Bias ("sieht gut aus, ist getestet, also passt es")

**Mentales Modell:** Behandle AI-Output wie Code von einem sehr schnellen, sehr selbstbewussten Junior mit lückenhaftem Kontextwissen. Kompetent, aber ohne Verantwortungsgefühl für das Gesamtsystem.

**Der unterschätzte Effekt:** AI verschiebt den Engpass vom *Schreiben* zum *Verifizieren*. Wenn du 5x so viel Code produzierst, aber nicht 5x so viel Review-Kapazität hast, entsteht Verification Debt — und die wächst still.

**Hands-on:** Ein AI-generiertes Feature systematisch auditieren. Prompt Injection im eigenen Agent-Setup ausprobieren. Dependency Scanning einbauen.

**Checkpoint:** Du führst ein Security-Review an AI-generiertem Code durch und findest mindestens zwei nicht-offensichtliche Probleme.

---

## Modul 11: Skalierung im Team & Wirkungsmessung

**Kernbegriffe:** DORA-Metriken (Deployment Frequency, Lead Time for Changes, Change Failure Rate, MTTR) · Golden Path · Platform Engineering · Definition of Done · Code Ownership · Onboarding-Zeit als Qualitätsmetrik

**Die Frage, die dir gestellt werden wird:** "Bringt AI uns eigentlich was?" — Du sollst darauf mit Daten antworten können, nicht mit Gefühl. Achte darauf, dass gestiegene Deployment Frequency bei gleichzeitig gestiegener Change Failure Rate *kein* Erfolg ist.

**Checkpoint:** Du entwirfst ein Messkonzept für den AI-Impact in einem Team, das nicht durch "mehr Zeilen Code" ausgetrickst werden kann.

---

# Capstone

Ein Feature von der Spec bis in Produktion — allein, mit vollem Harness:

SPEC.md → Plan → Agent implementiert → Hooks + Tests grün → PR mit AI-Review → CI-Gates → Staging → Canary mit Metrik-Gate → Feature Flag → Measure → Retro auf AGENTS.md.

Wenn du das eigenständig kannst, ist die Ausgangsfrage beantwortet.

---

## Glossar

Wir führen ein laufendes Glossar mit — jeder Begriff, bei dem du zögerst, kommt rein. Das ist kein Beiwerk: Der schnellste Weg, in einem Team als fachfremd aufzufallen, ist nicht fehlendes Wissen, sondern unsicherer Umgang mit Vokabular.

---

## Wenn du sofort loslegen willst

Die drei Module mit dem größten Sofort-Effekt für dich sind **2 (Hooks & statische Analyse)**, **3 (Testen)** und **6 (Release & Canary)**. Modul 3 ist der eigentliche Hebel — es ist die Voraussetzung dafür, dass das gesamte Harness-Konzept aus deinen Notizen überhaupt funktioniert.

Wir können auch nicht-linear vorgehen, wenn du akut etwas brauchst. Nur Modul 3 solltest du nicht überspringen.
