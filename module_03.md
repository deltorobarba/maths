# Modul 3: Testen — die Sprache, in der du mit Agenten über „fertig" redest

Das ist das wichtigste Modul der Agenda. Alles, was in deinen Notizen unter „Harness" steht, steht und fällt damit. Ein Agent kann nur dann autonom iterieren, wenn es eine maschinenprüfbare Aussage darüber gibt, ob er fertig ist — und die einzige Aussage dieser Art, die es gibt, ist eine grüne Testsuite.

Vorweg zur Terminologie: Du hattest „unit / root tests" erwähnt. „Root Test" ist kein etablierter Begriff — gemeint sind fast sicher Unit-Tests und Smoke- oder E2E-Tests. Nach diesem Modul hast du die Landkarte vollständig.

## 1. Was ein Test wirklich ist

Ein Test ist keine Qualitätsprüfung. Ein Test ist eine **ausführbare Spezifikation**.

Er sagt nicht „der Code ist gut". Er sagt: „Dieses konkrete Verhalten ist zugesichert und darf nicht kaputtgehen." Daraus folgt der wichtigste Maßstab: **Der Wert eines Tests bemisst sich daran, was er verhindert — nicht daran, was er abdeckt.** Ein Test, der niemals fehlschlagen wird, hat den Wert null und Wartungskosten größer null. Er ist ein Nettoverlust.

Die Anatomie ist immer dieselbe, unter zwei Namen:

```
Arrange  →  Act  →  Assert          (klassisch)
Given    →  When →  Then            (BDD-Sprache)
```

Ausgangszustand herstellen, genau eine Sache tun, Erwartung prüfen. Wenn dein Test drei „Act"-Schritte hat, sind es drei Tests.

Und hier ist die eine Regel, an der die meisten Testsuiten sterben: **Ein Test prüft Verhalten, nicht Implementierung.**

```python
# schlecht — prüft, WIE es gemacht wird
def test_discount():
    calc = PriceCalculator()
    calc.apply_discount(100, 0.2)
    assert calc._discount_strategy.was_called   # koppelt an Interna

# gut — prüft, WAS herauskommt
def test_applies_twenty_percent_discount():
    assert PriceCalculator().apply_discount(100, 0.2) == 80
```

Der erste Test bricht bei jedem Refactoring, obwohl das Verhalten unverändert ist. Damit hat er die Sache in ihr Gegenteil verkehrt: Statt Refactoring abzusichern, bestraft er es. Eine Suite voller solcher Tests ist der Grund, warum Teams sagen „wir können den Code nicht mehr anfassen" — nicht *obwohl* sie Tests haben, sondern *weil*.

## 2. Die Testpyramide — und warum „Unit" ein schwammiger Begriff istDie drei Ebenen inhaltlich:

**E2E** durchläuft einen kompletten Nutzerpfad durch das echte, deployte System — Browser klickt, echte Datenbank, echte Nachbarsysteme. **Integration** prüft das Zusammenspiel mehrerer Komponenten: dein Code plus Datenbank, dein Code plus externe API. **Unit** prüft eine Einheit isoliert.

Jetzt der Teil, den kaum jemand ausspricht: **„Unit" ist einer der am schlechtesten definierten Begriffe der Softwareentwicklung.** Ist eine Unit eine Funktion? Eine Klasse? Ein Modul? Ein zusammenhängendes Verhalten? Es gibt zwei Schulen, und sie streiten seit zwanzig Jahren:

- **London School** (mockistisch): Eine Unit ist eine Klasse. Alle Kollaborateure werden durch Test Doubles ersetzt. Ergebnis: maximale Isolation, aber die Tests kennen die Struktur des Codes — und brechen beim Refactoring.
- **Detroit School** (klassisch): Eine Unit ist eine Verhaltenseinheit. Echte Kollaborateure sind erlaubt, solange sie schnell und deterministisch sind. Ersetzt wird nur, was langsam oder unkontrollierbar ist.

