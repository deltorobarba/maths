# Modul 8: Context Engineering & Spec-Driven Development

Mit diesem Modul beginnt Phase 3, und hier bringst du bereits Vorwissen mit. Ich überspringe deshalb die Grundlagen und konzentriere mich auf zwei Dinge: die Mechanik, die hinter den Praktiken steckt, und die Verzahnung mit allem aus Phase 1 und 2 — denn dort liegt der Teil, den man in AI-Kreisen selten hört.

## 1. Warum der Begriff gewechselt hat

Der Wechsel von „Prompt Engineering" zu „Context Engineering" ist kein Marketing. Er beschreibt eine echte Verschiebung des Gegenstands.

**Prompt Engineering** heißt: die richtige Formulierung finden. Es ist eine Fähigkeit auf der Ebene einer einzelnen Anfrage.

**Context Engineering** heißt: die richtige *Informationsumgebung* herstellen, in der ein Modell arbeitet. Das ist eine Systemdesign-Aufgabe — und bei agentischen Systemen, die über hunderte Schritte laufen, ist sie die entscheidende.

Das nützlichste mentale Modell: Das Modell ist die CPU, das Kontextfenster ist der Arbeitsspeicher. Du betreibst **Memory Management**. Was lädst du wann? Was hältst du wie lange? Was lagerst du aus? Wer schon einmal mit begrenztem RAM gearbeitet hat, kennt die Denkweise.

## 2. Die Physik des Kontextfensters

Vier Eigenschaften, aus denen sich fast alle praktischen Regeln ableiten.

**Kontext ist endlich und geteilt.** System-Instruktionen, Werkzeugdefinitionen, Projektkontext, Konversationshistorie, gelesene Dateien, Werkzeugausgaben und die Antwort selbst konkurrieren um denselben Platz.

**Die Aufmerksamkeit über den Kontext ist ungleich verteilt.** Das ist der wichtigste empirische Befund und unter dem Namen *Lost in the Middle* gut untersucht:Daraus folgt praktisch: Das Wichtigste gehört an den Anfang oder ans Ende — nicht in die Mitte eines langen Dumps.

**Mehr Kontext ist nicht besser.** Das ist der kontraintuitivste Punkt und der am häufigsten verletzte. Irrelevanter Kontext verschlechtert die Leistung messbar: Er verdünnt das Signal, erzeugt konkurrierende Muster und lädt zu Ablenkung ein. Ein Agent, dem du das halbe Repository in den Kontext kippst, arbeitet schlechter als einer, der die drei relevanten Dateien bekommt. Die Frage ist nicht „was könnte nützlich sein", sondern „was ist notwendig".

**Kontext hat eine Kostenstruktur, die man ausnutzen kann.** Prefix-Caching macht stabile Anfangsteile drastisch billiger und schneller — aber nur, solange sie *unverändert* bleiben. Daraus folgt eine konkrete Designregel: Statisches nach vorn (System-Instruktionen, Werkzeugdefinitionen, Projektkontext), Variables nach hinten. Wer einen Zeitstempel oder eine wechselnde ID in den System-Prompt schreibt, invalidiert bei jeder Anfrage den gesamten Cache. Das ist ein Fehler, der nichts kaputtmacht und nur Geld kostet — und deshalb lange unentdeckt bleibt.

## 3. Was den Kontext tatsächlich füllt

| Bestandteil | Kontrolle | Typische Größe |
|---|---|---|
| System-Instruktionen | vollständig | klein, stabil |
| Werkzeugdefinitionen | vollständig | wächst mit der Anzahl der Tools |
| Projektkontext (`AGENTS.md`) | vollständig | klein bis mittel |
| Konversationshistorie | mittelbar | wächst monoton |
| Gelesene Dateien | mittelbar | mittel bis groß |
| **Werkzeugausgaben** | **kaum** | **unbegrenzt** |

Die letzte Zeile ist der Punkt. Ein `cat` auf eine 4000-Zeilen-Datei, ein Testlauf mit vollem Stacktrace-Wald, ein `grep` mit 800 Treffern — Werkzeugausgaben sind die mit Abstand größte unkurierte Quelle und der häufigste Grund, warum ein Agent nach zwanzig Minuten anfängt, offensichtliche Dinge zu vergessen.

Praktische Konsequenz: Werkzeuge sollten so gebaut sein, dass sie *knappe* Ausgaben liefern. `pytest -q --tb=short` statt voller Ausgabe. `ruff check --output-format=concise`. Ein Skript, das Suchergebnisse auf die relevanten Zeilen reduziert, statt ganze Dateien zurückzugeben. Das ist Werkzeugdesign als Kontextdesign.

