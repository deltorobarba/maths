# Modul 11: Skalierung im Team & Wirkungsmessung

Die ersten zehn Module handelten von deinem Repo und deiner Arbeit. Dieses handelt von der Organisation — und von der Frage, die dir früher oder später gestellt wird:

> „Bringt uns AI eigentlich etwas?"

Diese Frage wird fast immer schlecht beantwortet, und zwar in beide Richtungen. Die Enthusiasten zeigen Adoptionszahlen. Die Skeptiker zeigen Anekdoten. Beides ist wertlos. Nach diesem Modul kannst du sie mit Zahlen beantworten, die sich nicht austricksen lassen.

## 1. DORA — die vier Metriken

Aus der langjährigen DORA-Forschung (inzwischen bei Google) stammen die vier Kennzahlen, die sich als Industriestandard durchgesetzt haben:

| Metrik | Was sie misst | Dimension |
|---|---|---|
| **Deployment Frequency** | Wie oft geht Code in Produktion? | Durchsatz |
| **Lead Time for Changes** | Wie lange vom Commit bis in Produktion? | Durchsatz |
| **Change Failure Rate** | Welcher Anteil der Deployments erzeugt eine Störung? | Stabilität |
| **Failed Deployment Recovery Time** | Wie schnell ist der Zustand wiederhergestellt? | Stabilität |

Die Struktur ist der Punkt: **zwei Durchsatz-, zwei Stabilitätsmetriken.** Nur als Vierergespann sagen sie etwas aus.

Und der wichtigste empirische Befund der DORA-Arbeit ist kontraintuitiv: **Durchsatz und Stabilität korrelieren positiv, nicht negativ.** Teams, die häufiger deployen, haben nicht mehr Ausfälle, sondern weniger. Der Mechanismus ist der aus Phase 2: Häufiges Deployen erzwingt kleine Änderungen, kleine Änderungen tragen weniger Risiko, und die Fähigkeit, überhaupt häufig zu deployen, ist ein Nebenprodukt genau der Praktiken, die auch Stabilität erzeugen — automatisierte Tests, Trunk-Based Development, schnelles Rollback, Observability.

Der übliche Trade-off-Gedanke „schnell oder stabil, such dir was aus" ist also empirisch falsch. Was es tatsächlich gibt, ist ein Trade-off zwischen *beidem* und dem Aufwand, die Praktiken zu etablieren.

Alle vier lassen sich aus Git und deiner CI erheben, ohne jemanden zu befragen — das ist ein weiterer Vorteil gegenüber den meisten Alternativen.

## 2. Was AI tatsächlich verschiebt

Hier ist die Messung, die mehr wert ist als jede andere in diesem Modul: die **Zerlegung der Lead Time**.Das ist die Antwort auf „warum merken wir eigentlich nichts". Wenn Coding 20 % der Lead Time ausmacht und sich halbiert, sind das 10 % Gesamtverbesserung — im Rauschen der Wochenschwankung. Der größte Block ist bei den meisten Teams die **Wartezeit auf Review**, und die berührt AI-gestütztes Coding überhaupt nicht.

Das ist Modul 0 in Zahlen: Der Engpass wandert nach rechts. Und daraus folgt die eigentlich handlungsleitende Erkenntnis: **Wenn du Lead Time verbessern willst, ist der Hebel nicht das Codieren, sondern die Review-Latenz** — kleinere PRs, klarere Zuständigkeit, feste Review-Slots, Merge Queue. Ein Team, das AI einführt, ohne den Review-Prozess anzufassen, wird in den Zahlen wenig sehen.

Und der Satz, der in jeder Diskussion fallen sollte:

> **Steigende Deployment Frequency bei gleichzeitig steigender Change Failure Rate ist kein Erfolg.** Das ist mehr Ausschuss pro Woche.

## 3. Metriken, die du nicht benutzen solltest

- **Codezeilen.** Immer falsch, und im AI-Zeitalter absurd — die Größe, die früher wenigstens mühsam zu erzeugen war, ist jetzt kostenlos.
- **Anzahl Commits oder PRs.** Misst Zerlegung, nicht Wert.
- **Story Points und Velocity.** Messen Schätzverhalten. Sie steigen zuverlässig, sobald man sie zum Ziel macht.
- **„Anteil des Codes, der von AI geschrieben wurde."** Die verführerischste von allen, weil sie so leicht zu erheben ist und so gut in Vorstandsfolien aussieht. Sie misst **Adoption, nicht Wert** — und schafft einen Anreiz, mehr generieren zu lassen, statt besser zu arbeiten.
- **Acceptance Rate von Autocomplete-Vorschlägen.** Misst Bequemlichkeit. Ein Vorschlag, der akzeptiert und zwei Minuten später umgeschrieben wird, zählt positiv.
- **Selbsteingeschätzte Zeitersparnis.** Der schwächste Datenpunkt überhaupt.