Die Detroit-Position hat sich in der Praxis weitgehend durchgesetzt, und sie deckt sich mit der Regel aus Abschnitt 1: Wer Verhalten testet statt Implementierung, mockt sparsam.

Google umgeht die Definitionsfrage elegant, indem es nicht nach Umfang, sondern nach **Ressourcennutzung** klassifiziert — eine operationalisierbare Taxonomie, die ich dir empfehle:

- **Small**: läuft in einem Prozess. Kein Netzwerk, kein Dateisystem, kein `sleep`, keine Nebenläufigkeit. Millisekunden, garantiert deterministisch.
- **Medium**: darf auf `localhost` zugreifen — Datenbank im Container, lokaler Testserver. Sekunden.
- **Large**: alles Weitere. Echte externe Dienste, Netzwerk, ganze Umgebungen.

Der Vorteil: „Kein Netzwerk" ist überprüfbar, „ist das eine Unit?" nicht.

**Und ist die Pyramide noch aktuell?** Bedingt. Zwei Entwicklungen haben sie relativiert. Erstens ist die Testing Trophy (Kent C. Dodds) mit dem Argument angetreten, dass die Integrationsebene das beste Verhältnis von Aufwand zu Vertrauen liefert. Zweitens — und das wiegt schwerer — hat Docker-basiertes Tooling wie Testcontainers Integrationstests drastisch verbilligt: Eine echte Postgres-Instanz für die Testsuite hochzufahren kostet heute Sekunden statt eines Nachmittags Setup.

Die Pyramide ist also kein Gesetz, sondern ein **ökonomisches Argument**: Bau viel von dem, was billig, schnell und präzise ist, und wenig von dem, was teuer, langsam und fragil ist. Wenn sich die Kosten verschieben, verschiebt sich die Form.

## 3. Die Typen jenseits der Pyramide

**Smoke Test** — der Begriff kommt aus der Elektronik: Neue Platine einschalten und schauen, ob sie raucht. In Software: der Minimalcheck nach einem Deployment. Antwortet der Health-Endpoint? Steht die DB-Verbindung? Lädt die Startseite? Fünf Tests, fünf Sekunden. Er beweist nichts über Korrektheit — er beweist, dass es überhaupt Sinn hat, weiterzuschauen. In Modul 6 ist er das erste Gate im Canary.

**Regressionstest** — kein eigener Typ, sondern eine *Rolle*. Jeder Test, der geschrieben wurde, weil ein Bug aufgetreten ist, ist ein Regressionstest. Die dazugehörige Disziplin: Bei jedem Produktionsfehler zuerst einen fehlschlagenden Test schreiben, der ihn reproduziert, dann fixen. Damit wächst deine Suite entlang der Stellen, an denen dein System tatsächlich brüchig ist — was ein weit besseres Auswahlkriterium ist als jede Coverage-Metrik.

**Contract Test** — zwischen Services. Statt beide Systeme gemeinsam E2E zu testen, prüft jede Seite einzeln gegen einen vereinbarten Vertrag: Der Konsument testet gegen einen Mock, der dem Vertrag entspricht; der Anbieter testet, dass er den Vertrag erfüllt. Damit lässt sich Integration prüfen, ohne alles gleichzeitig hochzufahren.

**Property-Based Testing** — der unterschätzteste Ansatz überhaupt. Statt Beispiele anzugeben, formulierst du **Invarianten**, und das Framework erzeugt hunderte Eingaben, um sie zu widerlegen:

```python
@given(st.lists(st.integers()))
def test_sort_is_idempotent(xs):
    assert sorted(sorted(xs)) == sorted(xs)
```

Bibliotheken: `hypothesis` (Python), `fast-check` (JS). Bei einem Fehlschlag *schrumpfen* sie das Gegenbeispiel automatisch auf den minimalen Fall. Das ist genau die Art von Test, die Randfälle findet, an die weder du noch ein Modell gedacht hätten — leere Liste, Unicode-Sonderfall, Ganzzahlüberlauf.

