Gut — dann Erklärung durchgehend, Übungen gesammelt am Schluss. (Die Checkpoint-Fragen aus Modul 0 lasse ich offen stehen; wir können sie später aufgreifen.)

# Modul 1: Version Control als Denkwerkzeug

Dieses Modul kommt zuerst, weil Git die Grundlage für alles Weitere ist: Ohne saubere Historie funktionieren Code Review, Bisect, Rollback und Agent-Checkpointing nicht. Die meisten Leute lernen Git als eine Sammlung von Befehlen, die sie auswendig können. Das ist der Grund, warum sie in Panik geraten, sobald etwas Ungewohntes passiert. Wir gehen den anderen Weg: erst das Datenmodell, dann folgen die Befehle von selbst.

## 1. Was Git wirklich ist

Git ist kein Backup-System und speichert keine Diffs. Git ist ein **gerichteter azyklischer Graph von Snapshots**.

Ein Commit enthält drei Dinge: einen vollständigen Snapshot des Dateibaums, einen Zeiger auf seinen Eltern-Commit (bei Merges: zwei) und Metadaten (Autor, Zeit, Message). Der Diff, den du in `git diff` siehst, wird zur Laufzeit berechnet — er ist nicht gespeichert.

Daraus folgen zwei Dinge, die das Verhalten von Git erklären:

**Ein Branch ist nur ein Zeiger auf einen Commit.** Wörtlich eine Datei mit einem 40-Zeichen-Hash darin. Deshalb ist Branchen in Git kostenlos und instantan — im Gegensatz zu älteren Systemen wie SVN, wo ein Branch eine Kopie war. Diese Billigkeit ist der Grund, warum sich die gesamte moderne PR-Kultur entwickelt hat.

**Die Historie ist ein Graph, keine Liste.** `git log` zeigt dir eine linearisierte Ansicht, aber darunter liegt ein Graph. Das ist die Ursache dafür, dass Merge und Rebase überhaupt zwei verschiedene Operationen sind.

`HEAD` ist der Zeiger auf den Branch, auf dem du gerade stehst. „Detached HEAD" heißt schlicht: HEAD zeigt direkt auf einen Commit statt auf einen Branch. Kein Fehlerzustand, nur ein Zustand.

## 2. Die drei Bereiche

```
Working Directory  →  Staging Area (Index)  →  Repository
     (Dateien)          (git add)              (git commit)
```

Fast alle Quereinsteiger empfinden die Staging Area als überflüssige Bürokratie. Sie ist das Gegenteil: Sie ist das Werkzeug, mit dem du aus chaotischer Arbeit saubere Commits herausschneidest.

Realität beim Programmieren: Du fixt einen Bug, räumst nebenbei eine Funktion auf und ergänzt eine Fehlermeldung. Drei logisch getrennte Dinge in einem Wust von Änderungen. Mit `git add -p` gehst du hunk-für-hunk durch deine Änderungen und entscheidest einzeln, was in den nächsten Commit gehört. Aus einem Chaos werden drei atomare Commits.

`git add -p` ist der Befehl, der Anfänger von Fortgeschrittenen trennt. Nicht weil er schwer ist, sondern weil seine Existenz voraussetzt, dass du Commits als Artefakte mit Zweck begreifst.

## 3. Der Commit als Behauptung

Ein Commit ist keine Speicherung. Er ist eine Behauptung:

> „Hier ist eine in sich geschlossene, funktionierende Änderung — mit einer Begründung."

Beide Teile sind wichtig.

**In sich geschlossen und funktionierend (Atomarität).** Jeder Commit auf `main` sollte für sich lauffähig und grün sein. Das klingt nach Perfektionismus, ist aber rein praktisch: `git bisect`, `git revert` und `git cherry-pick` funktionieren nur unter dieser Bedingung. Ein Commit „WIP" oder „fixes" zerstört alle drei Werkzeuge auf einen Schlag.

