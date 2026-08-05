# Modul 7: Observability & Betrieb

Modul 6 endete mit einem Versprechen: Canary ohne belastbares Monitoring ist Theater. Das lösen wir hier ein. Gleichzeitig ist das der Perspektivwechsel, der Phase 2 abschließt — von *Software bauen* zu *Software betreiben*.

Ein Punkt vorweg, der dir den Einstieg erleichtern dürfte: Betrieb bedeutet, mit Signalen aus einem System zu arbeiten, das man nicht vollständig durchschaut, unter Zeitdruck, auf Basis unvollständiger Evidenz. Diese Denkweise ist dir aus dem AI-Bereich vertrauter als vielen klassischen Entwicklern.

## 1. Monitoring vs. Observability

**Monitoring** beantwortet Fragen, die du vorher kanntest. Du baust ein Dashboard, definierst einen Schwellwert, bekommst eine Warnung. Das setzt voraus, dass du das Problem vorhergesehen hast.

**Observability** erlaubt dir, Fragen zu stellen, an die du vorher nicht gedacht hast. „Warum sind seit 14:20 Uhr die Anfragen von Android-Nutzern in Frankfurt langsam, aber nur bei aktiviertem Feature-Flag X?"

Der technische Unterschied liegt in **Kardinalität und Kontextreichtum**. Monitoring aggregiert vorab: Aus einer Million Anfragen wird eine Zahl, und die Dimensionen, nach denen du später aufschlüsseln kannst, musst du vorher festgelegt haben. Observability behält Ereignisse mit vielen Attributen und erlaubt beliebige Aufschlüsselung im Nachhinein.

In der klassischen Formulierung: Monitoring deckt *known unknowns* ab, Observability *unknown unknowns*. Und der praktische Test lautet:

> **Kannst du eine neue Frage über dein System beantworten, ohne Code zu ändern und neu zu deployen?**

Wenn nein, hast du Monitoring. Das ist nichts Schlimmes — für viele Systeme reicht es völlig. Aber der Unterschied entscheidet, ob du einen unerwarteten Incident in fünf Minuten oder in fünf Stunden verstehst.

## 2. Die drei Säulen — und warum die Metapher irreführt

**Logs** sind diskrete Ereignisse mit Kontext. Höchste Detailtiefe, teuerste Speicherung.
**Metriken** sind numerische Aggregate über Zeit. Sehr billig, sehr kontextarm.
**Traces** zeigen den Weg einer einzelnen Anfrage durch ein verteiltes System.

Die verbreitete „drei Säulen"-Metapher hat einen Konstruktionsfehler: Sie suggeriert drei getrennte Systeme mit drei getrennten Werkzeugen und drei getrennten Rechnungen. Genau so haben es viele Firmen gebaut, und genau das ist das Problem. Der eigentliche Wert entsteht in der **Korrelation** — vom Alert zur Metrik, von der Metrik zum betroffenen Trace, vom Trace zu den Logzeilen dieser einen Anfrage. Ohne gemeinsame Identifikatoren machst du diesen Weg von Hand über drei Oberflächen, und das kostet dich die entscheidenden Minuten.

**OpenTelemetry** existiert genau deshalb: ein herstellerneutraler Standard für Instrumentierung, der Logs, Metriken und Traces mit gemeinsamem Kontext erzeugt. Du instrumentierst einmal und kannst das Backend wechseln, ohne den Code anzufassen. Wenn du dir aus diesem Modul einen Technologienamen merkst, dann diesen.

## 3. Logs richtig

**Strukturiert, nicht Freitext.** Der Unterschied:

```python
# schlecht — nur mit Regex durchsuchbar
logger.info(f"Login failed for {email} from {ip}")

# gut — als Felder abfragbar
logger.info("login_failed", extra={"user_id": uid, "reason": "bad_password", "ip": ip})
```

Der Grund ist schlicht Skalierung: `grep` funktioniert bei tausend Zeilen und versagt bei Millionen. Felder lassen sich filtern, gruppieren und aggregieren.

