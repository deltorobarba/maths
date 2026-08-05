# Modul 10: AI-Code kritisch prüfen — Security & Verification Debt

Modul 9 hat einen Regelkreis gebaut. Aber ein Regelkreis kann nur auf Signale reagieren, die er misst. Dieses Modul handelt von den Fehlerklassen, die **kein rotes Signal erzeugen** — und deshalb kommt es nach dem Harness und nicht davor.

## 1. Warum diese Fehler unsichtbar sind

Der zentrale Befund vorweg:Ein fehlender Autorisierungscheck wirft keine Exception. Er hat keinen falschen Typ. Er bricht keinen Test, denn der Test prüft, dass der berechtigte Nutzer Zugriff bekommt — und das tut er. Alle Signale, die dein Harness liefert, prüfen **Anwesenheiten**: Ist der Code syntaktisch sauber, typkonsistent, verhält er sich wie spezifiziert. Die gefährlichsten Fehler in AI-generiertem Code sind aber **Abwesenheiten**.

## 2. Das mentale Modell

Behandle AI-Output wie Code von einem sehr schnellen, sehr selbstbewussten Entwickler, der drei Eigenschaften hat: Er kennt deine Fachdomäne nicht, er hat kein Verantwortungsgefühl für das Gesamtsystem, und — der wichtigste Punkt — er hat aus einem Korpus gelernt, der überwiegend aus öffentlichem Code besteht.

Dieser letzte Punkt wird selten ausgesprochen und erklärt viel. Öffentlicher Code enthält sehr viele Tutorials, Beispiele und Antworten aus Foren, die auf *Verständlichkeit* optimiert sind, nicht auf Sicherheit. Ein Tutorial zu FastAPI lässt Autorisierung weg, weil sie vom Thema ablenkt. Ein Beispiel zu Datenbankabfragen benutzt String-Formatierung, weil es kürzer aussieht. Die Verteilung, aus der generiert wird, ist nicht die Verteilung des produktionsreifen Codes.

Daraus folgt das charakteristische Profil: **AI-Code ist stilistisch überdurchschnittlich und sicherheitstechnisch bestenfalls durchschnittlich.**

## 3. Der Review-Bias — warum du anders lesen musst

Hier steckt der eigentlich schwierige Teil, und er ist psychologisch, nicht technisch.

Bei menschlichem Code sind Stilsignale **Korrelate von Sorgfalt**. Wenn jemand konsistent benennt, Docstrings schreibt und sauber strukturiert, hat er in aller Regel auch nachgedacht. Diese Heuristik ist über Jahre erlernt und funktioniert gut.

Bei AI-Code entkoppelt sich diese Korrelation vollständig. Konsistente Benennung, ordentliche Docstrings und saubere Struktur sind kostenlos — sie sagen exakt nichts über die Sorgfalt der Überlegung aus. Deine erlernte Heuristik führt dich also nicht nur nicht mehr, sie führt dich aktiv in die Irre.

Dazu kommt der **Automation Bias**: Menschen überschätzen systematisch die Korrektheit maschineller Ausgaben, und zwar umso stärker, je flüssiger sie präsentiert werden.

Die praktische Konsequenz ist ein Wechsel der Leitfrage beim Lesen:

> Nicht: „Sieht das richtig aus?"
> Sondern: **„Was müsste wahr sein, damit das falsch ist — und ist es das?"**

## 4. Die Fehlerklassen

### a) Fehlende Zugriffskontrolle

Das ist Platz eins der OWASP Top 10 und gleichzeitig die häufigste Lücke in AI-generiertem Code. Der Grund liegt in einer Verwechslung, die im Code fast unsichtbar ist:

- **Authentifizierung** — wer bist du?
- **Autorisierung** — darfst du *diese* Ressource?

Ein generierter Endpoint prüft in aller Regel das Erste und übergeht das Zweite. `GET /orders/{id}` lädt die Bestellung anhand der ID und gibt sie zurück, wenn ein gültiges Token vorliegt — ohne zu prüfen, ob diese Bestellung dem anfragenden Nutzer gehört. Das ist eine **Insecure Direct Object Reference**, und sie funktioniert im Test einwandfrei, weil der Test mit der eigenen Bestellung testet.

Die Gegenmaßnahme ist eine Testkategorie, die niemand ungefragt schreibt — weder Menschen noch Modelle:

> **Zu jedem „der berechtigte Nutzer kann X" gehört ein „der unberechtigte Nutzer kann X nicht."**

Diese **negativen Tests** sind das wirksamste Einzelmittel dieses Moduls. Sie verwandeln eine Abwesenheit in eine Anwesenheit — und damit in etwas, das dein Harness prüfen kann. Wenn du aus diesem Modul eine Sache mitnimmst, dann diese.

