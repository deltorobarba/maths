# Modul 4: Code-Design & Architektur

Dieses Modul funktioniert anders als die drei davor. Bei Git, Lintern und Tests gab es überwiegend richtige Antworten. Hier gibt es fast nur Abwägungen. Genau deshalb ist es der Teil, der sich am wenigsten automatisieren lässt — und damit der Teil, in dem deine Arbeit auch in fünf Jahren noch gebraucht wird.

## 1. Wovon Architektur eigentlich handelt

Software heißt „soft", weil man sie ändern kann. Das ist ihr eigentlicher Wert gegenüber Hardware. Eine Software, die genau das Richtige tut, aber nicht mehr geändert werden kann, ist ein Wegwerfprodukt — denn die Anforderungen werden sich ändern. Immer.

Daraus folgt die Leitfrage. Sie lautet nie „ist das schön?" und selten „ist das schnell?". Sie lautet:

> **Was kostet es, das in sechs Monaten zu ändern — und wie sicher bin ich mir, dass sich genau das ändern wird?**

Der zweite Halbsatz ist der wichtigere und wird fast immer vergessen. Jede Designentscheidung ist eine **Wette auf die Zukunft**. Du machst etwas leicht änderbar, indem du etwas anderes schwerer änderbar machst. Wer das nicht als Wette begreift, sondern als Qualitätsfrage, baut Systeme, die in alle Richtungen flexibel und in keiner brauchbar sind.

## 2. Kopplung und Kohäsion — das Fundament

Fast jedes Designprinzip, das du je hören wirst, ist eine Ableitung aus diesem Begriffspaar.

**Kohäsion** — wie stark gehören die Dinge *innerhalb* einer Einheit zusammen? Hoch ist gut. Ein Modul `user.py`, das Passwort-Hashing, Rechnungsstellung und PDF-Export enthält, hat niedrige Kohäsion.

**Kopplung** — wie stark hängen Einheiten *voneinander* ab? Niedrig ist gut.

Der praktische Test für Kopplung ist bemerkenswert einfach und schlägt jedes Architekturdiagramm:

> **Wenn ich A ändere, muss ich dann B anfassen?**

Wenn ja, sind A und B gekoppelt — egal, was in der Dokumentation steht, egal, ob sie in verschiedenen Repos liegen, egal, ob ein Interface dazwischen ist.

Kopplung hat Grade, und die Reihenfolge lohnt sich zu kennen (von harmlos nach schlimm):

- **Datenkopplung** — B bekommt von A genau die Werte, die es braucht. Das ist der Normalfall und in Ordnung.
- **Stempelkopplung** — B bekommt ein ganzes Objekt, obwohl es zwei Felder braucht. Jetzt hängt B an der Struktur des Objekts.
- **Steuerkopplung** — A übergibt ein Flag, das B's Verhalten umschaltet. `render(data, is_pdf=True)`. Der Aufrufer muss das Innenleben kennen; meist sind es in Wahrheit zwei Funktionen.
- **Gemeinsame Kopplung** — beide greifen auf globalen Zustand zu. Jetzt hängen sie aneinander, ohne dass es irgendwo sichtbar wäre.
- **Inhaltskopplung** — A greift in die Interna von B. Der Härtefall.

Wenn du in einer Diskussion mit einem Begriff punkten willst, der Kopplung wirklich präzise macht: **Connascence**. Das Modell benennt, *worüber* zwei Stellen verbunden sind — über einen Namen, einen Typ, eine Bedeutung, eine Reihenfolge, ein Timing. Die Regel dazu lautet: Je weiter zwei Stellen im Code auseinanderliegen, desto schwächer muss die Form ihrer Verbindung sein. Zwei Zeilen in derselben Funktion dürfen über eine Ausführungsreihenfolge verbunden sein. Zwei Services über zwei Kontinente nicht.

## 3. Der Kernkonflikt: Kopplung gegen Duplikation

Hier steckt das am meisten missverstandene Prinzip der Softwareentwicklung.

**DRY heißt nicht „kein Code darf doppelt vorkommen."** Das Original lautet: *Jedes Stück Wissen soll eine einzige, eindeutige, autoritative Repräsentation im System haben.* Der Schlüsselbegriff ist **Wissen**, nicht Text.