**Log-Level bedeuten etwas** — und der häufigste Fehler ist, dass in einer Codebase entweder alles `INFO` oder alles `ERROR` ist. Wenn alles ERROR ist, bedeutet ERROR nichts. Die Konvention, die trägt:

- `DEBUG` — nur für Entwicklung, in Produktion aus
- `INFO` — normale Geschäftsereignisse, die man später nachvollziehen will
- `WARNING` — etwas Ungewöhnliches, das aber behandelt wurde (Retry war nötig, Fallback griff)
- `ERROR` — eine Operation ist fehlgeschlagen, ein Nutzer ist betroffen
- `CRITICAL` — das System als Ganzes ist gefährdet

**Korrelations-IDs sind die wichtigste Einzelmaßnahme.** Jede eingehende Anfrage bekommt eine ID, die durch alle Services, alle Log-Zeilen und alle Queue-Nachrichten mitgereicht wird. Ohne sie kannst du in einem verteilten System nicht rekonstruieren, welche Log-Zeilen zusammengehören. Mit ihr ist es eine Filterabfrage.

**Was nie ins Log gehört:** Passwörter, Tokens, Sitzungsschlüssel, vollständige Kartennummern — und personenbezogene Daten ohne Rechtsgrundlage. Der letzte Punkt ist in deinem Kontext keine Formalie: Logs sind Datenverarbeitung im Sinne der DSGVO, sie werden typischerweise an einen Dritt-Anbieter gesendet, oft in ein Drittland, und sie haben eine Aufbewahrungsfrist, die jemand festlegen muss. Nutzer-IDs statt E-Mail-Adressen loggen ist die einfache Standardantwort.

**Kosten** sind der unterschätzte Faktor. Logging ist bei vielen Firmen der größte Posten der Observability-Rechnung, und der Reflex „loggen wir sicherheitshalber alles" ist teuer. Sampling bei hochfrequenten, wenig informativen Ereignissen ist normal und richtig.

## 4. Metriken richtig

Die Typen, die du kennen musst: **Counter** (nur steigend — Anfragen, Fehler), **Gauge** (auf und ab — aktive Verbindungen, Speicherbelegung), **Histogram** (Verteilung in Buckets — Latenzen).

Warum Latenz ein Histogramm sein muss und kein Gauge: Du willst Perzentile, und Perzentile brauchen die Verteilung. Ein Gauge „aktuelle Latenz" ist informationsfrei.

Und der Fehler, den fast alle einmal machen: **Perzentile lassen sich nicht mitteln.** Du kannst nicht den p99 von zehn Instanzen nehmen und den Durchschnitt bilden — das Ergebnis ist bedeutungslos. Deshalb aggregiert man Histogramm-Buckets und berechnet das Perzentil daraus.

**Kardinalität** ist der Begriff, an dem sich Erfahrung zeigt. Jede Kombination von Label-Werten erzeugt eine eigene Zeitreihe. `http_requests{method, status, endpoint}` ergibt vielleicht 200 Zeitreihen. Fügst du `user_id` als Label hinzu, sind es Millionen — dein Metriksystem geht in die Knie und die Rechnung explodiert. Hohe Kardinalität gehört in Logs und Traces, nicht in Metriken. Das ist die Arbeitsteilung zwischen den drei.

**Was misst man?** Auf die Frage gibt es drei etablierte Antworten:

- **RED** für Dienste: Rate (Anfragen pro Sekunde), Errors (Fehlerrate), Duration (Latenzverteilung)
- **USE** für Ressourcen: Utilization, Saturation, Errors
- **Four Golden Signals** (Google SRE): Latency, Traffic, Errors, Saturation

Die drei überschneiden sich stark. Für einen HTTP-Dienst ist RED der pragmatischste Einstieg — drei Metriken pro Endpoint, und du hast 80 % dessen, was du im Ernstfall brauchst.

## 5. Tracing

Ein **Trace** ist der vollständige Weg einer Anfrage. Ein **Span** ist ein Abschnitt darin — ein Serviceaufruf, eine Datenbankabfrage, eine Berechnung. Spans sind ineinander verschachtelt, und der Kontext wird über HTTP-Header weitergereicht (der Standard heißt `traceparent`).