### b) Die übrigen klassischen Lücken

- **Injection** — Datenbankabfragen per String-Konkatenation statt Parameterbindung, Shell-Aufrufe mit interpolierten Nutzerdaten, Template-Rendering mit unbereinigtem Input. Der Linter sieht nichts, der Typechecker sieht einen `str`.
- **Secrets an der falschen Stelle** — API-Schlüssel, die in den Client gelangen, oder hartkodierte Zugangsdaten „für den Anfang".
- **Unsichere Voreinstellungen** — CORS auf Wildcard, Debug-Modus aktiv, permissive Dateiberechtigungen, ausgeschaltete Zertifikatsprüfung. Häufig entstanden, weil etwas lokal nicht lief und die permissive Einstellung das Problem „gelöst" hat.
- **Fehlende Ratenbegrenzung** — wird praktisch nie generiert, wenn man nicht danach fragt.
- **Kryptografie** — veraltete Hashverfahren für Passwörter, selbstgebaute Verschlüsselung, unsichere Betriebsmodi. Modelle reproduzieren hier zuverlässig, was vor zehn Jahren in Tutorials stand.
- **Server-Side Request Forgery** — ein Dienst, der eine vom Nutzer gelieferte URL abruft, ohne einzuschränken, wohin.

Alle teilen dasselbe Merkmal aus Abschnitt 1. Deshalb gehört SAST (Modul 5) verbindlich in die Pipeline: Ein Sicherheitslinter prüft genau die Muster, für die es keine funktionalen Tests gibt.

### c) Lieferkette

Die charakteristischen Risiken, wenn Agenten Abhängigkeiten hinzufügen:

- **Halluzinierte Paketnamen.** Ein Modell erfindet gelegentlich ein Paket. Normalerweise scheitert die Installation — es sei denn, jemand hat den erfundenen Namen vorsorglich registriert und Schadcode hinterlegt. Dieses Muster hat den Namen *Slopsquatting* bekommen, weil die Halluzinationen von Modellen erstaunlich reproduzierbar sind und sich damit gezielt bedienen lassen.
- **Typosquatting** — die klassische Variante mit vertauschten Buchstaben.
- **Transitive Abhängigkeiten.** Du prüfst ein Paket, das zwölf weitere zieht, die wiederum vierzig ziehen. Die tatsächliche Angriffsfläche ist der gesamte Baum.
- **Installationsskripte**, die bei der Installation ausgeführt werden.

Die praktische Regel, die daraus folgt: **Eine neue Abhängigkeit ist eine Entscheidung, keine Implementierungsdetail.** Sie gehört in die Verbotsliste aus Modul 9 und über CODEOWNERS auf `pyproject.toml` und `package.json` in eine menschliche Freigabe. Dazu Lock-Dateien, Dependency Review als Gate im PR und eine SBOM (Modul 5).

### d) Prompt Injection in agentischen Workflows

Die neue Fehlerklasse, und die einzige, für die es keine saubere Lösung gibt.

Das Grundproblem: **Ein Modell unterscheidet nicht zuverlässig zwischen Instruktionen und Daten.** Beides ist Text im selben Kontext. Wenn dein Agent den Text eines GitHub-Issues liest, eine Webseite abruft, eine Fehlermeldung eines Drittsystems verarbeitet oder die Ausgabe eines MCP-Servers erhält, dann ist all das potenziell Text, der wie eine Anweisung wirken kann.

Ein nützliches Framing dafür — Simon Willison nennt es die *lethal trifecta*: Wirklich gefährlich wird es, wenn **drei Bedingungen gleichzeitig** erfüllt sind:

1. Der Agent hat Zugriff auf **private Daten** (dein Repository, deine Secrets, deine internen Systeme)
2. Er verarbeitet **nicht vertrauenswürdige Inhalte** (Issues von außen, Webseiten, fremde Abhängigkeiten)
3. Er kann **nach außen wirken** (committen, pushen, HTTP-Anfragen senden, Nachrichten schreiben)

Fehlt eine der drei, ist der mögliche Schaden erheblich begrenzt. Und genau das ist der Ansatzpunkt, denn:

> **Prompt Injection lässt sich nicht wegprompten.** Die Gegenmaßnahmen sind architektonisch, nicht sprachlich.

Für dich konkret: Ein CI-Workflow, der einen Agenten auf Basis eines von außen erstellten Issue-Textes startet, ihm Repository-Zugriff gibt und Commit-Rechte einräumt, kombiniert alle drei Faktoren. Das ist keine hypothetische Konstruktion — es ist ein Muster, das Teams tatsächlich bauen, weil es bequem ist.

Die praktischen Konsequenzen:

- **Trenne die Faktoren.** Ein Agent, der nicht vertrauenswürdige Inhalte verarbeitet, bekommt keine Schreibrechte. Ein Agent mit Schreibrechten liest nur kuratierte Eingaben.
- **Menschliche Bestätigung an den Stellen mit Außenwirkung** — beim Push, beim Netzwerkzugriff, bei irreversiblen Operationen.
- **Minimale Berechtigungen** (Modul 5) und Sandboxing (Modul 9).
- **MCP-Server sind eine Vertrauensentscheidung.** Ihre Ausgabe landet ungefiltert im Kontext.

### e) Verification Debt

Der strukturelle Punkt, und der, der dich langfristig am meisten betrifft.

**Definition:** der wachsende Abstand zwischen produziertem und tatsächlich geprüftem Code.

Die Ursache ist der Effekt aus Modul 0: Die Produktion von Code ist dramatisch billiger geworden, die Verifikation nicht. Review braucht weiterhin menschliche Aufmerksamkeit in Menschenzeit.

Was sie gefährlich macht, ist dasselbe wie bei den Sicherheitslücken — **sie erzeugt kein Signal**. Ein roter Build ist sichtbar. Ein PR, der mit „LGTM" durchgewunken wurde, sieht in jeder Statistik identisch aus wie einer, der gründlich geprüft wurde.

Die Symptome, an denen du sie erkennst:

- PRs werden im Schnitt größer
- Die Zeit pro geprüfter Codezeile sinkt
- Freigaben ohne einen einzigen Kommentar häufen sich
- In Produktion gefundene Fehler nehmen zu, während die Testabdeckung konstant bleibt

Und der Verstärker aus Modul 3: Wenn auch die Tests generiert sind, prüft die Verifikation sich selbst. Der Zirkelschluss ist dann vollständig, und die Zahlen sehen weiterhin gut aus.

Die Gegenmaßnahmen sind unbequem, weil sie alle das Tempo drosseln — und genau deshalb werden sie oft nicht ergriffen:

- **PR-Größe begrenzen**, hart. Das ist die wirksamste Einzelmaßnahme.
- **Review-Kapazität als Engpass planen**, nicht als unbegrenzte Ressource behandeln.
- **Mutation Testing** als periodische Stichprobe darauf, ob die Tests noch etwas zusichern.
- **Testschreiber und Implementierer trennen** (Modul 3, Abschnitt 9).

Der ehrliche Satz dazu, den du in einer Diskussion brauchen wirst:

> Der Durchsatz eines Teams ist durch seine **Verifikationskapazität** begrenzt, nicht durch seine Generierungskapazität. Wer das ignoriert, erhöht nicht die Geschwindigkeit, sondern verlagert nur Fehler in die Produktion.

### f) Lizenz und geistiges Eigentum

Kurz, weil die Rechtslage im Fluss ist und ich kein Jurist bin: Trainingskorpora enthalten Code unter unterschiedlichen Lizenzen, und bei größeren übernommenen Blöcken — insbesondere bei sehr spezifischen Algorithmen — besteht ein Restrisiko. Die betriebliche Antwort ist ein Lizenz-Scanner in der Pipeline, eine dokumentierte Richtlinie und im Zweifel eine Rückfrage bei der Rechtsabteilung. Für regulierte oder produktnahe Codebasen ist das keine Formalie.

## 5. Der Review-Ablauf für AI-Code

Eine Reihenfolge, die du dir angewöhnen solltest — sie ergibt sich aus allem Vorherigen:

1. **Test-Diff zuerst.** Wurden bestehende Tests geändert, abgeschwächt oder übersprungen? Das ist das wichtigste Warnsignal überhaupt (Modul 3).
2. **Nach Abwesenheiten suchen.** Gibt es einen Autorisierungscheck? Eine Ratenbegrenzung? Negative Tests? Eingabevalidierung? Was *fehlt* ist die eigentliche Frage.
3. **Den Diff lesen, nicht die Dateien** (Modul 1). Die fertige Datei sieht plausibel aus; der Diff zeigt die stillschweigend entfernte Fehlerbehandlung.
4. **Neue Abhängigkeiten prüfen.** Existiert das Paket wirklich? Wird es gepflegt? Brauchen wir es?
5. **Fachlogik gegen die Spec prüfen.** Nicht „tut es etwas", sondern „tut es das Richtige" (Modul 8).
6. **Migrationen und Konfiguration gesondert.** Beide sehen harmlos aus und haben die größten Konsequenzen (Modul 6).

## 6. AI als Verteidiger

Die andere Richtung ist real und unterschätzt: Modelle sind gut darin, bekannte Sicherheitsmuster zu erkennen — *wenn man gezielt danach fragt*.