Ein zweiter, weniger bekannter Punkt: **Zu viele Werkzeuge verschlechtern die Werkzeugauswahl.** Jedes Tool kostet Kontext, und die Trefferquote bei der Auswahl sinkt, wenn dreißig ähnlich klingende Optionen zur Verfügung stehen. Weniger, klarer abgegrenzte Werkzeuge schlagen mehr.

## 4. Techniken der Kontextverwaltung

**Progressive Disclosure / Just-in-Time.** Der wichtigste Architekturwechsel: nicht alles vorab laden, sondern dem Agenten Werkzeuge geben, mit denen er gezielt nachlädt. Ein Verzeichnisbaum plus ein Suchwerkzeug ist besserer Kontext als das eingebettete Repository — weil der Agent dann liest, was er braucht, und nicht, was du vermutet hast.

**Kompaktierung.** Wenn die Historie zu lang wird, wird sie zusammengefasst und ersetzt. Verlustbehaftet — und *was* dabei erhalten bleibt, ist eine Designentscheidung: Entscheidungen und ihre Begründungen ja, Werkzeugausgaben nein.

**Externalisierung.** Zustand gehört in Dateien, nicht in den Kontext. Ein Plan in `PLAN.md`, Zwischenergebnisse in Dateien, offene Punkte in einer Liste. Das Dateisystem ist das Langzeitgedächtnis — und es hat den angenehmen Nebeneffekt, dass ein Kontextverlust nicht die Arbeit vernichtet.

**Sub-Agenten.** Ein Unteragent mit eigenem Fenster erledigt eine abgegrenzte Aufgabe und gibt nur das Ergebnis zurück. Die 40.000 Tokens Recherche landen nie im Hauptkontext. Mehr dazu in Modul 9.

**Kontext-Hygiene.** Die billigste und meistübersehene Maßnahme: neue Sitzung starten, wenn das Thema wechselt. Ein Kontext, der noch die Reste einer fehlgeschlagenen Debugging-Session enthält, zieht die nächste Aufgabe in dieselbe Richtung.

## 5. Der Brückenschlag: Engineering ist Kontext

Das ist der Abschnitt, der in Phase 3 den Unterschied macht, und ich habe ihn in den vorherigen Modulen mehrfach angekündigt.

> **Gut strukturierter Code ist komprimierter Kontext.**

Konkret, mit Rückverweis auf die Module:

- **Typannotationen (Modul 2)** — eine Signatur ersetzt das Raten über Datenstrukturen. `def process(data)` zwingt zum Lesen der ganzen Datei; `def process(orders: list[Order]) -> Invoice` beantwortet die Frage in einer Zeile.
- **Präzise Namen und Ubiquitous Language (Modul 4)** — ein Modell schließt aus Namen auf Bedeutung. `manager`, `handle`, `result` sind nahezu informationsfrei.
- **Schichtenstruktur (Modul 4)** — konsistente Muster im Gelesenen erzeugen konsistente Muster im Geschriebenen, ohne dass eine Regel formuliert werden muss.
- **Tests (Modul 3)** — die einzige Form von Spezifikation, die sich selbst überprüft.
- **Kleine, kohäsive Module** — passen ins Fenster. Ein 3000-Zeilen-Modul erzwingt entweder Vollständigkeit oder Raten.
- **Konventionen und ein sauberes `AGENTS.md`** — reduzieren die Freiheitsgrade, in denen ein Modell abdriften kann.

Der Umkehrschluss ist der wirklich interessante: Eine Codebase, die für Menschen schwer zu lesen ist, ist auch für Modelle schwer zu bearbeiten — und die Verschlechterung ist bei Modellen ausgeprägter, weil ihnen der Erfahrungskontext fehlt, mit dem ein erfahrener Entwickler Lücken füllt. Wenn dich also jemand fragt, warum du Zeit in Struktur steckst, obwohl doch AI den Code schreibt: **weil die Struktur die Bedingung dafür ist, dass AI den Code gut schreibt.**

## 6. Spec-Driven Development

Die Idee: **Die Spezifikation ist das primäre Artefakt, der Code ist Ableitung.**

Das wurde jahrzehntelang versucht (MDA, UML-Codegenerierung) und ist jedes Mal daran gescheitert, dass die Übersetzung Spezifikation → Code teuer und die Rückübersetzung unmöglich war. Was sich geändert hat, ist genau das: Die Übersetzung ist billig geworden. Damit verschiebt sich das wertvolle Artefakt eine Ebene nach oben.

Die Kette: **Intent → Spec → Plan → Tasks → Implementierung → Verifikation.** Werkzeuge wie GitHub Spec Kit formalisieren das; die Methode ist aber wichtiger als das Werkzeug, und du kannst sie mit drei Markdown-Dateien betreiben.