**Snapshot- und Approval-Tests** — der Test speichert die Ausgabe beim ersten Lauf und vergleicht künftig dagegen. Praktisch für UI und komplexe Strukturen, aber mit einer eingebauten Falle: Wenn der Test rot wird, ist der bequeme Weg „Snapshot aktualisieren". Nach dreimal Aktualisieren ohne Hinsehen ist der Test wertlos. Für AI-generierten Code besonders gefährlich, weil die Ausgabe immer plausibel aussieht.

**Characterization Test** (auch Golden Test) — bei Legacy-Code, dessen Verhalten niemand mehr kennt: Erst festschreiben, *was* das System heute tut (inklusive der Bugs), dann refactoren. Der Test sichert nicht Korrektheit, sondern Unverändertheit.

## 4. Test Doubles — sauber getrennt

Alle sagen „Mock" zu allem. Die eigentliche Taxonomie (nach Gerard Meszaros) hat fünf Einträge, und die Unterscheidung ist praktisch relevant:

| Typ | Verhalten |
|---|---|
| **Dummy** | Wird nur übergeben, um eine Signatur zu erfüllen. Nie benutzt. |
| **Stub** | Liefert vorgefertigte Antworten. Prüft nichts. |
| **Spy** | Stub, der zusätzlich protokolliert, wie er aufgerufen wurde. |
| **Mock** | Erwartet bestimmte Aufrufe im Voraus und schlägt fehl, wenn sie ausbleiben. |
| **Fake** | Funktionierende, aber vereinfachte Implementierung — z. B. eine In-Memory-Datenbank. |

Warum die Unterscheidung zählt: Ein **Mock** koppelt den Test an die Interaktionsstruktur des Codes. Er prüft „Funktion X wurde mit diesen Argumenten aufgerufen" — also *Implementierung*, nicht Verhalten. Damit bist du wieder bei dem Problem aus Abschnitt 1. Ein **Fake** dagegen koppelt an gar nichts.

Die Faustregel, die dich durch fast alle Fälle trägt:

> **Ersetze, was du nicht kontrollierst. Ersetze nicht, was dir gehört.**

Nicht kontrollierbar: externe APIs, die Systemzeit, Zufallszahlen, das Netzwerk, Zahlungsanbieter. Deins: deine eigenen Klassen. Wenn du deine eigenen Module gegeneinander mocken musst, ist das meist ein Signal für zu enge Kopplung — und damit eigentlich ein Designproblem (Modul 4), kein Testproblem.

Zeit und Zufall verdienen eine Sonderbemerkung, weil sie die häufigste Ursache für unzuverlässige Tests sind. Injiziere sie als Abhängigkeit (`clock`, `rng`) statt `datetime.now()` und `random.random()` direkt aufzurufen. Dann sind sie im Test einfach kontrollierbar — und dein Code wird nebenbei besser.

## 5. Was einen guten Test ausmacht

- **Deterministisch.** Gleiches Ergebnis bei jedem Lauf, in jeder Umgebung, in jeder Zeitzone.
- **Isoliert.** Reihenfolge egal, kein geteilter Zustand zwischen Tests. Ein Test, der nur läuft, wenn ein anderer vorher lief, ist kaputt.
- **Schnell.** Siehe die Reibungsleiter aus Modul 2 — Geschwindigkeit ist kein Komfort, sondern bestimmt, wie oft der Test überhaupt läuft.
- **Diagnostisch im Fehlerfall.** Der Fehlertext sollte sagen, *was* falsch war, nicht nur *dass* etwas falsch war. `assert result == expected` mit aussagekräftigen Werten schlägt `assert is_valid` um Längen.
- **Ein Grund zu scheitern.** Ein Test mit acht Assertions über fünf verschiedene Dinge sagt dir bei Rot nicht, was kaputt ist.
- **Der Name ist die Zusicherung.** `test_rejects_login_after_three_failed_attempts` — nicht `test_login_2`.

## 6. Flaky Tests — das größte Praxisproblem