Zwei Codestellen, die zufällig gleich aussehen, aber unterschiedliches Wissen kodieren, dürfen nicht zusammengelegt werden. Die Berechnung der Mehrwertsteuer und die Berechnung einer Provision sind vielleicht heute beide `betrag * satz`. Legst du sie zusammen, koppelst du zwei Dinge aneinander, die sich unabhängig voneinander ändern werden — und in einem halben Jahr steht ein Flag-Parameter in der gemeinsamen Funktion, dann zwei, dann eine Verzweigung.

Sandi Metz hat den Satz geprägt, der dir das für immer merkbar macht:

> **Duplikation ist billiger als die falsche Abstraktion.**

Der Grund: Duplikation kostet linear und ist leicht rückgängig zu machen. Eine falsche Abstraktion kostet doppelt — die Abstraktion selbst, plus den ständigen Aufwand, sie zu umgehen. Und sie ist schwer rückgängig zu machen, weil inzwischen sieben Stellen daran hängen.

Die praktische Regel heißt **Rule of Three**: Beim ersten Mal schreibst du es. Beim zweiten Mal duplizierst du es und zuckst zusammen. Erst beim dritten Mal siehst du das Muster deutlich genug, um zu wissen, *welche* Abstraktion die richtige ist.

Genau diese Urteilsfrage — sind diese zwei Dinge dasselbe oder sehen sie nur gleich aus? — ist etwas, das ein Agent nicht beantworten kann. Er sieht die syntaktische Ähnlichkeit. Er kennt die Fachdomäne nicht.

## 4. Abstraktion und ihre Kosten

Jede Abstraktion verschiebt Komplexität, sie eliminiert sie nicht. Sie macht den einen Fall einfacher und alle anderen schwerer.