**Was in eine gute Spec gehört:**

- **Problem und Kontext** — warum überhaupt, welche Randbedingungen
- **Nicht-Ziele** — mindestens so wichtig wie die Ziele. Sie sind das, was den Agenten davon abhält, aus einem Endpoint ein Framework zu machen.
- **Akzeptanzkriterien, testbar formuliert** — „gibt bei ungültiger Eingabe 422 mit Feldnamen zurück", nicht „behandelt Fehler sauber"
- **Constraints** — bestehende Muster, zu verwendende Bibliotheken, Performance-Vorgaben, was *nicht* angefasst werden darf
- **Offene Fragen** — explizit, damit sie beantwortet und nicht geraten werden

Was *nicht* hineingehört: Implementierungsdetails, die du bewusst dem Agenten überlässt. Eine Spec, die den Code in Prosa beschreibt, hat den Vorteil verschenkt.

Und der Rückverweis: **Das ist ein Design Doc aus Modul 4, adressiert an eine Maschine statt an Menschen.** Es ist derselbe Design Loop aus Modul 0 — nur dass der Empfänger sich nicht beschwert, wenn die Spec unklar ist, sondern einfach etwas Plausibles erfindet.

## 7. Der Arbeitszyklus: Explore → Plan → Implement → Verify

Die eine Regel, die den größten Unterschied macht:

> **Plan first, code second.**

Der Grund ist die Ökonomie aus Modul 0, angewandt auf die Zusammenarbeit mit einem Agenten. Ein Missverständnis in der Planungsphase kostet dich 200 Tokens, wenn du es dort korrigierst — und 2000 Zeilen Code plus eine Review-Runde, wenn du es hinterher entdeckst.

**Explore** — erst lesen lassen, nicht schreiben. „Lies diese vier Dateien und erkläre mir, wie der Bestellprozess aktuell funktioniert." Bevor irgendein Code entsteht. Du prüfst dabei nicht den Code, sondern das *Verständnis* — und Missverständnisse sind hier für den Preis eines Absatzes korrigierbar.

**Plan** — der Plan als eigenes, sichtbares Artefakt. Die meisten Werkzeuge haben dafür einen expliziten Modus. Deine Aufgabe hier ist die interessanteste: In den meisten Fällen schlägt ein Modell einen zu komplizierten Ansatz vor — eine Abstraktionsschicht zu viel, ein Pattern, wo eine Funktion gereicht hätte, ein Konfigurationsmechanismus für einen Fall. Das ist Modul 4 in Aktion, und Vereinfachen ist hier deine wertvollste Intervention.

**Implement** — in kleinen Schritten, jeder mit Verifikation. Das Harness aus Modul 2 und 3 läuft dabei die ganze Zeit mit.

**Verify** — nicht nur „Tests grün", sondern die Prüfung gegen die Akzeptanzkriterien aus der Spec. Das ist eine andere Frage als „läuft es".

## 8. Aufgabenzuschnitt

Die richtige Größe für eine Agentenaufgabe ist: **das, was du als Mensch in einem Pull Request noch ernsthaft reviewen kannst.** Nicht das, was der Agent schafft — er schafft mehr, und genau das ist die Falle.

Größere Aufgaben werden überproportional schlechter, aus drei sich verstärkenden Gründen: Die Fehlerwahrscheinlichkeit multipliziert sich über die Schritte, der Kontext läuft während der Arbeit voll, und der Review wird ab einer gewissen Diff-Größe faktisch nicht mehr durchgeführt (Modul 1, Abschnitt 6).

Und ein Zuschnittprinzip, das lohnt: **vertikal statt horizontal schneiden.** Ein dünnes, aber durchgehendes Feature — von der API bis zur Datenbank, mit Test — ist besser als „erst alle Datenmodelle, dann alle Services". Der vertikale Schnitt ist lauffähig, testbar und einzeln zurückrollbar. Der horizontale ist erst am Ende überprüfbar.

## 9. Was Kontext nicht löst

Zum Abschluss die Ehrlichkeit, die in diesem Themenfeld oft fehlt.

Kein Kontext macht ein Modell zum Experten für *deine* Fachdomäne — für die Regeln, die nur in den Köpfen von drei Leuten existieren, für den Sonderfall, den ein Großkunde vor vier Jahren durchgesetzt hat.

Kein Kontext ersetzt Verifikation. Besserer Kontext erhöht die Trefferquote; er macht die Ausgabe nicht überprüfungsfrei.