Der wirksame Kniff: ein **separater Review-Durchlauf mit expliziter Sicherheitsrolle**, in einer eigenen Sitzung, ohne den Kontext der Implementierung. Der Prompt lautet nicht „prüf das mal", sondern etwa: *Prüfe diesen Diff auf fehlende Autorisierung, fehlende Eingabevalidierung, fehlende Ratenbegrenzung und unsichere Voreinstellungen. Liste, was fehlt, nicht was da ist.* Der fehlende Implementierungskontext ist dabei ein Feature — er durchbricht den Zirkelschluss zumindest teilweise.

Die Grenze: Das findet bekannte Muster. Es findet nicht, dass in deiner Fachlogik ein Sachbearbeiter der Region Nord keine Vorgänge der Region Süd sehen darf. Diese Art von Autorisierungsregel existiert nur in deinem Kopf und in der Spec.

## 7. Anti-Patterns

- Stilqualität als Hinweis auf Sorgfalt lesen
- Nur positive Testfälle schreiben
- Neue Abhängigkeiten ohne menschliche Freigabe
- Ein Agent, der nicht vertrauenswürdige Eingaben liest *und* Schreibrechte hat
- Prompt Injection durch Formulierungen im System-Prompt „lösen" wollen
- Review-Kapazität als unbegrenzt annehmen
- PR-Größe unbegrenzt lassen, weil Agenten schnell liefern
- Generierte Tests als Beleg für geprüften Code werten
- Sicherheitsprüfung im selben Kontext wie die Implementierung
- Ein geleaktes Secret durch Löschen statt durch Rotation „beheben" (Modul 1)

## 8. Glossar

Abwesenheitsfehler · Automation Bias · OWASP Top 10 · Authentifizierung vs. Autorisierung · Insecure Direct Object Reference · Negativer Test · Injection · SSRF · Unsichere Voreinstellung · SAST · Slopsquatting · Typosquatting · Transitive Abhängigkeit · SBOM · Dependency Review · Prompt Injection · Lethal Trifecta · Instruktion-Daten-Vermischung · Verification Debt · Zirkelschluss bei generierten Tests · Mutation Testing · Lizenz-Scan

## 9. Übungen

**Ü1 — Die Abwesenheit finden.** Lass einen Agenten einen Endpoint bauen, der eine Ressource anhand einer ID zurückgibt, mit Authentifizierung. Prüfe dann, ob eine Autorisierungsprüfung vorhanden ist. Schreibe anschließend selbst den negativen Test — und beobachte, ob er fehlschlägt.

**Ü2 — Negative Tests systematisch.** Nimm drei bestehende Endpoints deines Projekts und ergänze zu jedem positiven Test einen negativen. Notiere, wie viele davon bei der ersten Ausführung eine echte Lücke aufdecken.

**Ü3 — SAST einbauen.** Ergänze `bandit` oder `semgrep` in deiner Pipeline und lass es auf AI-generierten Code aus früheren Übungen laufen. Kategorisiere die Treffer nach echt und Fehlalarm.

**Ü4 — Den Stilbias prüfen.** Lass einen Agenten dieselbe Funktion zweimal schreiben, einmal mit expliziter Anweisung zu ausführlichen Docstrings und sauberer Struktur. Gib beide Varianten einem Kollegen zum Review, ohne zu sagen, worum es geht, und vergleiche die Gründlichkeit der Rückmeldungen.

**Ü5 — Lieferkette.** Lass einen Agenten eine Aufgabe lösen, die eine externe Bibliothek nahelegt. Prüfe für jedes vorgeschlagene Paket: Existiert es? Wann war der letzte Commit? Wie viele transitive Abhängigkeiten zieht es? Richte danach CODEOWNERS auf deine Abhängigkeitsdateien ein.

**Ü6 — Die Trifecta auf dein Setup anwenden.** Zeichne für jeden Agenten-Workflow in deinem Projekt auf, welche der drei Bedingungen erfüllt sind. Wo alle drei zutreffen: Welchen Faktor kannst du am billigsten entfernen?

**Ü7 — Sicherheitsreview im separaten Kontext.** Nimm einen deiner früheren AI-generierten PRs und lass ihn in einer frischen Sitzung mit explizitem Sicherheitsauftrag prüfen. Vergleiche die Funde mit dem, was dein normaler Review gefunden hatte.

**Ü8 — Verification Debt messen.** Erhebe für die letzten zwanzig PRs in einem Repo, auf das du Zugriff hast: durchschnittliche Diff-Größe, Zeit von Erstellung bis Freigabe, Anzahl der Review-Kommentare. Wenn Diff-Größe steigt und Kommentare pro Zeile sinken, hast du die Kurve gefunden.

Bleibt Modul 11: Skalierung im Team und Wirkungsmessung. Dort geht es um die Frage, die dir irgendwann gestellt wird — „bringt uns AI eigentlich etwas?" — und darum, sie mit Zahlen zu beantworten, die sich nicht austricksen lassen.
