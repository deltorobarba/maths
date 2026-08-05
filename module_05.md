# Modul 5: CI/CD & Quality Gates

Mit diesem Modul beginnt Phase 2. Bis hierher lief alles auf deiner Maschine und war umgehbar — Hooks lassen sich mit einem Flag abschalten, Tests lokal überspringen. Ab jetzt geht es um Mechanismen, die **verbindlich** sind, weil sie nicht mehr dir gehören.

## 1. Die drei Begriffe, die alle verwechseln

„CI/CD" ist eine Abkürzung, die drei verschiedene Dinge in sich versteckt, und das zweite „C" ist systematisch mehrdeutig.**Continuous Integration** ist — wie in Modul 1 angerissen — zuerst eine *Praktik*: mindestens täglich in den Hauptzweig integrieren, und jede Integration automatisch verifizieren. Das Ziel ist ein Hauptzweig, der jederzeit grün ist. Die Pipeline ist das Werkzeug dafür, nicht die Sache selbst.

**Continuous Delivery** heißt: Jeder grüne Commit auf `main` ist *deploybar*. Alles bis zum Produktionstor ist automatisiert, aber jemand drückt den Knopf. Die Entscheidung, *ob* ausgeliefert wird, ist damit eine Geschäftsentscheidung und keine technische mehr.

**Continuous Deployment** heißt: Jeder grüne Commit geht automatisch in Produktion. Kein Mensch dazwischen.

Und hier die Einordnung, die dich von jemandem unterscheidet, der die Begriffe nur nachplappert: **Continuous Deployment ist nicht „besser" als Continuous Delivery.** Es ist ein anderer Reifegrad mit harten Voraussetzungen. Wer ohne Feature Flags, Canary-Rollout, belastbares Monitoring und automatischen Rollback (alles Modul 6 und 7) auf Continuous Deployment umstellt, hat nicht Reife erreicht, sondern nur das Sicherheitsnetz entfernt. Für viele regulierte Domänen ist der manuelle Freigabeschritt außerdem eine Compliance-Anforderung.

Wenn dich jemand fragt „habt ihr CD?", ist die richtige Rückfrage: *welches*.

## 2. Anatomie einer Pipeline

Die Vokabeln, am Beispiel von GitHub Actions, aber begrifflich überall gleich:

- **Workflow / Pipeline** — die gesamte Definition, versioniert im Repo (`.github/workflows/*.yml`)
- **Trigger / Event** — was sie startet: `push`, `pull_request`, `schedule` (nachts), `workflow_dispatch` (manuell), Tags
- **Job** — läuft auf einem Runner in einer frischen, isolierten Umgebung. Jobs laufen standardmäßig parallel; `needs` erzeugt Abhängigkeiten.
- **Step** — ein einzelner Befehl oder eine wiederverwendbare Action innerhalb eines Jobs
- **Runner** — die Maschine. Gehostet (vom Anbieter, pro Minute abgerechnet) oder selbst betrieben (eigene Hardware, mehr Kontrolle, mehr Verantwortung)
- **Artifact** — Dateien, die einen Job überleben und an spätere Jobs oder an dich weitergereicht werden
- **Cache** — wiederverwendete Zwischenstände zwischen Läufen, meist Dependencies
- **Matrix** — derselbe Job über mehrere Dimensionen: Python 3.11/3.12/3.13 × Linux/macOS
- **Environment** — benannter Zielkontext mit eigenen Secrets und optionalen Freigaberegeln
- **Concurrency** — verhindert, dass fünf Läufe desselben PRs gleichzeitig arbeiten; typischerweise mit „laufenden Vorgänger abbrechen"

Wichtig zum mentalen Modell: **Jeder Job startet in einer frischen Umgebung.** Was ein Job auf die Platte schreibt, ist für den nächsten weg, sofern es nicht explizit als Artefakt übergeben wird. Genau diese Frischheit ist der Wert — sie ist der Grund, warum die Pipeline „läuft bei mir" ausschließt.

## 3. Die Pipeline entwerfen