Und der häufigste Fehler in der Praxis: Wenn das Ergebnis schlecht ist, wird mehr Kontext nachgeschoben. Meistens ist aber nicht die Menge das Problem, sondern die **Klarheit der Aufgabe**. Ein schlecht definiertes Problem wird durch mehr Kontext nur ausführlicher falsch beantwortet. Die richtige Reaktion ist fast immer, die Spec zu schärfen, nicht das Fenster zu füllen.

## 10. Anti-Patterns

- Das ganze Repository in den Kontext kippen
- Wichtiges in der Mitte eines langen Dumps platzieren
- Variables im System-Prompt, das den Cache bei jeder Anfrage invalidiert
- Werkzeuge, die ungefilterte Rohausgaben zurückgeben
- Dreißig Werkzeuge, von denen fünf gebraucht werden
- Über Stunden dieselbe Sitzung weiterführen, obwohl das Thema dreimal gewechselt hat
- Implementieren lassen, ohne den Plan gesehen zu haben
- Eine Spec, die den Code in Prosa beschreibt
- Akzeptanzkriterien ohne Testbarkeit („soll robust sein")
- Aufgaben nach Agentenkapazität schneiden statt nach Review-Kapazität
- Bei schlechtem Ergebnis Kontext nachschieben statt die Aufgabe zu schärfen

## 11. Glossar

Context Engineering vs. Prompt Engineering · Kontextfenster als Arbeitsspeicher · Lost in the Middle · Context Rot · Signalverdünnung · Prefix-Caching · Werkzeugdefinitionen als Kontextkosten · Progressive Disclosure / Just-in-Time-Kontext · Kompaktierung · Externalisierung · Sub-Agent · Kontext-Hygiene · Spec-Driven Development · Nicht-Ziele · Akzeptanzkriterien · Constraints · Plan Mode · Explore–Plan–Implement–Verify · Vertikaler vs. horizontaler Schnitt

## 12. Übungen

**Ü1 — Signalverdünnung messen.** Gib einem Agenten dieselbe Aufgabe zweimal: einmal mit den drei relevanten Dateien, einmal mit fünfzehn Dateien, von denen zwölf irrelevant sind. Vergleiche die Ergebnisse. Das ist der empirische Beleg für Abschnitt 2.

**Ü2 — Positionseffekt testen.** Platziere eine spezifische, ungewöhnliche Regel („Beträge werden immer kaufmännisch gerundet") einmal am Anfang, einmal in der Mitte und einmal am Ende eines langen Kontexts. Prüfe jeweils, ob sie in der Ausgabe berücksichtigt wurde.

**Ü3 — Werkzeugausgaben zähmen.** Miss die Kontextlänge eines Agentenlaufs mit vollständiger Testausgabe und mit `-q --tb=short`. Baue dann ein kleines Wrapper-Skript, das nur die fehlgeschlagenen Tests und deren Kernzeilen zurückgibt.

**Ü4 — Externalisierung.** Lass einen Agenten eine mehrstufige Aufgabe bearbeiten, bei der er den Plan und den Fortschritt in `PLAN.md` pflegt. Beende die Sitzung mittendrin und starte eine neue, die nur die Datei liest. Prüfe, ob er nahtlos weitermachen kann.

**Ü5 — Eine echte Spec schreiben.** Für ein Feature deines Projekts: Problem, Nicht-Ziele, testbare Akzeptanzkriterien, Constraints, offene Fragen. Eine Seite. Gib sie einem Agenten ohne weitere Erklärung und beobachte, wo er nachfragt oder rät — jede solche Stelle ist eine Lücke in deiner Spec.

**Ü6 — Plan first belegen.** Dasselbe Feature zweimal bauen lassen: einmal per direktem Prompt, einmal über Explore → Plan (mit deiner Korrektur) → Implement. Vergleiche Diff-Größe, Anzahl der Nachbesserungen und deinen eigenen Review-Aufwand.

**Ü7 — Den Plan vereinfachen.** Lass dir für eine mittelgroße Aufgabe einen Plan geben und streiche darin alles, was nicht gebraucht wird. Notiere, was du gestrichen hast. Nach fünf Durchläufen hast du eine Liste der typischen Überkomplizierungen — und die geht direkt als Regeln in deine `AGENTS.md`.

**Ü8 — Zuschnitt.** Nimm ein Feature, das du als eine Aufgabe formuliert hättest, und zerlege es in drei vertikale Schnitte, die jeweils lauffähig und einzeln zurückrollbar sind. Bearbeite sie nacheinander und vergleiche den Review-Aufwand mit dem, was ein einzelner großer PR gekostet hätte.

Modul 9 wäre der nächste Schritt: das Agent-Harness selbst bauen. Dort setzt sich alles zusammen — die Hooks aus Modul 2, die Tests aus Modul 3, die Gates aus Modul 5, und die Kontextprinzipien von hier.