Warum das den Unterschied macht:Ohne Trace hast du die Information „der Checkout-Endpoint braucht 420 Millisekunden" — damit kannst du nichts anfangen. Mit Trace siehst du sofort, dass fünf identische Abfragen nacheinander laufen statt einer gebündelten. Das ist das **N+1-Problem**, der häufigste Performance-Fehler in Anwendungen mit ORM überhaupt — und ein schönes Beispiel dafür, wie Modul 4 und Modul 7 zusammenhängen: Der Trace zeigt dir ein Betriebssymptom, dessen Ursache eine Designentscheidung ist.

**Sampling** ist bei Traces unvermeidlich, weil vollständige Erfassung bei hohem Volumen unbezahlbar ist. Zwei Verfahren: **head-based** entscheidet zu Beginn der Anfrage (billig, einfach, aber du verlierst genau die interessanten Fälle mit derselben Wahrscheinlichkeit wie die langweiligen) und **tail-based** entscheidet nach Abschluss des Traces (behalte alles, was fehlerhaft oder langsam war — deutlich wertvoller, aber aufwändiger, weil man den Trace bis zur Entscheidung zwischenspeichern muss).

## 6. SLI, SLO, SLA und das Error Budget

Das ist der konzeptionelle Kern des Moduls.

**SLI** — Service Level Indicator. Eine Messgröße, definiert aus Nutzersicht. Beispiel: *Anteil der Anfragen, die in unter 300 ms mit einem Status unter 500 beantwortet werden.*

**SLO** — Service Level Objective. Ein Zielwert für den SLI über ein Zeitfenster. *99,9 % über 30 rollierende Tage.* Ein internes Versprechen.

**SLA** — Service Level Agreement. Ein Vertrag mit Kunden, an den Konsequenzen geknüpft sind (Gutschriften, Vertragsstrafen). Immer laxer als das SLO gesetzt, damit man Puffer hat, bevor es Geld kostet.

**Error Budget** — der Rest zu 100 %. Bei einem SLO von 99,9 % über 30 Tage sind das **43 Minuten** außerhalb des Ziels. Und jetzt kommt die Idee, die das Ganze von Buchhaltung zu einem Steuerungsinstrument macht:

> Diese 43 Minuten sind kein Versagen. Sie sind ein **Budget**, das du ausgeben darfst.

Damit löst sich der ewige Konflikt zwischen „schnell liefern" und „stabil bleiben" in eine gemeinsame Zahl auf. Budget übrig? Dann darf schneller und riskanter ausgerollt werden — der Canary darf kürzere Fenster haben, Continuous Deployment ist vertretbar. Budget aufgebraucht? Dann Feature-Stopp, bis die Zuverlässigkeit wieder stimmt. Die Entscheidung ist nicht mehr Verhandlungssache zwischen zwei Abteilungen mit unterschiedlichen Anreizen, sondern folgt aus einer Messung, auf die sich beide vorher geeinigt haben.

**Warum 100 % Verfügbarkeit ein falsches Ziel ist** — drei unabhängige Gründe:

1. **Die Kosten sind exponentiell.** Jede weitere Neun kostet ein Vielfaches der vorherigen. Von 99,9 % auf 99,99 % zu kommen bedeutet Multi-Region, aufwändige Failover-Mechanik und alles, was daran hängt.
2. **Der Nutzer merkt es nicht.** Sein Mobilfunknetz, sein WLAN und sein Browser sind zusammen deutlich unzuverlässiger als dein Dienst. Ab einem gewissen Punkt investierst du in Zuverlässigkeit, die niemand wahrnimmt.
3. **100 % bedeutet: nie ändern.** Und ein System, das nicht mehr geändert wird, ist ein sterbendes System.

Ein bisschen Unzuverlässigkeit ist die **Währung, mit der du Geschwindigkeit kaufst**. Das Error Budget macht diesen Handel explizit statt implizit.