Zwei Prinzipien, die sich teilweise widersprechen und deshalb abgewogen werden müssen.

**Billig zuerst, fail fast.** Lint braucht zehn Sekunden, E2E braucht acht Minuten. Es wäre absurd, acht Minuten zu warten, um dann an einem Formatierungsfehler zu scheitern. Also: Formatprüfung und Linting zuerst, dann Typecheck, dann Unit-Tests, dann Integration, dann E2E.

**Parallelisieren, wo es geht.** Die relevante Metrik ist die Wartezeit des Menschen, nicht die verbrauchte Rechenzeit. Lint, Typecheck und Security-Scan hängen nicht voneinander ab — die dürfen gleichzeitig laufen.

Die Zielgröße für die PR-Pipeline: **unter zehn Minuten.** Ab etwa einer Viertelstunde fangen Leute an, in der Wartezeit an etwas anderem zu arbeiten, und dann ist der Kontext weg — derselbe Mechanismus wie beim langsamen Build aus Modul 0, nur eine Stufe weiter rechts.

Daraus ergibt sich eine Staffelung, die du bewusst entwerfen solltest:

| Wann | Was | Budget |
|---|---|---|
| Bei jedem PR | Lint, Typecheck, Unit, schnelle Integration, Security-Scan | unter 10 min |
| Nach dem Merge auf `main` (Postsubmit) | vollständige Integration, E2E, Build des Artefakts | bis 30 min |
| Nachts (Nightly) | Matrix über alle Versionen, Last- und Performancetests, Mutation Testing | beliebig |

Der Postsubmit-Lauf ist der Grund, warum es diesen Begriff aus dem Diagramm in Modul 0 überhaupt gibt: Manche Prüfungen sind zu langsam für den PR, aber zu wichtig, um sie wegzulassen.

## 4. Build once, deploy many

Ein Prinzip, das erstaunlich oft verletzt wird und dann erstaunlich teuer wird:

> **Das Artefakt wird genau einmal gebaut und anschließend unverändert durch alle Umgebungen geschoben.**

Nicht: für Staging bauen, dann für Produktion nochmal bauen. Denn dann testest du in Staging etwas anderes, als du in Produktion ausrollst — und genau der Unterschied ist der Fehler, der durchrutscht.