Ein **flaky test** ist ein Test, der bei unverändertem Code mal grün und mal rot ist. Typische Ursachen: Abhängigkeit von Timing oder `sleep`, Reihenfolgeabhängigkeit, geteilter Zustand zwischen Tests, echte Netzwerkaufrufe, ungeseeder Zufall, Zeitzonen und Sommerzeit, Nebenläufigkeit.

Der Schaden ist größer, als er aussieht, und er ist **sozial, nicht technisch**. Sobald es normal ist, dass Builds „einfach nochmal laufen" müssen, hat ein roter Build keine Bedeutung mehr. Und dann ist nicht ein Test kaputt, sondern die gesamte Suite als Signal — inklusive aller Tests, die echte Fehler finden würden. Die Retry-Funktion in CI-Systemen ist eine der gefährlichsten Bequemlichkeiten überhaupt.

Der richtige Umgang: flaky Test sofort in Quarantäne (aus dem blockierenden Lauf nehmen, aber mit Ticket und Frist), Ursache beheben, zurückholen. Nicht: retryen und vergessen.

Für Agenten ist das kritisch. Ein Agent, der auf einen flaky Test trifft, sieht einen roten Test bei korrektem Code — und wird ihn „reparieren". Also: die Assertion abschwächen, ein `sleep` verlängern, den Test skippen. Er hat aus seiner Sicht die Aufgabe gelöst.

## 7. Coverage — und warum 100 % ein Anti-Ziel ist

**Line Coverage** misst, welche Zeilen ausgeführt wurden. **Branch Coverage** misst, welche Verzweigungen in beide Richtungen durchlaufen wurden — das ist das aussagekräftigere Maß.

Der entscheidende Punkt: **Coverage misst Ausführung, nicht Prüfung.** Dieser Test erzeugt 100 % Coverage und sichert nichts zu:

```python
def test_process():
    process(sample_input)   # keine Assertion
```

Damit ist Coverage in genau eine Richtung nützlich: Sie sagt dir zuverlässig, was **nicht** getestet ist. Sie sagt dir nichts darüber, ob das Getestete gut getestet ist. Sobald sie zum Ziel wird, greift Goodhart's Law — Teams mit einer 90-%-Vorgabe produzieren zuverlässig assertion-freie Tests für Getter und Setter.

Das ehrlichere Maß heißt **Mutation Testing**. Das Werkzeug verändert deinen Produktivcode systematisch — dreht ein `>` in ein `>=`, ersetzt `+` durch `-`, entfernt einen Funktionsaufruf — und prüft, ob deine Tests das bemerken. Ein überlebender Mutant ist eine Codestelle, für die du zwar Coverage hast, aber keine Zusicherung. Werkzeuge: `mutmut` und `cosmic-ray` (Python), `Stryker` (JS).

Mutation Testing ist langsam und gehört nicht in den Inner Loop — aber es ist im AI-Zeitalter erheblich relevanter geworden. Weil generierte Tests genau die Sorte Test sind, die viel Coverage und wenig Zusicherung produzieren: strukturell korrekt, gut benannt, plausibel — und die Assertion prüft das, was der Code ohnehin tut.

## 8. TDD — ehrlich betrachtet

Der Zyklus: **Red** (Test schreiben, der fehlschlägt) → **Green** (minimal implementieren, bis er grün ist) → **Refactor** (aufräumen, Tests bleiben grün).

Der eigentliche Nutzen von TDD ist nicht Testabdeckung. Er ist **Designdruck**: Du musst das Interface festlegen, bevor du eine Implementierung hast. Und schwer testbarer Code ist fast immer schlecht geschnittener Code — TDD macht Designprobleme früh spürbar, statt sie erst nach zwei Monaten sichtbar werden zu lassen. Wer TDD als Testtechnik verkauft, verkauft es unter Wert.

Wo es nicht passt: Exploration, Prototyping, UI-Feinschliff, alles, wo du das gewünschte Verhalten noch nicht kennst.