Zwei Regeln für gute SLOs: Sie werden aus der **Nutzerperspektive** formuliert — „CPU unter 80 %" ist kein SLO, weil es keinen Nutzer interessiert. Und sie sind **erreichbar**: Ein SLO, das seit vier Monaten gerissen wird, wird ignoriert und ist damit wertlos.

## 7. Alerting

Der wichtigste Satz:

> **Alerte auf Symptome, nicht auf Ursachen.**

„Fehlerrate für Nutzer erhöht" ist ein Alert. „CPU bei 91 %" ist keiner — vielleicht ist das völlig normal, vielleicht läuft gerade ein Batch. Ursachen-Alerts erzeugen Lärm ohne Nutzerbezug, während echte Ausfälle mit unerwarteter Ursache durchrutschen.

Jeder Alert muss drei Kriterien erfüllen: **dringend** (kann nicht bis morgen warten), **handlungsfähig** (es gibt etwas zu tun) und **für einen Menschen** (nicht automatisch behebbar). Fällt eines weg, gehört es in ein Dashboard oder in ein Ticket, nicht auf ein Telefon um 3 Uhr.

**Alert Fatigue** ist das eigentliche Problem im Betrieb. Kommen nachts fünf Alerts und vier bedeuten nichts, wird der fünfte ignoriert — und der fünfte war der echte. Alert-Hygiene ist deshalb kein einmaliges Setup, sondern ein Dauerauftrag: Jeder Alert, der ausgelöst hat und keine Handlung nach sich zog, gehört auf den Prüfstand.

Die moderne Praxis knüpft Alerts an die **Burn Rate** des Error Budgets: Wie schnell wird es verbraucht? Bei sehr schnellem Verbrauch (das Monatsbudget wäre in einer Stunde weg) wird jemand geweckt. Bei langsamem Verbrauch entsteht ein Ticket für den nächsten Werktag. Das trennt „jetzt sofort" von „wichtig, aber nicht nachts" automatisch — und es alarmiert auf das, was tatsächlich das Versprechen gefährdet.

Und: In jeden Alert gehört ein Link auf ein **Runbook** — was bedeutet das, was prüfe ich zuerst, was sind die üblichen Ursachen, wie mildere ich es ab. Um 3 Uhr denkt niemand gut.

## 8. Incident Response

**Rollen** trennen ist der wichtigste organisatorische Kniff. Der **Incident Commander** koordiniert, entscheidet und debuggt *nicht selbst* — sobald er in den Code abtaucht, ist niemand mehr da, der den Überblick behält. Daneben: jemand für Kommunikation (Statusseite, Stakeholder) und jemand für die technische Arbeit.

**Mitigation vor Ursachenforschung.** Das ist die Lektion, die am schwersten fällt, weil der Reflex jedes technisch denkenden Menschen genau umgekehrt ist. Erst den Blutfluss stoppen — zurückrollen, Flag ausschalten, Traffic umleiten, Last abwerfen. Verstehen kommt danach. Ein Rollback, dessen Ursache man nie geklärt hat, ist ein erfolgreicher Incident.

**Die Kennzahlen:** MTTD (Zeit bis zur Erkennung), MTTR (Zeit bis zur Behebung), MTBF (Zeit zwischen Ausfällen). Der interessante Punkt: In komplexen Systemen ist MTTR meist der lohnendere Hebel als MTBF. Ausfälle vollständig zu verhindern ist unbezahlbar; sie in drei statt in dreißig Minuten zu beheben, ist machbar.

**Blameless Postmortem.** Die Grundannahme lautet: Jeder Beteiligte hat mit den Informationen, die er zu diesem Zeitpunkt hatte, vernünftig gehandelt. Das ist keine Höflichkeitsfloskel, sondern eine Funktionsvoraussetzung — sobald Schuldzuweisungen im Raum stehen, hörst du auf, die Wahrheit zu erfahren, und damit verlierst du die Fähigkeit zu lernen. Genau deshalb ist die Kultur hier eine technische Frage, keine soziale.