Daraus folgt zwingend: **Konfiguration kommt von außen**, nicht in den Build hinein. Datenbank-URL, API-Keys, Feature-Schalter kommen zur Laufzeit aus Umgebungsvariablen oder einem Secret-Store. Das ist einer der Zwölf-Faktoren („strikte Trennung von Konfiguration und Code") und die Voraussetzung dafür, dass dasselbe Image überall laufen kann.

Praktisch: Das Artefakt wird mit dem Commit-SHA getaggt und ist unveränderlich. `myapp:a3f9c21` — nicht `myapp:latest`. Damit ist jederzeit beantwortbar, welcher Code gerade in Produktion läuft, und ein Rollback ist das Umschalten auf ein älteres, existierendes Image statt eines Neubaus.

## 5. Reproduzierbarkeit

Das Ideal: Derselbe Commit ergibt dasselbe Artefakt — heute, nächsten Monat und nach einem Personalwechsel. Vollständig deterministische Builds sind aufwändig, aber die 90 % kosten wenig:

- **Lock-Dateien** für alle Abhängigkeiten (`uv.lock`, `poetry.lock`, `package-lock.json`), im Repo, und in der CI wird strikt daraus installiert (`npm ci`, nicht `npm install`).
- **Werkzeugversionen festnageln** — dieselbe `ruff`- und `mypy`-Version lokal wie in der CI, sonst hast du genau das Driftproblem aus Modul 2.
- **Basis-Images per Digest**, nicht per Tag. `python:3.12-slim` zeigt heute auf ein anderes Image als vor drei Monaten.
- **`latest` ist immer ein Fehler.** In jedem Kontext.

## 6. Quality Gates: hier wird es verbindlich

Jetzt führen wir den Punkt aus Modul 2 zu Ende. Hooks sind eine Bequemlichkeit; **Branch Protection ist der Mechanismus**, weil er auf dem Server durchgesetzt wird und nicht auf der Maschine des Entwicklers.

Die üblichen Regeln auf `main`:

- **Required status checks** — bestimmte Jobs müssen grün sein, sonst ist der Merge-Button aus
- **Required reviews** — mindestens eine Freigabe, häufig mit „stale reviews verwerfen, wenn neue Commits kommen"
- **CODEOWNERS** — bestimmte Pfade erfordern die Freigabe bestimmter Personen oder Teams
- **Kein direkter Push, kein Force-Push** auf `main`
- **Linear history**, wenn ihr auf Squash-Merge setzt

**Merge Queue** ist das Konzept, das an dieser Stelle am wenigsten bekannt und am meisten wert ist. Das Problem heißt **semantischer Merge-Konflikt**: Zwei PRs sind einzeln grün. PR A benennt eine Funktion um, PR B fügt einen neuen Aufruf der alten Funktion hinzu. Git meldet keinen Konflikt — die Zeilen überschneiden sich nicht. Beide werden gemerged, und `main` ist rot.

Eine Merge Queue löst das, indem sie PRs seriell in der voraussichtlichen Merge-Reihenfolge *gegen den kombinierten Zustand* testet, bevor sie tatsächlich landen. Für ein Team von drei Leuten Overkill. Ab etwa zehn aktiven Entwicklern — oder ab dem Moment, in dem Agenten viele PRs parallel erzeugen — der Unterschied zwischen einem grünen und einem chronisch roten Hauptzweig.

## 7. Security in und um die Pipeline

Zwei Ebenen, und die zweite wird meist vergessen.

**Sicherheit *durch* die Pipeline** — die Werkzeuge aus Modul 2, jetzt verbindlich: SAST (`semgrep`, CodeQL), Dependency Scanning (`pip-audit`, Dependabot), Secret Scanning mit Push Protection, Container-Image-Scanning (`trivy`). Dazu die **SBOM** (Software Bill of Materials) — eine maschinenlesbare Stückliste aller enthaltenen Komponenten. Seit den großen Lieferkettenangriffen zunehmend eine Anforderung, weil sie die Frage „sind wir von dieser Schwachstelle betroffen?" in Minuten statt Wochen beantwortbar macht.

**Sicherheit *der* Pipeline** — der unterschätzte Teil. Deine CI hat Zugriff auf Produktions-Credentials und darf deployen. Sie ist damit ein hochwertiges Ziel, und der Angriffsweg führt über Dritt-Code:

- **Actions per Commit-SHA pinnen**, nicht per Tag. `actions/checkout@v4` ist ein bewegliches Ziel; wer das Tag kontrolliert, kontrolliert deine Pipeline. Genau dieser Weg wurde in der Vergangenheit ausgenutzt.
- **Minimale Berechtigungen.** Standardmäßig `permissions: contents: read`, und mehr nur dort, wo es gebraucht wird.
- **Keine Secrets in Workflows, die auf Fork-PRs laufen.** Der Trigger `pull_request_target` läuft mit Schreibrechten und Secrets im Kontext des Zielrepos — kombiniert man ihn mit dem Auschecken des PR-Codes, führt man fremden Code mit den eigenen Schlüsseln aus. Das ist eine der bekanntesten Fehlkonfigurationen überhaupt.
- **OIDC statt Langzeit-Credentials.** Die Pipeline holt sich pro Lauf ein kurzlebiges Token beim Cloud-Anbieter, statt dass ein dauerhafter Schlüssel in den Secrets liegt.

Wenn du in einem Gespräch mit einem Punkt zeigen willst, dass du über Oberflächenwissen hinaus bist: Das ist er.

## 8. Die Pipeline als Reviewer-Gate für Agenten

Deine Notizen beschreiben es so: *Schlägt die Pipeline fehl, wird das Log automatisch als neuer Prompt an das Worker-Modell zurückgespielt, ohne dass ein menschlicher Entwickler manuell debuggen muss.*

Das ist mechanisch derselbe Mechanismus wie der Agent-Hook aus Modul 2 — Exit-Code plus Fehlertext als Rückkanal —, nur auf der nächsten Reibungsstufe: langsamer, dafür vollständiger und verbindlich. Technisch lässt sich das mit einem Workflow bauen, der bei `failure()` einen Agenten mit dem Joblog anstößt.

Und jetzt die vier Bedingungen, unter denen das eine gute Idee ist und ohne die es teuer wird:

1. **Die Pipeline muss verlässlich sein.** Ein flaky Test in dieser Konstruktion führt dazu, dass ein Agent korrekten Code „repariert", bis er tatsächlich kaputt ist. Modul 3, Abschnitt 6, ist die Voraussetzung, nicht das Beiwerk.
2. **Ein hartes Iterationslimit.** Ohne Abbruchbedingung hast du eine Schleife, die Tokens und CI-Minuten verbrennt und bei jedem Durchlauf mehr Zufallsänderungen anhäuft. Drei Versuche, dann übernimmt ein Mensch.
3. **Der Agent darf die Pipeline-Definition nicht ändern.** Sonst ist der einfachste Weg zu Grün, den Job zu löschen. Das ist exakt dasselbe Ausweichmanöver wie beim Test aus Modul 3, nur eine Ebene höher — und es sieht in einem großen Diff völlig unauffällig aus. `.github/` gehört in die CODEOWNERS und in die Verbotsliste.
4. **Der Agent darf Branch Protection nicht umgehen.** Sein Token braucht dieselben Beschränkungen wie ein Mensch. Ein Bot mit Admin-Rechten macht die ganze Konstruktion sinnlos.

## 9. AI-Code-Review als Gate

PR-Bots (CodeRabbit wird in deinen Notizen genannt, es gibt mehrere) sind stark bei dem, was mechanisch prüfbar ist: Konsistenz mit dem Rest der Codebase, offensichtliche Fehler, vergessene Fehlerbehandlung, fehlende Tests zu neuem Verhalten, bekannte Sicherheitsmuster. Sie sind auch unermüdlich, was ein echter Vorteil gegenüber Menschen um 17 Uhr ist.

Sie sind schwach bei genau dem, was Modul 4 behandelt hat: ob die Abstraktion die richtige Wette ist, ob die Fachlogik dem entspricht, was die Fachabteilung meinte, ob dieses Feature überhaupt gebaut werden sollte.

Und die Gefahr hat einen Namen, den du kennen solltest: **Review-Theater**. Ein PR mit vierzehn AI-Kommentaren *fühlt* sich gründlich geprüft an. Der menschliche Reviewer sieht die Aktivität, schließt daraus auf Sorgfalt und liest oberflächlicher. Das Ergebnis ist weniger echte Prüfung bei mehr sichtbarem Prüfaufwand — ein besonders unangenehmes Muster, weil es sich wie Fortschritt anfühlt.

Die Regel: AI-Review ergänzt, es ersetzt nicht. Und die Reihenfolge aus Modul 3 gilt weiterhin: Der Mensch liest zuerst den Test-Diff.

## 10. Kosten und Skalierung

CI-Minuten sind reales Geld, und hier trifft dich der Effekt aus Modul 0 zum ersten Mal finanziell: Wenn AI-gestützte Entwicklung die Zahl der PRs verfünffacht, verfünffacht sich zunächst auch die CI-Last. Der Engpass wandert nach rechts und zeigt sich zuerst auf der Rechnung.

Die Hebel dagegen:

- **Caching** von Dependencies und Build-Zwischenständen — meist der größte Einzelgewinn
- **Test Impact Analysis** — nur die Tests laufen lassen, die von der Änderung überhaupt berührt sein können (in Monorepos machen das Werkzeuge wie Nx oder Bazel; `pytest-testmon` ist die kleine Variante)
- **Concurrency mit Abbruch**, damit nicht fünf veraltete Läufe desselben PRs weiterlaufen
- **Staffelung** nach Abschnitt 3 statt alles bei jedem PR
- **Merge Queue**, damit nicht jeder PR einzeln gegen einen sich ständig ändernden `main` neu getestet wird

## 11. Anti-Patterns

- Manuelle Schritte innerhalb der Pipeline („dann kopiert jemand die Datei rüber")
- Pro Umgebung neu bauen, statt ein Artefakt weiterzureichen
- `latest` als Tag, ungepinnte Actions, kein Lockfile
- Flaky Tests mit automatischem Retry überdecken
- Secrets, die im Log landen (jede `echo`-Zeile ist eine potenzielle Leckage)
- Ein einziger Riesenjob, statt sinnvoll geteilter, parallelisierbarer Jobs
- Branch Protection, die Administratoren nicht einschließt
- Ein Bot-Token mit mehr Rechten als ein Mensch
- Die Pipeline als reine Formalität behandeln, während rote Builds toleriert werden

## 12. Glossar

Continuous Integration / Delivery / Deployment · Workflow · Trigger · Job · Step · Runner (hosted/self-hosted) · `needs` · Artifact · Cache · Matrix Build · Environment · Concurrency · Postsubmit · Nightly · Build once, deploy many · Zwölf-Faktoren-Konfiguration · Immutable Artifact · SHA-Tagging · Lock-Datei · Digest-Pinning · Branch Protection · Required Status Check · CODEOWNERS · Semantischer Merge-Konflikt · Merge Queue · SAST · SCA · Push Protection · SBOM · OIDC · `pull_request_target` · Test Impact Analysis · Review-Theater

## 13. Übungen

**Ü1 — Grundpipeline bauen.** GitHub-Actions-Workflow für dein Projekt: Lint, Typecheck und Security-Scan parallel, danach Unit-Tests, danach Integration. Miss die Gesamtlaufzeit und die Wartezeit bis zum ersten Fehlschlag.

**Ü2 — Fail fast belegen.** Baue absichtlich einen Formatierungsfehler ein und miss, wie lange die Pipeline bis zum Abbruch braucht. Stelle dann die Reihenfolge um (Tests zuerst) und miss erneut. Der Unterschied ist das Argument für Abschnitt 3.

**Ü3 — Cache einbauen.** Miss die Laufzeit ohne und mit Dependency-Cache. Erwarte einen Faktor zwischen zwei und fünf beim reinen Installationsschritt.

**Ü4 — Gate scharfschalten.** Branch Protection auf `main`: erforderliche Checks, keine direkten Pushes, mindestens eine Freigabe. Versuche danach, direkt auf `main` zu pushen, und lies die Fehlermeldung genau.

**Ü5 — Semantischen Merge-Konflikt erzeugen.** Zwei Branches: einer benennt eine Funktion um, der andere fügt einen Aufruf des alten Namens hinzu. Beide einzeln grün. Merge beide und beobachte, dass `main` rot wird, ohne dass Git je einen Konflikt gemeldet hat.

**Ü6 — Build once, deploy many.** Baue ein Container-Image, tagge es mit dem Commit-SHA, und lass dieselbe Anwendung mit zwei verschiedenen Konfigurationen aus Umgebungsvariablen laufen. Prüfe mit `docker inspect`, dass es wirklich dasselbe Image ist.

**Ü7 — Die Pipeline härten.** Pinne alle Actions auf Commit-SHAs, setze einen expliziten `permissions`-Block auf minimale Rechte, und prüfe, ob irgendwo `pull_request_target` verwendet wird. Notiere, was du geändert hast — das ist dein Einstieg in Modul 10.

**Ü8 — Reviewer-Gate mit Agent.** Baue einen Workflow, der bei einem Fehlschlag das Joblog an einen Agenten übergibt, mit einem Limit von drei Iterationen. Provoziere dann bewusst einen Fall, in dem die einfachste Lösung wäre, den Test oder den Job zu entfernen — und prüfe, ob deine Schutzmaßnahmen aus Abschnitt 8 greifen.

Als Nächstes Modul 6: Release Engineering. Da räumen wir deinen ausdrücklich genannten Blindspot ab — Umgebungen, Blue-Green, Canary, Feature Flags und die Frage, warum Deployment und Release zwei verschiedene Dinge sind.