**Und jetzt die Wendung, die für dich zählt:** TDD ist im Agentenzeitalter *nützlicher* geworden, nicht weniger. Der Test ist die Spezifikation, die der Agent nicht wegverhandeln kann. Die Aufgabe „mach diesen fehlschlagenden Test grün" ist ungleich präziser als jede Prosabeschreibung — sie hat ein eindeutiges Abbruchkriterium und keinen Interpretationsspielraum.

## 9. Der Kern: Tests als Definition of Done für Agenten

Deine Notizen formulieren es so: *Der Task gilt für den Agenten erst dann als „DONE", wenn der Exit-Code 0 zurückkommt. Bei Fehlern liest das Modell den Stacktrace und unternimmt selbstständig den nächsten Fix-Versuch.*

Damit das funktioniert, muss deine Suite drei Bedingungen erfüllen, und alle drei sind Konsequenzen aus den vorherigen Abschnitten:

1. **Schnell genug für viele Iterationen.** Eine Suite, die 20 Minuten läuft, verträgt keine 30 Agentendurchläufe. Deshalb die Trennung: `pytest -m "not slow"` im Agent-Loop, alles in der CI.
2. **Aussagekräftig genug, dass grün auch korrekt bedeutet.** Sonst optimierst du gegen ein kaputtes Signal — und ein Agent optimiert *sehr* effizient gegen kaputte Signale.
3. **Diagnostisch im Fehlerfall.** Der Stacktrace ist der einzige Kanal, über den der Agent versteht, was schiefging.

**Die vier Ausweichmanöver**, gegen die du absichern musst — in aufsteigender Tücke:

- Den Test an die Implementierung anpassen, statt den Code zu fixen
- Die Assertion abschwächen (`assert x == 42` wird zu `assert x is not None`)
- Den Test skippen oder als `xfail` markieren
- So lange mocken, bis der Test nichts Echtes mehr durchläuft

Alle vier führen zu Exit-Code 0. Alle vier sind aus Sicht des Agenten Lösungen. Deshalb steht in deinen Notizen die Regel *„Ignoriere keine Warnungen und passe keinen Test an, nur damit er grün wird"* — und deshalb reicht diese Regel allein nicht aus.

**Der Zirkelschluss ist das tiefere Problem.** Wenn derselbe Agent Test *und* Implementierung schreibt, stammen beide aus demselben Verständnis der Aufgabe. Ist dieses Verständnis falsch, ist der Test falsch — und bestätigt den Bug, statt ihn zu finden. Ein grüner Test beweist dann nur Selbstkonsistenz. Das ist strukturell dieselbe Schwäche wie ein Modell, das seine eigene Antwort bewertet.

Daraus folgen vier praktische Gegenmaßnahmen:

- **Lies im PR den Test-Diff zuerst.** Nicht den Code. Änderungen an bestehenden Tests sind das wichtigste Warnsignal in AI-gestützter Entwicklung überhaupt.
- **Spezifikation außerhalb des Agenten.** Der wertvollste Workflow ist: Du (oder ein zweites, separat beauftragtes Modell) schreibst den fehlschlagenden Test, der Agent macht ihn grün. Das durchbricht den Zirkel.
- **Tests in separate Commits.** Dann ist im Diff sofort sichtbar, ob Tests mit der Implementierung „mitgewandert" sind.
- **Mutation Testing periodisch**, nicht ständig — als Stichprobe darauf, ob deine Tests noch etwas zusichern.

## 10. Praktisches in pytest

Kompakt, weil du das in den Übungen ohnehin anfasst: **Fixtures** stellen Vorbedingungen bereit und räumen danach auf, mit `scope` von `function` bis `session`; gemeinsame Fixtures leben in `conftest.py` und werden automatisch gefunden. **`@pytest.mark.parametrize`** erzeugt aus einer Tabelle viele Testfälle — der beste Weg, Randfälle systematisch abzudecken, ohne zu kopieren. **Marker** (`@pytest.mark.slow`) erlauben die Trennung von schneller und vollständiger Suite. Für Testdaten skalieren Factories (`factory_boy`) besser als eine wachsende Sammlung von Fixtures. Und für Datenbanken: Jeder Test läuft in einer Transaktion, die am Ende zurückgerollt wird — das ist um Größenordnungen schneller als eine frische Datenbank pro Test.