Praktisch heißt das: **„Menschliches Versagen" ist nie eine Ursache, sondern ein Ausgangspunkt.** Warum war die falsche Aktion möglich? Warum war sie naheliegend? Warum hat nichts sie abgefangen? Wenn jemand das Produktivsystem gelöscht hat, ist die Erkenntnis nicht „mehr aufpassen", sondern dass ein einzelner Befehl das konnte.

Eine Warnung zu den beliebten **Five Whys**: Sie suggerieren eine einzelne lineare Ursachenkette. Komplexe Systeme scheitern fast nie so, sondern durch das Zusammentreffen mehrerer Faktoren, von denen jeder für sich harmlos war. Suche mehrere beitragende Faktoren, nicht die eine Wurzel.

Und: **Maßnahmen mit Eigentümer und Frist.** Ein Postmortem ohne umgesetzte Maßnahmen ist Dokumentationstheater — und dieselbe Störung kommt wieder.

Zum Thema **On-Call** nur der eine Punkt, der konzeptionell zählt: *You build it, you run it.* Wer nachts von seinem eigenen Code geweckt wird, baut anders. Das ist die stärkste Qualitätsrückkopplung, die es in der Softwareentwicklung gibt — und der Grund, warum die Trennung von Entwicklung und Betrieb in die falsche Richtung wirkt.

## 9. Die AI-Verbindung

**Produktionssignale sind Debug-Kontext für Agenten.** „Der Endpoint ist langsam" ist für ein Modell genauso unbrauchbar wie für dich. Ein Trace mit Span-Aufschlüsselung ist ein lösbares Problem — die Information, dass fünf identische Abfragen nacheinander laufen, führt praktisch direkt zur richtigen Änderung. Das ist der „Measure"-Knoten aus Modul 0 und das, was Addy Osmani als *Resource efficiency* und *AI performance refactorings* am äußeren Rand des Diagramms markiert.

**AI im Incident** — nützlich für Korrelation über viele Signale, für Zusammenfassung der Chronologie, für Hypothesengenerierung. Gefährlich als Entscheider: Ein Modell, das überzeugend die falsche Ursache benennt, kostet dich die entscheidenden Minuten, in denen sonst gemildert worden wäre. Und die Reihenfolge aus Abschnitt 8 gilt weiterhin — Mitigation braucht keine Ursachenanalyse. Automatisch erzeugte Postmortems sind gut für die Zeitleiste und wertlos für die Erkenntnis, weil die Erkenntnis aus dem Gespräch der Beteiligten entsteht.

**Und der Punkt, an dem du einen Startvorteil hast:** Alles in diesem Modul gilt genauso für AI-Systeme selbst — nur mit anderen Signalen. LLM-Anwendungen brauchen Traces über Agentenschritte und Werkzeugaufrufe, Token-Verbrauch und Kosten pro Anfrage, Latenz pro Modellaufruf, Trefferquoten im Retrieval, Eval-Ergebnisse gegen Produktionsverkehr. Das ist LLM-Observability, und es ist derselbe Werkzeugkasten — OpenTelemetry hat inzwischen Konventionen dafür.

Eine Besonderheit solltest du dabei mitdenken: **Agenten in Produktion sind nicht deterministisch.** Ein Retry liefert ein anderes Ergebnis, nicht dasselbe. Das bricht eine stillschweigende Annahme klassischer Betriebspraxis — „gleicher Input, gleicher Output" —, und es heißt, dass Fehlerraten bei AI-Komponenten Verteilungen sind statt Ja/Nein. Deine SLIs müssen das abbilden, und deine Alerts dürfen nicht bei jeder einzelnen abweichenden Antwort feuern.

## 10. Anti-Patterns

- Auf Ursachen alarmieren statt auf Nutzersymptome
- Alerts, die niemand mehr ernst nimmt, weil sie zu oft ohne Handlung feuern
- Freitext-Logs ohne Struktur und ohne Korrelations-ID
- Hochkardinale Labels in Metriken
- Latenz als Mittelwert
- Dashboards, die niemand ansieht, statt SLOs, an die sich jemand hält
- SLOs, die seit Monaten gerissen werden und trotzdem unverändert stehen
- Ursachenforschung während des laufenden Incidents statt Mitigation
- Postmortem ohne Maßnahmen mit Eigentümer und Frist
- Personenbezogene Daten in Logs bei einem Dritt-Anbieter
- Ein Incident Commander, der selbst debuggt