**Mit einer Begründung.** Die Commit-Message beschreibt nicht *was* du geändert hast — das steht im Diff, den kann jeder lesen. Sie beschreibt, **warum**. Das ist die einzige Information, die sonst nirgends existiert und die in sechs Monaten niemand mehr rekonstruieren kann.

Schlecht: `update user service`
Gut: `fix: session timeout on concurrent logins — server rejected the second token because the cache key omitted the device id`

**Conventional Commits** ist die verbreitetste Konvention dafür: `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `test:`, `perf:`. Der Nutzen ist nicht Ästhetik, sondern Maschinenlesbarkeit: Aus dem Präfix lassen sich Changelogs generieren und Versionsnummern nach Semantic Versioning automatisch ableiten (`fix:` → Patch, `feat:` → Minor, `BREAKING CHANGE` → Major).

## 4. Merge vs. Rebase

Der Klassiker, an dem sich Teams streiten. Der Unterschied ist mechanisch simpel.

Ausgangslage: Du hast von `main` bei Commit B abgezweigt und zwei Commits (D, E) gebaut. Währenddessen ist auf `main` Commit C dazugekommen. Jetzt willst du deine Arbeit integrieren.**Merge** erzeugt einen neuen Commit M mit zwei Elternteilen. Die Historie bleibt wahrheitsgetreu — man sieht, dass parallel gearbeitet wurde — ist aber verzweigt und in großen Repos schnell unlesbar.

**Rebase** nimmt deine Commits D und E, wirft sie weg und erzeugt sie neu auf Basis von C. Achte auf die Striche: D' und E' sind **neue Commits mit neuen Hashes**. Der Inhalt ist derselbe, die Identität nicht. Die Historie ist linear und schön — aber sie erzählt eine Geschichte, die so nie passiert ist.

Daraus folgt die einzige harte Regel:

> **Rebase niemals Commits, die schon jemand anderes hat.**

Wenn du Commits neu schreibst, die dein Kollege bereits gezogen hat, hat er Commits, die es nicht mehr gibt — und beim nächsten Pull entsteht ein Zwilling der gesamten Historie. Rebase auf deinem eigenen, noch nicht geteilten Branch: unproblematisch und nützlich. Rebase auf `main`: nie.

In der Praxis lösen die meisten Teams das mit **Squash-Merge**: Der komplette PR wird beim Merge zu einem einzigen Commit auf `main` zusammengefasst. Du bekommst eine lineare `main`-Historie, in der ein Commit exakt einem PR entspricht — und du darfst auf deinem Branch so chaotisch committen, wie du willst. Das ist der pragmatische Standard bei GitHub-basierten Teams. Der Preis: Die Feingliedrigkeit innerhalb des PRs geht verloren, und ein 2000-Zeilen-PR wird zu einem 2000-Zeilen-Commit — was `git bisect` wieder stumpf macht. Also: kleine PRs.

## 5. Branching-Strategien

**GitFlow** — `develop`, `release/*`, `hotfix/*`, `feature/*`, `main`. Entstand 2010 für Software mit expliziten Versionsreleases: Desktop-Anwendungen, On-Prem-Installationen, Dinge, von denen mehrere Versionen gleichzeitig gepflegt werden. Für einen Webservice, der zwanzigmal am Tag deployt wird, ist das durchgehend Overkill. Der Autor selbst hat inzwischen einen Hinweis vorangestellt, dass es für Continuous Delivery nicht das richtige Modell ist.

**Trunk-Based Development** — alle arbeiten gegen `main`. Branches leben Stunden, maximal ein bis zwei Tage. Unfertige Features liegen hinter Feature Flags im Code auf `main`, statt auf einem Branch zu warten.

Der Kernkonflikt dahinter ist **Integrationsschmerz**: Die Wahrscheinlichkeit eines Merge-Konflikts wächst mit der Lebensdauer eines Branches *und* mit der Anzahl paralleler Branches. Zwei Wochen alte Feature-Branches in einem Team von acht Leuten produzieren einen Merge-Tag.

Hier ist eine Begriffsfalle, die dich sofort als fachfremd verraten würde: **Continuous Integration ist ursprünglich eine Praktik, kein Werkzeug.** Es bedeutet wörtlich „laufend integrieren" — mindestens täglich in den Hauptzweig mergen. Der Satz „wir haben CI" im Sinne von „wir haben eine Pipeline" ist eine Verkürzung, die sich eingebürgert hat. Ein Team kann eine perfekte Pipeline haben und trotzdem kein Continuous Integration betreiben, wenn seine Branches drei Wochen alt werden. Wenn du diesen Unterschied kennst, bist du in dieser Frage schon über dem Durchschnitt.

## 6. Der Pull Request

Der PR ist ein soziales Artefakt, kein technisches. Git kennt keine PRs — das ist eine Erfindung von GitHub.

Was einen guten PR ausmacht:

**Klein.** Die Fehlerfindungsrate bei Reviews sinkt ab etwa 200–400 geänderten Zeilen deutlich, und die Review-Latenz steigt überproportional — große PRs bleiben liegen, weil niemand den Kopf dafür frei hat. Ein 1000-Zeilen-PR bekommt „LGTM" und keinen echten Review.

**Eine Sache.** Refactoring und Feature im selben PR sind für den Reviewer nicht trennbar. Trenne sie in zwei PRs.

**Mit Kontext.** Die PR-Beschreibung sollte beantworten: Was ist das Problem? Warum diese Lösung und nicht eine andere? Was sollte der Reviewer besonders anschauen? Wie hast du es verifiziert?

**Selbst gelesen.** Lies deinen eigenen Diff, bevor du ihn schickst. Du findest damit ein Drittel der Anmerkungen selbst — und das ist bei AI-generiertem Code der wichtigste einzelne Arbeitsschritt überhaupt.

## 7. Die Werkzeuge, die deine Historie nutzbar machen

Diese Befehle sind der Grund, warum die Disziplin aus Abschnitt 3 sich auszahlt.

**`git bisect`** — binäre Suche über die Historie. Du sagst „hier war es kaputt, dort war es gut", Git springt in die Mitte, du sagst gut/schlecht, und nach `log₂(n)` Schritten hast du den Commit, der den Fehler eingeführt hat. Bei 1000 Commits sind das zehn Schritte. Mit `git bisect run ./test.sh` läuft es vollautomatisch.

Der Preis: Es funktioniert nur, wenn jeder Commit lauffähig ist. Deshalb die Atomarität.

**`git blame`** — zeigt zeilenweise, welcher Commit sie zuletzt geändert hat. Der Wert liegt in der Kette: Zeile → Commit → Commit-Message → PR → Diskussion. Deshalb sind gute Commit-Messages kein Selbstzweck, sondern der Einstieg in das archivierte Warum.

**`git revert` vs. `git reset`** — der wichtigste Unterschied im Alltag. `revert` erzeugt einen *neuen* Commit, der eine Änderung rückgängig macht; die Historie bleibt intakt. Das ist die sichere Variante und die einzige, die auf geteilten Branches erlaubt ist. `reset` verschiebt den Branch-Zeiger und schreibt Historie um — nur lokal, nur auf ungeteilten Commits.

**`git reflog`** — dein Rettungsanker. Git protokolliert jede Bewegung von HEAD, auch wenn Commits scheinbar verschwunden sind. Ein „verlorener" Commit nach einem falschen `reset` ist über den reflog fast immer wiederherstellbar (Standard: 90 Tage). Merksatz: Git verliert praktisch nie etwas, das einmal committed wurde.

**`git stash`** — Änderungen zwischenparken, ohne zu committen. Nützlich, aber ein Zeichen dafür, dass du zu selten committest.

## 8. Worktrees — der Baustein für parallele Agenten

`git worktree add ../feature-x feature-x` erzeugt ein **zweites Arbeitsverzeichnis** für dasselbe Repository, mit einem eigenen ausgecheckten Branch.

Der Unterschied zu einem zweiten `git clone`: Beide Worktrees teilen sich dieselbe Objektdatenbank. Kein doppelter Speicher, keine Synchronisation zwischen Klonen, alle Branches sofort in beiden sichtbar.

Klassisch war das eine Nischenfunktion — praktisch, wenn man mitten in einem Feature einen Hotfix bauen muss, ohne zu stashen. Mit agentischer Entwicklung ist es zentral geworden: Du willst drei Agenten gleichzeitig an drei Aufgaben arbeiten lassen, ohne dass sie sich gegenseitig die Dateien unter den Füßen wegziehen. Drei Worktrees, drei Branches, drei Agenten, keine Kollision.

## 9. Was nie ins Repo gehört

**Secrets.** Und hier ist die Falle, die viele erst spät verstehen: Ein einmal committetes Secret ist **für immer in der Historie**, auch wenn du es im nächsten Commit löschst. Die Historie ist unveränderlich. Selbst wenn du sie mit `git filter-repo` umschreibst — jeder, der vorher gezogen hat, hat es noch, und bei einem öffentlichen Repo hat es innerhalb von Minuten ein Scraper. Die einzige echte Antwort auf ein geleaktes Secret ist **Rotation**: den Key ungültig machen und neu ausstellen. Das Löschen ist Kosmetik.

Vorbeugend: `.gitignore`, plus automatisiertes Secret Scanning als Pre-Commit-Hook (`gitleaks`, `detect-secrets`). Wir bauen das in Modul 2 ein.

Ebenfalls nicht ins Repo: Build-Artefakte, `node_modules`, virtuelle Umgebungen, große Binärdateien (dafür gibt es Git LFS), IDE-Konfiguration.

## 10. Die AI-Verbindung

Jetzt der Teil, der für dich beruflich zählt.

**Git ist der Undo-Button für Agenten.** Ein Agent, der zwanzig Dateien anfasst, ist nur dann handhabbar, wenn du jederzeit auf einen definierten Zustand zurück kannst. Die Checkpointing-Funktionen der Coding-Tools sind im Kern Git-Operationen mit hübscherer Oberfläche.

**Sauberer Working Tree vor dem Agent-Start.** Das ist die einzige nicht verhandelbare Regel. Startest du einen Agenten mit uncommitteten eigenen Änderungen, kannst du hinterher im Diff nicht mehr unterscheiden, was du und was er geändert hat. Damit ist Review unmöglich und Rollback riskant. Committen oder stashen, dann starten.

**`git diff` ist deine Review-Oberfläche für AI-Output.** Das ist ein echter Verhaltensunterschied. Der Reflex ist, sich die fertigen Dateien anzuschauen — die sehen plausibel aus, der Agent schreibt sauberen Code. Der Diff zeigt dir dagegen, was *sich geändert hat*, und genau dort sitzen die Probleme: eine stillschweigend entfernte Fehlerbehandlung, eine geänderte Default-Einstellung, ein aufgeweichter Test. Lies den Diff, nicht die Datei.

**Trunk-Based passt zu Agenten, GitFlow nicht.** Agenten erzeugen viele kleine Änderungen in kurzer Zeit. Langlebige Branches würden den Integrationsschmerz vervielfachen. Die Kehrseite: Mehr Änderungen auf `main` bedeuten mehr Bedarf an Feature Flags, damit Unfertiges deployt, aber nicht aktiv sein kann (Modul 6).

**Die konkreten Gefahren.** Ein Agent mit Shell-Zugriff, der `git add -A && git commit` ausführt, committet zuverlässig auch alles, was nicht ins Repo gehört — inklusive der `.env`, die du vergessen hast zu ignorieren. Und `git push --force` auf einem geteilten Branch ist eine der wenigen Git-Operationen, die tatsächlich Arbeit vernichtet. Beides gehört auf die Verbotsliste in deiner Agent-Konfiguration und in Branch-Protection-Regeln (Modul 5).

**AI-generierte Commit-Messages** sind gut für das *Was* — das kann ein Modell aus dem Diff ableiten. Sie sind systematisch schlecht für das *Warum*, weil das Warum nicht im Diff steht. Es lebt in deinem Kopf, im Ticket, in der Diskussion. Nutze die Generierung als Rohentwurf und ergänze den Grund selbst.

## 11. Anti-Patterns

- `git commit -m "fix"` — vernichtet den Wert von blame und bisect
- Der Sammel-Commit am Feierabend, der acht unzusammenhängende Änderungen enthält
- Branches, die älter als eine Woche werden
- `git push --force` auf geteilte Branches (statt `--force-with-lease`, das prüft, ob jemand anderes gepusht hat)
- Commit ohne vorher den eigenen Diff gelesen zu haben
- Generierte Dateien im Repo — der Merge-Konflikt darin ist immer sinnlose Arbeit

## 12. Glossar

Commit · Snapshot vs. Diff · Branch als Zeiger · HEAD · Detached HEAD · Working Directory / Staging Area (Index) / Repository · Atomarer Commit · Conventional Commits · Semantic Versioning · Merge Commit · Rebase · Squash-Merge · Fast-Forward · Trunk-Based Development · GitFlow · Integrationsschmerz · Pull Request · `git bisect` · `git blame` · `revert` vs. `reset` · reflog · `git stash` · Worktree · `.gitignore` · Git LFS · Secret Rotation · `--force-with-lease`

## 13. Übungen

Sechs Aufgaben, aufsteigend. Etwa 90 Minuten insgesamt, ein leeres Testrepo genügt.

**Ü1 — Das Datenmodell sichtbar machen.** Neues Repo, drei Commits. Dann `git log --graph --oneline --all` und `git cat-file -p HEAD`. Schau dir an, was in einem Commit-Objekt wirklich steht: Tree-Hash, Parent-Hash, Autor, Message.

**Ü2 — Chaos in atomare Commits schneiden.** Ändere in einer Datei drei logisch unabhängige Dinge gleichzeitig. Zerlege sie danach mit `git add -p` in drei separate Commits mit ordentlichen Messages. Das ist die Kernübung des Moduls.

**Ü3 — Merge und Rebase im Vergleich.** Baue die Ausgangslage aus dem Diagramm nach (Branch bei B, zwei Commits, währenddessen ein Commit auf `main`). Merge sie in einem Branch, rebase sie in einem Duplikat. Vergleiche `git log --graph --oneline` und schau dir die Hashes an.

**Ü4 — Bisect.** Erzeuge fünfzehn Commits, davon einer mit einem Bug (z. B. eine Funktion, die ab da falsch rechnet). Schreib ein Skript, das mit Exit-Code 0/1 antwortet, und lass `git bisect run ./check.sh` den Übeltäter finden. Der Aha-Moment ist, wie wenige Schritte es braucht.

**Ü5 — Kaputtmachen und retten.** Mach einen `git reset --hard HEAD~3` und hol die Commits über `git reflog` zurück. Danach hast du keine Angst mehr vor Git — das ist der eigentliche Zweck der Übung.

**Ü6 — Agent-Szenario.** Committe deinen Stand. Lass Claude Code oder Antigravity eine kleine, mehrdateiige Änderung machen. Lies ausschließlich `git diff` — nicht die Dateien. Notiere, was dir im Diff auffällt, das dir beim Lesen der fertigen Dateien entgangen wäre. Danach: `git worktree add` für einen zweiten Branch, zweiten Agenten parallel starten, beobachten, dass sie sich nicht stören.

Wenn du willst, gehen wir bei Ü2 oder Ü6 gemeinsam durch — die haben den größten Übertrag. Oder wir ziehen direkt weiter zu Modul 2, wo wir Linter, Typechecker und Hooks aufsetzen und dabei das Secret Scanning aus Abschnitt 9 konkret einbauen.