Zum letzten Punkt lohnt ein Befund, den du kennen solltest: METR hat 2025 eine randomisierte Studie mit erfahrenen Open-Source-Entwicklern veröffentlicht, die an großen, ihnen sehr vertrauten Repositories arbeiteten. Mit AI-Werkzeugen waren sie im Mittel **langsamer** — während sie sowohl vorher eine deutliche Beschleunigung erwarteten als auch hinterher glaubten, schneller gewesen zu sein.

Die Studie ist klein und ihr Kontext sehr spezifisch; sie belegt *nicht*, dass AI-Werkzeuge generell verlangsamen — in anderen Kontexten sind die Gewinne gut belegt. Was sie robust zeigt, ist die **Wahrnehmungslücke**: Die subjektive Einschätzung der eigenen Geschwindigkeit ist als Messinstrument unbrauchbar. Wenn jemand in deiner Organisation die Wirkung von AI mit einer Umfrage belegen will, ist das der Befund, auf den du verweisen kannst.

## 4. Was du stattdessen messen solltest

**Das SPACE-Framework** ist die etablierte Antwort auf die Frage, warum eine einzelne Zahl nie reicht. Fünf Dimensionen: **S**atisfaction (Zufriedenheit und Wohlbefinden), **P**erformance (Ergebnisqualität), **A**ctivity (Menge an Output), **C**ommunication (Zusammenarbeit), **E**fficiency (Fluss und Unterbrechungsfreiheit). Die Kernregel: **Miss immer aus mindestens drei Dimensionen.** Wer nur Activity misst, misst das Falsche.

Konkret ergänzend zu DORA:

- **Die vier Zahlen aus Modul 0** — Build-Zeit, Testlaufzeit, Review-Latenz, Deploy-Zeit. Du hast sie als Baseline erhoben. Sie sind die direkteste Messung der Reibung im Loop.
- **Rework Rate / Code Churn** — der Anteil des Codes, der innerhalb von zwei bis drei Wochen wieder geändert wird. Das ist im AI-Zeitalter eine der aussagekräftigsten Metriken überhaupt, weil sie genau den Fall abbildet, in dem schnell produzierter Code sich als falsch erweist. Steigt die Churn Rate parallel zur Ausbringungsmenge, hast du Geschwindigkeit gegen Nacharbeit getauscht.
- **Review-Gründlichkeit** — Kommentare pro geänderter Zeile, Anteil der Freigaben ohne Kommentar. Das ist die Messung für Verification Debt aus Modul 10.
- **Onboarding-Zeit** — wie lange braucht jemand Neues bis zum ersten produktiven PR? Eine indirekte, aber ehrliche Metrik für die Qualität von Codebase und Dokumentation. Und, wie in Modul 8 gezeigt: dieselbe Größe, die für Modelle zählt.

## 5. Wie man ein Messkonzept baut, das nicht austricksbar ist

Fünf Prinzipien:

**Immer Paare.** Jede Durchsatzmetrik bekommt eine Qualitätsmetrik als Gegengewicht. Deployment Frequency gegen Change Failure Rate. Ausbringungsmenge gegen Rework Rate. Das ist die einzige verlässliche Absicherung gegen Goodhart's Law (Modul 3).

**Teamebene, nie Individualebene.** Sobald eine Metrik einer Person zugeordnet wird, wird sie optimiert statt genutzt. DORA misst ausdrücklich das *Delivery-System*, nicht Menschen — der häufigste Missbrauch dieser Kennzahlen besteht genau darin, sie in Leistungsbeurteilungen zu tragen.

**Baseline vor der Einführung.** Der mit Abstand häufigste Fehler in der Praxis: Man führt ein Werkzeug ein und fragt drei Monate später, ob es geholfen hat — ohne ein Vorher. Dann bleibt nur Selbstauskunft, und die ist nach Abschnitt 3 wertlos.