## 11. Glossar

Monitoring vs. Observability · Known/unknown unknowns · Kardinalität · OpenTelemetry · Strukturiertes Logging · Log-Level · Korrelations-ID · Sampling · Counter / Gauge / Histogram · RED · USE · Four Golden Signals · Trace · Span · Context Propagation · `traceparent` · Head- vs. Tail-based Sampling · N+1-Problem · SLI · SLO · SLA · Error Budget · Burn Rate · Runbook · Alert Fatigue · Incident Commander · Severity · Mitigation · MTTD / MTTR / MTBF · Blameless Postmortem · Beitragende Faktoren · On-Call · You build it, you run it · LLM-Observability

## 12. Übungen

**Ü1 — Strukturiert loggen.** Stelle dein Projekt auf JSON-Logs um und führe eine Korrelations-ID ein, die von der eingehenden Anfrage durch alle Funktionsaufrufe gereicht wird. Stelle danach eine Frage, die du vorher nicht beantworten konntest — etwa alle Log-Zeilen einer bestimmten fehlgeschlagenen Anfrage.

**Ü2 — RED-Metriken.** Instrumentiere einen Endpoint mit Rate, Fehlerrate und Latenz-Histogramm. Erzeuge Last und lies p50, p95 und p99 ab. Vergleiche sie mit dem Mittelwert — das ist Ü3 aus Modul 6, jetzt mit echten Daten.

**Ü3 — Kardinalität sprengen.** Füge einer Metrik bewusst ein hochkardinales Label hinzu (etwa eine Request-ID) und beobachte, wie viele Zeitreihen entstehen. Entferne es wieder. Das ist eine Lektion, die man einmal selbst sehen sollte.

**Ü4 — Tracing einbauen.** Instrumentiere dein Projekt mit OpenTelemetry und baue absichtlich ein N+1-Problem ein. Sieh es dir im Wasserfall an und behebe es. Miss vorher und nachher.

**Ü5 — Ein SLO formulieren.** Definiere für dein Projekt einen SLI aus Nutzersicht, ein SLO über 30 Tage und das daraus folgende Error Budget in Minuten. Formuliere dazu, was passiert, wenn das Budget zu 50 %, zu 90 % und zu 100 % verbraucht ist. Eine halbe Seite — und ein Artefakt, das in einem echten Team funktionieren würde.

**Ü6 — Alert schreiben.** Formuliere einen Burn-Rate-Alert auf dein SLO und ein zugehöriges Runbook. Prüfe deinen Alert gegen die drei Kriterien aus Abschnitt 7.

**Ü7 — Incident simulieren.** Erzeuge einen künstlichen Ausfall (Datenbank stoppen, Latenz künstlich erhöhen, eine fehlerhafte Version deployen). Stoppe die Zeit bis zur Erkennung und bis zur Behebung. Schreibe danach ein Postmortem mit Zeitleiste, mindestens drei beitragenden Faktoren und Maßnahmen mit Eigentümer.

**Ü8 — Der eigene Vorsprung.** Instrumentiere einen Agentenlauf mit Traces über die einzelnen Schritte und Werkzeugaufrufe, plus Kosten und Token pro Schritt. Überlege dir dann, wie ein SLI für eine nichtdeterministische Komponente aussehen müsste — das ist eine offene Frage in der Branche, und du bist gut aufgestellt, um dazu etwas Eigenes zu sagen.

Damit ist Phase 2 abgeschlossen. Ab Modul 8 dreht sich die Perspektive: Bis hier hast du das Fundament gebaut, ab jetzt bauen wir AI systematisch darauf — Context Engineering und Spec-Driven Development zuerst, dann das eigentliche Harness.