**YAGNI** (You Aren't Gonna Need It) ist die Gegenkraft: Baue keine Flexibilität für Fälle, die du dir nur ausdenkst. Die überwiegende Mehrheit vorausschauend eingebauter Erweiterungspunkte wird nie benutzt — und die Änderung, die tatsächlich kommt, passt nicht in den Punkt, den du vorgesehen hattest.

Und ein Konzept, das dir Enttäuschungen erspart: **Das Gesetz der undichten Abstraktionen** (Joel Spolsky). Jede nicht-triviale Abstraktion leckt. Ein ORM verbirgt SQL — bis die Query langsam ist und du SQL verstehen musst. TCP verbirgt unzuverlässige Netzwerke — bis das Netzwerk so unzuverlässig ist, dass es durchschlägt. Die praktische Konsequenz: Du musst die Schicht unter deiner Abstraktion trotzdem verstehen. Abstraktionen sparen Tipparbeit, nicht Wissen.

## 5. Schichten und die Richtung der Abhängigkeiten

Das ist die zentrale Strukturidee, und sie ist gleichzeitig die am wenigsten intuitive.Der linke Pfeil ist intuitiv: Der HTTP-Controller ruft die Geschäftslogik auf, also hängt er von ihr ab.

Der rechte Pfeil ist die eigentliche Pointe. Naiv würde man sagen: „Mein Code benutzt Postgres, also hängt mein Code von Postgres ab." Genau das wird umgedreht. Das nennt sich **Dependency Inversion**, und der Mechanismus ist:

> **Das Interface gehört dem Aufrufer, nicht dem Implementierer.**

Die Domäne definiert, was sie braucht — `UserRepository` mit `get(id)` und `save(user)`. Die Infrastruktur implementiert das. Damit hängt Postgres von deiner Domäne ab, nicht umgekehrt.

```python
# domain/ports.py — die Domäne definiert ihren Bedarf
class UserRepository(Protocol):
    def get(self, user_id: UserId) -> User | None: ...
    def save(self, user: User) -> None: ...

# domain/service.py — kennt weder SQL noch HTTP
class Registration:
    def __init__(self, users: UserRepository) -> None:
        self._users = users

# infrastructure/postgres.py — hängt von der Domäne ab
class PostgresUserRepository:
    def get(self, user_id: UserId) -> User | None: ...
```

Diese Struktur heißt je nach Autor Hexagonal Architecture, Ports and Adapters oder Clean Architecture. Die Namen unterscheiden sich, das Prinzip ist dasselbe.

Zwei Prüffragen, an denen du in jeder Codebase in fünf Minuten erkennst, wie es um sie steht:

1. **Kann ich die Geschäftslogik testen, ohne einen HTTP-Server oder eine Datenbank zu starten?**
2. **Wenn ich `grep` nach `import psycopg` mache — taucht das in Dateien auf, die Geschäftslogik enthalten?**

Und hier schließt sich der Kreis zu Modul 3: **Schwer testbarer Code ist fast immer schlecht geschnittener Code.** Wenn du für einen Unit-Test sechs Dinge mocken musst, sagt dir der Test nicht, dass Testen mühsam ist — er sagt dir, dass deine Einheit sechs Abhängigkeiten hat. Testschmerz ist ein Designsignal, kein Testproblem. Das ist der Grund, warum ich TDD in Modul 8 als Designwerkzeug bezeichnet habe.

## 6. Dependency Injection, entzaubert

Klingt nach Framework, ist keins. Dependency Injection heißt bloß: **Abhängigkeiten hereinreichen, statt sie selbst zu erzeugen.**

```python
# ohne DI — die Abhängigkeit ist einbetoniert
class Registration:
    def __init__(self):
        self._users = PostgresUserRepository()   # nicht austauschbar

# mit DI — ein Konstruktorparameter, mehr nicht
class Registration:
    def __init__(self, users: UserRepository):
        self._users = users
```

Das ist alles. In Python brauchst du dafür kein Container-Framework; ein Konstruktorparameter und eine zentrale Stelle, die alles zusammensteckt (die „Composition Root", meist `main.py`), genügen völlig. Der Gewinn: Im Test übergibst du ein Fake statt einer Datenbank — ohne Patching-Magie, die an Modulpfaden klebt.

## 7. SOLID — ehrlich eingeordnet

Du wirst diesen Begriff hören, also solltest du ihn kennen. Aber nicht als Katechismus.

**S — Single Responsibility.** Die verbreitete Lesart „eine Klasse macht nur eine Sache" ist die falsche. Die ursprüngliche lautet: *Eine Klasse sollte nur einen Grund haben, sich zu ändern* — und ein Grund ist ein **Akteur**, eine Interessengruppe. Wenn die Buchhaltung und die Personalabteilung beide Änderungen an derselben Klasse auslösen können, ist das die Verletzung. Das ist im Kern eine Aussage über Kohäsion.

**O — Open/Closed.** Erweiterbar ohne Modifikation. Wurde in den 90ern über Vererbung gelöst, heute fast immer über Komposition und Strategien. In der Praxis eher ein Ideal als eine Regel.

**L — Liskov Substitution.** Der einzige mit einem harten Kriterium: Wo der Obertyp erwartet wird, muss jeder Untertyp funktionieren, ohne dass der Aufrufer etwas merkt. Die klassische Verletzung ist das Quadrat, das von Rechteck erbt — und dann `setWidth` und `setHeight` nicht mehr unabhängig unterstützen kann.

**I — Interface Segregation.** Viele schmale Interfaces statt eines breiten. In dynamischen Sprachen mit Duck Typing weit weniger relevant.

**D — Dependency Inversion.** Abschnitt 5. Der wichtigste der fünf.

Die ehrliche Einordnung: SOLID entstand im objektorientierten Kontext um 2000. LSP und DIP sind zeitlos, der Rest ist teilweise von der Sprachentwicklung eingeholt worden. Als Prüfliste beim Codelesen nützlich, als Doktrin schädlich — SOLID-Fetischismus erzeugt Codebasen mit fünfzehn Interfaces, die je eine Implementierung haben. Im Alltag tragen dich Kopplung, Kohäsion, gute Namen und expliziter Datenfluss deutlich weiter.

## 8. Zustand, Seiteneffekte, Idempotenz

Eine **reine Funktion** gibt bei gleicher Eingabe immer dieselbe Ausgabe und verändert nichts außerhalb ihrer selbst. Ein **Seiteneffekt** ist alles andere: schreiben, lesen, loggen, Netzwerk, Zeit, Zufall.

Reine Funktionen sind billiger, und zwar in jeder Dimension: trivial testbar (keine Fixtures, keine Mocks), trivial nachvollziehbar (kein versteckter Zustand), trivial parallelisierbar, trivial cachebar.

Daraus folgt das nützlichste Strukturmuster, das ich kenne — **funktionaler Kern, imperative Schale**: Entscheidungen und Berechnungen in reine Funktionen, I/O an den Rand. Lesen, dann rechnen, dann schreiben — statt Lesen und Rechnen zu verzahnen. Die Testbarkeit verbessert sich dadurch sprunghaft, ohne dass du irgendein Framework brauchst.

**Idempotenz** verdient einen eigenen Absatz, weil sie in Modul 6 wiederkommt. Eine Operation ist idempotent, wenn die mehrfache Ausführung dasselbe Ergebnis liefert wie die einmalige. `set_status("paid")` ist idempotent, `increment_counter()` nicht.

Warum das existenziell ist: In verteilten Systemen weißt du nie, ob eine Anfrage angekommen ist oder nur die Antwort verlorenging. Also wird wiederholt. Ist die Operation nicht idempotent, wird der Kunde zweimal belastet. Die Standardlösung ist ein Idempotenzschlüssel, den der Aufrufer mitschickt und der serverseitig deduplizieren lässt. Jede Retry-Logik, jede Message Queue, jedes „at least once"-Zustellungsversprechen setzt Idempotenz voraus.

## 9. Fehlerbehandlung ist eine Designentscheidung

Der Grundsatz: **Fange einen Fehler dort, wo du etwas Sinnvolles tun kannst** — nicht dort, wo er entsteht. In den meisten Schichten heißt „sinnvoll" schlicht: durchlassen.

Was in AI-generiertem Code überdurchschnittlich häufig auftaucht und was du sofort erkennen solltest:

```python
try:
    result = risky_operation()
except Exception:
    pass          # der Fehler ist jetzt unsichtbar
```

Das ist die schlimmste Zeile Code, die es gibt. Der Fehler verschwindet nicht, er wird nur unsichtbar — und taucht drei Schichten weiter als unerklärliches `None` wieder auf. Verwandt und fast so schlimm: `except Exception:` mit einem generischen Fallback-Wert.

Die Alternative zum Exception-Modell sind **Result-Typen**: Der Rückgabewert ist explizit entweder Erfolg oder Fehler, und der Aufrufer *muss* beides behandeln (Rust, Go, und in Python/TS über eigene Typen nachgebaut). Der Vorteil ist Sichtbarkeit in der Signatur — man sieht am Typ, dass etwas schiefgehen kann. Der Nachteil ist Rauschen. Beide Modelle sind vertretbar; entscheidend ist, dass du dich innerhalb eines Systems für eines entscheidest.

Und die Grundhaltung: **Fail fast.** Ungültiger Zustand soll sofort und laut scheitern, an der Stelle, an der er entsteht — Validierung am Systemrand, dann darf die Domäne davon ausgehen, dass ihre Daten stimmen.

## 10. Namen

Ein Name ist die dichteste verfügbare Form von Dokumentation. Er wird tausendmal gelesen und einmal geschrieben.

Der Maßstab, den ich dir empfehle, stammt aus Domain-Driven Design und heißt **Ubiquitous Language**: Der Code benutzt exakt die Begriffe der Fachdomäne. Wenn die Fachabteilung von einer „Police" spricht und dein Code von `Contract`, zahlst du bei jedem Gespräch, jedem Ticket und jedem Onboarding eine Übersetzungsgebühr — und irgendwann übersetzt jemand falsch.

Für Agenten hat das eine zusätzliche Dimension: Ein Modell schließt aus Namen auf Bedeutung. `data`, `result`, `handle`, `process`, `manager` sind für ein Modell nahezu informationsfrei — es muss den Rest der Datei lesen, um zu erraten, was gemeint ist. Präzise Namen sind, wie schon die Typannotationen aus Modul 2, unmittelbar wirksames Context Engineering.

## 11. Refactoring als eigene Disziplin

Die Definition (Fowler) ist eng und die Enge ist der Punkt: **Refactoring ist die Änderung der internen Struktur, ohne das äußere Verhalten zu ändern.** Wenn sich das Verhalten ändert, ist es kein Refactoring, sondern eine Änderung.

Die wichtigste Regel dazu — Kent Beck nennt sie die **zwei Hüte**:

> Du trägst entweder den Refactoring-Hut oder den Feature-Hut. Nie beide gleichzeitig.

Der Grund ist praktisch und hart: Ein Diff, der Umbenennungen und neue Logik vermischt, ist nicht reviewbar. Der Reviewer kann die zwei Zeilen echte Änderung in 400 Zeilen Verschiebung nicht finden — er wird durchwinken. Und wenn es hinterher kaputt ist, weißt du nicht, ob es am Refactoring oder am Feature lag. Also: getrennte Commits, idealerweise getrennte PRs, Refactoring zuerst.

Als Vokabular für die Diagnose gibt es die **Code Smells**. Zwei davon sind besonders lehrreich, weil sie das exakte Gegensatzpaar zu Abschnitt 2 bilden:

- **Shotgun Surgery** — eine fachliche Änderung zwingt dich, an zwölf Stellen kleine Anpassungen zu machen. Diagnose: zu hohe Kopplung, das Wissen ist verstreut.
- **Divergent Change** — ein Modul muss aus fünf völlig verschiedenen Gründen geändert werden. Diagnose: zu niedrige Kohäsion, das Modul enthält Unzusammengehöriges.

Weitere, die du im Sprachgebrauch haben solltest: Long Method, Feature Envy (eine Methode interessiert sich mehr für die Daten eines anderen Objekts als für die eigenen), Primitive Obsession (alles ist `str` und `dict`, statt dass es einen Typ `EmailAddress` gäbe), Data Clumps (dieselben drei Parameter reisen immer gemeinsam — meist ein ungeborenes Objekt).

## 12. Technische Schuld — präziser als üblich

Die Metapher stammt von Ward Cunningham und meinte ursprünglich etwas Enges: Man liefert bewusst eine unfertige Struktur aus, um früher zu lernen, und räumt anschließend auf. Wie ein Kredit: legitim, wenn man ihn bewusst aufnimmt und zurückzahlt.

Martin Fowlers Quadrant macht daraus ein brauchbares Diagnosewerkzeug — zwei Achsen: *bewusst/unbewusst* und *besonnen/leichtfertig*.

- Bewusst und besonnen: „Wir liefern jetzt ohne Cache aus, Ticket ist angelegt, Frist steht." Das ist echte Schuld und völlig legitim.
- Unbewusst und leichtfertig: „Was ist eine Schichtenarchitektur?" Das ist keine Schuld, das ist ein Wissensdefizit.
- Unbewusst und besonnen: „Jetzt, wo es fertig ist, sehen wir, wie es hätte aussehen sollen." Das ist der Normalfall guten Lernens.

Die Zinsen sind real und messbar: Jede künftige Änderung in dem Bereich kostet mehr. Deshalb sollte Schuld sichtbar sein — als Ticket, nicht als Bauchgefühl.

**Und die AI-Wendung:** Die Aufnahme technischer Schuld ist dramatisch billiger geworden, die Tilgung nicht. Man kann heute in einem Nachmittag Codemengen erzeugen, für deren Aufräumen man Wochen braucht. Das verschiebt das Gleichgewicht, und zwar in eine unangenehme Richtung.

## 13. Design Docs und ADRs

Schreiben ist das Denkwerkzeug des Design Loop. Der Satz, den ich dir mitgeben würde: *Wenn du es nicht aufschreiben kannst, hast du es nicht verstanden.* Die Hälfte aller schlechten Architekturentscheidungen zerfällt beim Versuch, sie in drei Absätzen zu begründen.

Ein **ADR** (Architecture Decision Record) ist bewusst klein — eine Seite, vier Abschnitte:

- **Kontext**: Welche Situation, welche Zwänge?
- **Entscheidung**: Was wird gemacht?
- **Konsequenzen**: Was wird dadurch leichter, was schwerer? (Beides — nur Vorteile aufzuzählen ist ein Zeichen für unfertiges Denken.)
- **Alternativen und warum verworfen**

Der eigentliche Wert liegt im letzten Punkt. In zwei Jahren fragt jemand — vielleicht du selbst — „warum haben wir eigentlich nicht X gemacht?", und die Antwort steht schon da, mitsamt der Randbedingung, die inzwischen weggefallen sein könnte. ADRs liegen versioniert im Repo unter `docs/adr/`, weil sie mit dem Code altern.

Für größere Vorhaben kommt vor die Implementierung ein **Design Doc** oder **RFC**: Problem, Ziele und Nicht-Ziele, betrachtete Ansätze, gewählter Ansatz, Risiken, offene Fragen. Das ist der „Deliberate"-Schritt aus Modul 0 — und der Ort, an dem Fehler zehnmal billiger sind als in der Implementierung.

## 14. Die AI-Verbindung

Jetzt der Teil, der für dich beruflich entscheidend ist.

**Ein Agent optimiert lokal.** Er löst die Aufgabe, die vor ihm liegt, mit dem Kontext, den er gerade hat. Was er strukturell nicht sieht:

- dass die dritte ähnliche Funktion ein Signal für eine fehlende Abstraktion ist
- dass zwei ähnliche Funktionen fachlich *nicht* zusammengehören und deshalb getrennt bleiben müssen
- dass es die gebrauchte Hilfsfunktion schon gibt — sie lag nur nicht im Kontextfenster
- dass sein Vorgehen dieselbe Konvention verletzt, die drei Module weiter etabliert ist

Daraus ergibt sich ein wiedererkennbares Profil von AI-Code: lokal sauber, global inkonsistent. Duplikation, weil jede Sitzung isoliert ist. Gleichzeitig Überabstraktion, sobald man danach fragt — eine Factory für zwei Fälle, ein Strategy-Pattern für eine `if`-Verzweigung. Defensive `try/except` an Stellen, an denen nichts schiefgehen kann. Und eine langsame Konventionsdrift über die Codebase, weil jede Sitzung ihre eigenen Vorlieben mitbringt.

**Der Hebel dagegen ist der wichtigste Satz dieses Moduls: Architektur ist Kontext.**

Ein Agent, der eine klare Schichtenstruktur vorfindet, in der Domänencode keine Infrastruktur importiert, produziert automatisch besseren Code — nicht weil er die Regel kennt, sondern weil das Muster in dem, was er liest, konsistent ist. Präzise Typen, sprechende Namen, konsequente Struktur und explizite Designregeln in `AGENTS.md` wirken direkt auf die Qualität der Ausgabe. Die Investition in Struktur zahlt sich damit zweimal aus: einmal für Menschen, einmal für Modelle. Das ist einer der wenigen Punkte, an denen sich klassisches Engineering und AI-Produktivität nicht widersprechen, sondern verstärken.

**Und die Arbeitsteilung.** Deine Notizen beschreiben ein Orchestrator-Worker-Muster, in dem ein großes Modell plant und ein schnelles implementiert. Das ist ökonomisch richtig, aber die entscheidende Ergänzung fehlt darin: Der eigentliche Orchestrator bist du. Die Design-Loop-Entscheidungen — ist diese Abstraktion die richtige Wette, gehören diese zwei Dinge zusammen, welchen Preis zahlen wir für diese Flexibilität — sind genau die Urteilsfragen, für die kein Modell die Verantwortung übernehmen kann, weil es die Zukunft deiner Fachdomäne nicht kennt.

Zum Refactoring durch AI: mechanisch ausgezeichnet. Umbenennen, Extrahieren, Signaturen ändern, ein Muster über vierzig Dateien durchziehen — dafür ist es ein hervorragendes Werkzeug, und Modul 3 hat dir das Sicherheitsnetz dafür gegeben. Die Entscheidung, *ob* und *wohin* refactored wird, ist die eigentliche Arbeit, und die bleibt bei dir.

## 15. Anti-Patterns

- Abstrahieren beim zweiten Vorkommen statt beim dritten
- Flag-Parameter, die das Verhalten einer Funktion umschalten
- Geschäftslogik, die SQL, HTTP oder Dateipfade kennt
- Interfaces mit genau einer Implementierung, die nie eine zweite bekommen wird
- Fehler stillschweigend schlucken
- Refactoring und Feature im selben Commit
- Design-Entscheidungen, deren Begründung nur im Kopf einer Person existiert
- Anämische Domäne: Klassen ohne Verhalten, nur Datenhalter, und die gesamte Logik in „Service"-Klassen
- Ein Agent darf großflächig umstrukturieren, ohne dass ein Mensch die Zielstruktur festgelegt hat

## 16. Glossar

Kopplung (Daten-, Stempel-, Steuer-, gemeinsame, Inhalts-) · Kohäsion · Connascence · DRY · Rule of Three · Die falsche Abstraktion · YAGNI · Leaky Abstraction · Schichtenarchitektur · Ports and Adapters / Hexagonal / Clean Architecture · Dependency Inversion · Dependency Injection · Composition Root · SOLID (SRP, OCP, LSP, ISP, DIP) · Reine Funktion · Seiteneffekt · Functional Core, Imperative Shell · Idempotenz · Idempotenzschlüssel · Fail Fast · Result-Typ · Ubiquitous Language · Refactoring · Zwei Hüte · Code Smell (Shotgun Surgery, Divergent Change, Feature Envy, Primitive Obsession, Data Clumps) · Technische Schuld · Fowler-Quadrant · ADR · Design Doc / RFC

## 17. Übungen

**Ü1 — Kopplung messen.** Nimm ein Modul aus deinem Projekt und beantworte für jede seiner Abhängigkeiten: Wenn sich die ändert, muss ich hier anfassen? Notiere für jede die Kopplungsform aus Abschnitt 2. Suche mindestens einen Flag-Parameter und löse ihn in zwei Funktionen auf.

**Ü2 — Die falsche Abstraktion rückbauen.** Finde in deinem oder fremdem Code eine gemeinsame Funktion mit mehr als zwei Flag- oder Modus-Parametern. Zerlege sie in getrennte Funktionen und beobachte, ob der Code insgesamt länger, aber verständlicher wird.

**Ü3 — Abhängigkeitsrichtung prüfen.** Führe die beiden Prüffragen aus Abschnitt 5 auf deinem Projekt aus. Wenn Punkt 2 Treffer liefert: Ziehe genau eine davon hinter ein Protocol und injiziere sie. Schreibe dann einen Test der Geschäftslogik, der ohne Datenbank läuft.

**Ü4 — Funktionaler Kern.** Nimm eine Funktion, die Lesen, Rechnen und Schreiben vermischt. Zerlege sie in eine reine Berechnungsfunktion und eine dünne Schale. Vergleiche, wie viele Zeilen Testcode du vorher und nachher für dieselbe Abdeckung brauchst.

**Ü5 — Ein ADR schreiben.** Für eine echte Entscheidung in deinem Projekt, eine Seite, mit mindestens zwei verworfenen Alternativen und ehrlichen Nachteilen der gewählten Option. Wenn dir keine Nachteile einfallen, hast du die Entscheidung nicht verstanden.

**Ü6 — Das AI-Profil erkennen.** Lass einen Agenten in drei getrennten Sitzungen drei verwandte Features bauen, ohne ihm Konventionen vorzugeben. Analysiere danach das Ergebnis auf die Muster aus Abschnitt 14: Duplikation, ignorierte vorhandene Hilfsfunktionen, Konventionsdrift, überflüssige Verteidigungsschichten. Schreibe daraus die ersten Designregeln für deine spätere `AGENTS.md`.

**Ü7 — Architektur als Kontext belegen.** Wiederhole Ü6, aber diesmal mit klarer Schichtenstruktur, vollständigen Typannotationen und drei expliziten Designregeln im Kontext. Vergleiche die Ergebnisse. Das ist der empirische Beleg für den zentralen Satz aus Abschnitt 14 — und ein ziemlich gutes Argument, wenn dich jemand fragt, warum du Zeit in Struktur steckst.

Damit ist Phase 1 abgeschlossen — das Handwerk steht. Weiter ginge es mit Phase 2: Modul 5, CI/CD und Quality Gates, wo aus deinen lokalen Prüfungen verbindliche Tore werden.