## 11. Anti-Patterns

- Tests, die Implementierungsdetails prüfen
- Assertion-freie Tests, die nur Coverage erzeugen
- Retry als Umgang mit Flakiness
- Coverage als Zielvorgabe
- Ein Test, der von der Ausführung eines anderen abhängt
- `sleep()` als Synchronisationsmechanismus
- Alles mocken, bis nur noch die Mocks getestet werden
- Snapshot aktualisieren, ohne den Diff zu lesen
- Test und Implementierung im selben Commit vom selben Agenten

## 12. Glossar

Ausführbare Spezifikation · Arrange-Act-Assert / Given-When-Then · Unit / Integration / E2E · Small / Medium / Large · London vs. Detroit School · Testing Trophy · Testcontainers · Smoke Test · Regressionstest · Contract Test · Property-Based Testing · Shrinking · Invariante · Snapshot-/Approval-Test · Characterization Test · Dummy / Stub / Spy / Mock / Fake · Flaky Test · Quarantäne · Line vs. Branch Coverage · Goodhart's Law · Mutation Testing · Red-Green-Refactor · Fixture · Scope · `conftest.py` · `parametrize` · Marker · `xfail`

## 13. Übungen

**Ü1 — Verhalten vs. Implementierung.** Schreibe zu derselben Funktion zwei Tests: einen, der Interna prüft, und einen, der nur die Ausgabe prüft. Refactore dann die Funktion, ohne ihr Verhalten zu ändern. Beobachte, welcher Test bricht.

**Ü2 — Die Pyramide durchspielen.** Für dein Projekt je einen Test pro Ebene: Unit für eine reine Funktion, Integration gegen eine echte Datenbank (Testcontainers), E2E über den HTTP-Endpoint. Miss die Laufzeiten und vergleiche sie mit der Tabelle im Diagramm.

**Ü3 — Flakiness erzeugen und beseitigen.** Schreibe absichtlich drei flaky Tests: einen zeitabhängigen, einen reihenfolgeabhängigen, einen mit ungeseedem Zufall. Lass die Suite zwanzigmal laufen. Repariere dann alle drei durch Injektion statt durch Retry.

**Ü4 — Coverage entzaubern.** Schreibe eine assertion-freie Testsuite, die 100 % Line Coverage erreicht. Lass danach `mutmut` darauf laufen und sieh dir an, wie viele Mutanten überleben.

**Ü5 — Property-Based Testing.** Nimm eine Funktion mit interessanten Randfällen (Parser, Sortierung, Rundung, Währungsberechnung) und formuliere zwei Invarianten in `hypothesis`. Rechne damit, dass ein Fehler gefunden wird, den du nicht erwartet hättest.

**Ü6 — Der Zirkelschluss.** Lass einen Agenten Test *und* Implementierung für eine Funktion mit einer subtilen Anforderung schreiben (z. B. Rundung nach kaufmännischen Regeln bei negativen Beträgen). Prüfe selbst, ob der Test die Anforderung überhaupt richtig verstanden hat. Wiederhole dann mit dem umgekehrten Ablauf: Du schreibst den Test, der Agent implementiert.

**Ü7 — Das Ausweichmanöver provozieren.** Gib einem Agenten einen echten Bug und den Auftrag „mach die Testsuite grün" — bewusst schlecht formuliert. Beobachte, welches der vier Manöver er wählt. Formuliere danach die Regel um, die das verhindert hätte, und teste sie erneut.

Modul 4 wäre als Nächstes dran: Code-Design und Architektur — der Teil, in dem es um die Urteilskraft geht, die Agenten strukturell fehlt.