**Ergebnis statt Aktivität.** „Wie viel wurde generiert" ist Aktivität. „Wie schnell kommt Wert beim Nutzer an, bei welcher Fehlerrate, mit welchem Nacharbeitsanteil" ist Ergebnis.

**Qualitative Daten strukturiert erheben, nicht ignorieren.** Sie ersetzen keine Zahlen, aber sie erklären sie. Der Unterschied zwischen „gefühlt läuft's besser" und einer regelmäßigen, gleichbleibenden Befragung mit denselben Fragen ist erheblich.

## 6. Golden Paths und Platform Engineering

Das Skalierungsproblem lautet: Was bei dir funktioniert, muss im Team reproduzierbar sein — ohne dass du bei jedem neuen Projekt daneben sitzt.

Ein **Golden Path** ist der gepflegte, unterstützte, dokumentierte Weg, etwas zu tun. Nicht der einzige erlaubte, sondern der einfachste. Der Leitsatz:

> **Mach den richtigen Weg zum einfachsten Weg.**

Das ist Governance durch Reibungsgefälle statt durch Verbote — und es funktioniert nachweislich besser, weil es keine Durchsetzung braucht.

Für dein Thema heißt das konkret: ein Template-Repository mit fertigem Harness (Hooks, `make lint`, `make test`, Pipeline, Branch Protection), geteilte `AGENTS.md`-Bausteine für die Konventionen, die überall gelten, und einheitliche Prüfbefehle über alle Projekte hinweg. Wer ein neues Projekt startet, bekommt Modul 2, 3, 5 und 9 geschenkt.

**Platform Engineering** ist die Disziplin dahinter: eine interne Plattform als *Produkt*, mit Entwicklern als Kunden. Der typische Fehler ist, sie als Zwang zu bauen statt als Angebot. Wenn der Golden Path unbequemer ist als der Umweg, wird er umgangen — und dann hast du Governance-Theater plus ein Team, das dich für einen Bremser hält.

## 7. Standards und Verantwortung

Die **Definition of Done** gehört im AI-Kontext erweitert und explizit gemacht: Tests grün, inklusive negativer Tests für neue Endpoints (Modul 10); Test-Diff im Review gelesen; keine neuen Abhängigkeiten ohne Freigabe; Migrationen nach Expand–Migrate–Contract; Flags mit Ablaufdatum.

Und ein Punkt, der ausgesprochen werden muss, weil sich sonst diffuse Verantwortungslosigkeit einschleicht:

> **Verantwortlich für AI-generierten Code ist der Mensch, der den Pull Request aufmacht.**

Nicht das Modell, nicht das Werkzeug, nicht „die AI hat das so gemacht". Wenn dieser Satz nicht explizit gilt, entsteht eine Zone, in der sich niemand zuständig fühlt — und genau dort sammelt sich Verification Debt.

## 8. Die Adoptionskurve — was tatsächlich passiert

Damit du differenziert antworten kannst statt evangelistisch:

Der typische Verlauf ist Begeisterung, dann Ernüchterung, dann differenzierter Einsatz. In den ersten Wochen sind die Gewinne real und groß — Boilerplate, unbekannte APIs, Tests nach Vorlage. Danach stellt sich heraus, dass die schwierigen Teile schwierig bleiben, weil sie nie am Tippen hingen.

**Wo der Nutzen empirisch am robustesten ist:** neue Projekte ohne Altlasten, unbekannte Sprachen und Frameworks, Boilerplate und Glue Code, mechanische Migrationen über viele Dateien, das Verstehen fremder Codebasen, Tests nach etabliertem Muster.

**Wo er klein oder negativ ist:** sehr große, sehr vertraute Codebasen mit hohen impliziten Qualitätsanforderungen — genau der Kontext der METR-Studie. Hochspezialisierte Fachdomänen. Und alle Aufgaben, bei denen die *Spezifikation* die eigentliche Arbeit ist und die Implementierung der triviale Teil.

Diese Differenzierung sauber vortragen zu können, ist übrigens genau das, was dich in einem Raum voller Software Engineers glaubwürdig macht — deutlich mehr als jede Werkzeugkenntnis.

## 9. Die Frage beantworten

Ein Antwortgerüst, das in einem Führungskreis funktioniert:

1. **Was wir gemessen haben** — DORA-Vierergespann plus Rework Rate, Baseline von vor der Einführung
2. **Wo es gewirkt hat** — mit Zahl und Kontext, welche Art von Arbeit
3. **Wo nicht** — ehrlich, mit der Lead-Time-Zerlegung als Erklärung
4. **Was der Engpass jetzt ist** — meist Review-Kapazität, und was das kostet
5. **Was wir als Nächstes tun** — der nächste Hebel, nicht mehr Werkzeuglizenzen

Punkt 3 ist der, der dir Glaubwürdigkeit verschafft. Wer nur Erfolge berichtet, wird nicht geglaubt.

## 10. Anti-Patterns

- Eine einzelne Metrik zum Ziel machen
- DORA-Kennzahlen auf Individuen anwenden
- Nach der Einführung messen, ohne Baseline davor
- „Anteil AI-generierten Codes" als Erfolgsmaß
- Selbsteingeschätzte Zeitersparnis als Beleg
- Deployment Frequency feiern, ohne Change Failure Rate danebenzustellen
- Golden Path als Zwang statt als Angebot
- AI einführen, ohne den Review-Prozess anzufassen
- Verantwortung für generierten Code nicht explizit zuweisen
- Nur Erfolge berichten

## 11. Glossar

DORA-Metriken · Deployment Frequency · Lead Time for Changes · Change Failure Rate · Recovery Time · Durchsatz vs. Stabilität · Lead-Time-Zerlegung · Review-Latenz · SPACE-Framework · Rework Rate / Code Churn · Vanity-Metrik · Goodhart's Law · Wahrnehmungslücke · Baseline · Golden Path · Platform Engineering · Interne Plattform als Produkt · Definition of Done · CODEOWNERS · Adoptionskurve

## 12. Übungen

**Ü1 — DORA erheben.** Berechne für ein Repository, auf das du Zugriff hast, alle vier Kennzahlen aus Git- und CI-Daten. Wenn Change Failure Rate schwer zu bestimmen ist: Definiere sie als Anteil der Deployments, denen innerhalb von 24 Stunden ein Hotfix oder Rollback folgte.

**Ü2 — Lead Time zerlegen.** Nimm die letzten dreißig PRs und miss die fünf Abschnitte aus dem Diagramm einzeln. Der größte Block ist dein tatsächlicher Engpass — und mit hoher Wahrscheinlichkeit nicht der, den du erwartet hättest.

**Ü3 — Rework Rate.** Bestimme für die letzten zwei Monate, welcher Anteil der geänderten Zeilen innerhalb von drei Wochen erneut geändert wurde. Vergleiche, wenn möglich, Zeiträume mit unterschiedlich starkem AI-Einsatz.

**Ü4 — Eine Vanity-Metrik aushebeln.** Nimm „Anteil AI-generierten Codes" und beschreibe drei konkrete Wege, sie zu erhöhen, ohne dass irgendein Wert entsteht. Das ist die Übung, die dich in Diskussionen schützt.

**Ü5 — Messkonzept entwerfen.** Entwirf für ein Team von acht Entwicklern ein Messkonzept nach den fünf Prinzipien aus Abschnitt 5. Zwei Seiten: was, wie erhoben, in welcher Kadenz, welches Paar gegen welches. Das ist ein Artefakt, das du herzeigen könntest.

**Ü6 — Golden Path bauen.** Erstelle ein Template-Repository, das dein Harness aus Modul 9 vollständig enthält. Lass jemanden ohne deine Hilfe damit ein neues Projekt starten und miss, wie lange es bis zum ersten grünen PR dauert.

**Ü7 — Definition of Done.** Formuliere eine erweiterte Definition of Done für AI-gestützte Entwicklung, in der jede Zeile auf ein Modul dieser Agenda zurückgeht. Prüfe für jede Zeile: Ist das eine Regel, ein Check oder eine Grenze (Modul 9, Abschnitt 9)?

**Ü8 — Die Antwort proben.** Schreibe die Antwort auf „bringt uns AI eigentlich etwas?" nach dem Gerüst aus Abschnitt 9 auf — für einen Kontext, den du kennst, mit echten oder plausibel geschätzten Zahlen. Fünf Minuten Vortragszeit.

Damit ist die Agenda inhaltlich durch. Was bleibt, ist der Capstone: ein Feature von der Spec bis in Produktion, mit vollem Harness, allein durchgezogen. Sag Bescheid, wenn du soweit bist — oder ob du vorher die Übungen aus den Modulen angehen willst, was ich an dieser Stelle sogar empfehlen würde.
