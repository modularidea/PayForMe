# PayForMe (Fork `payforme-cat`)

Fork von [InteractionEngineer/PayForMe](https://github.com/InteractionEngineer/PayForMe) — iOS-Client
für Nextcloud Cospend & iHateMoney. Ziel des Forks: Kategorie-Support für den iHateMoney-Kategorie-
Fork (`ihatemoney-cat`) plus eine optionale Auto-Kategorie-Zuordnung (Titel → Kategorie), per
Schalter in den Einstellungen. Am Ende steht ein Pull Request an Upstream — Änderungen müssen
daher klein, upstream-konform und ohne Fork-spezifische Abhängigkeiten bleiben.
Fachlogik: `docs/concept.md`. UI/UX: `docs/ui-spec.md` (VOR dem ersten UI-Code lesen/erweitern).

## Referenzprojekte (nur lesen, nichts dort ändern)

- `../ihatemoney-cat` — Server-Fork. API-Vertrag: `categoryid` an Bills, `GET/POST/PUT/DELETE
  /api/projects/<id>/categories`, Projekt-Info enthält `categories`, `paymentmodes` und `features`
  (`categoryid categories paymentmodes settle repeat`) → Feature-Erkennung.
- `../obsidian/obsidian-ihm-plugin/src/categorize/` — Vorlage der Auto-Kategorie-Logik
  (`classifier.ts`, `text-match-utils.ts`, `default-categories.ts`, `cospend-category-map.ts`).
  Dort ist es ein Port von haushub (`bill_category_service.dart`) — Verhalten soll identisch bleiben.
- `../templates/` — Vorlagen für CLAUDE.md und ui-spec.

## Stack

- Swift, SwiftUI + Combine, Xcode-Projekt (`PayForMe.xcodeproj`), Tests in `PayForMeTests`
  (XCTest), Fastlane für Release/Screenshots (nicht anfassen).
- Runtime-Deps: nur die bestehenden (SlickLoadingSpinner etc.). Keine neuen Dependencies für den
  PR — die Klassifizierung ist reines Swift ohne Libraries.

## Projektstruktur

```
PayForMe/Model/        Bill, Project, CospendTag (Kategorie/Zahlart), Person, Server
PayForMe/Services/     NetworkService (HTTP/Combine), ProjectManager (State), StorageService
PayForMe/Util/         Helfer; NEU: Kategorie-Klassifizierer (siehe Ownership)
PayForMe/Views/        SwiftUI-Views je Feature-Ordner (BillList, BillDetail, Balance, Projects)
PayForMe/Strings/      Lokalisierung (en, de, cs, es, fr, ru) — jeder neue String in ALLEN
PayForMeTests/         Unit-Tests (Model, Network-Requests, JSON-Decoding, ViewModels)
docs/                  concept.md, ui-spec.md, todos.md, session_current.md
```

## Ownership

- `Model/CospendTag.swift` → Kategorie-/Zahlart-Datentyp (gilt für Cospend UND iHateMoney-Fork)
- `Services/NetworkService.swift` → alle HTTP-Aufrufe; `loadTagsPublisher` ist aktuell Cospend-only
  (`guard project.backend == .cospend`) — hier kommt die iHateMoney-Fork-Erkennung hin
- `Util/` (neu: `TextMatchUtils.swift`, `CategoryClassifier.swift`) → reine Logik, KEIN UI, KEIN
  Netzwerk, testbar ohne Mocks
- `Views/BillDetail/` → Kategorie-Picker + Vorschlag beim Tippen des Titels
- `docs/concept.md` → Fachlogik-Spezifikation
- `docs/ui-spec.md` → UI/UX — lebendes Dokument, §4 (Komponenten-Inventar) bei jeder neuen
  wiederverwendeten Komponente aktualisieren
- `CLAUDE.md` → nach jeder Session aktualisieren (State-Abschnitt)

## Git

- Remotes: `origin` = `modularidea/PayForMe` (Fork), `upstream` = `InteractionEngineer/PayForMe`
- Nie direkt auf main committen — immer Branch (`feat/<slug>`, `fix/<slug>`, `refactor/<slug>`)
- PR-Branches von `upstream/main` abzweigen. `CLAUDE.md` und `docs/` sind Fork-intern und dürfen
  NICHT im PR landen → Docs auf `main` des Forks halten, Feature-Branches nur mit App-Code.
- Commit-Format: `feat: <summary ≤50 chars>` + Leerzeile + `Why: <ein Satz>`
- Commit-Autor: GitHub-Username `modularidea <266259136+modularidea@users.noreply.github.com>`, nie
  ein Klarname
- Commit + push nach jeder abgeschlossenen Phase
- PR erst nach Bestätigung durch den Nutzer öffnen (außenwirksam)

## Conventions

1. Swift 5 strict; explizite Typen an Modulgrenzen, kein `!` außer in Previews/Tests
2. Upstream-Stil beibehalten (Einrückung, MVVM mit `…ViewModel`, Combine-Publisher im
   `NetworkService`) — kein Refactoring nebenbei
3. Kommentare erklären WARUM, nicht WAS
4. Nur anfassen was der Task erfordert — kein Silent Cleanup
5. Unbekanntes Problem entdeckt → erwähnen, nicht fixen
6. Tests für reine Logik-Module (Klassifizierer, JSON-Decoding, Request-Parameter) — kein
   Framework-Mock nötig wo vermeidbar
7. Keine Passwörter/Tokens/Server-URLs in Logs, Tests oder Commits
8. **Design-Tokens statt Magic Numbers** (`docs/ui-spec.md` §2): hier = SwiftUI-/System-Werte
   (Standard-Spacing, `.font(.body)`, System-Farben), keine frei erfundenen Pixelwerte
9. **Komponenten-Zwang** (`docs/ui-spec.md` §4): Kategorie-Darstellung (Icon+Name) kommt IMMER aus
   `CospendTag.label` bzw. der dort registrierten Komponente, nie von Hand zusammengesetzt
10. **Motion ist Teil der Spec** (`docs/ui-spec.md` §5): Übergänge mitplanen
11. Neue Strings → in allen sechs `Strings/*.lproj/Localizable.strings` anlegen
12. Feature-Schalter standardmäßig AUS, damit Upstream-Verhalten ohne Opt-in unverändert bleibt

## Tooling

```bash
# Build + Tests (Simulator-Name ggf. anpassen)
xcodebuild -project PayForMe.xcodeproj -scheme PayForMe \
  -destination 'platform=iOS Simulator,name=iPhone 16' test
```

## Workflow

**Spec → Clarify → Read → Plan → Execute** — nie überspringen.

- **Spec**: vor UI-Code `docs/ui-spec.md` konsultieren/erweitern (Layout, Zustände, Komponenten,
  Motion). Reine Logikarbeit ohne UI: entfällt.
- **Clarify**: Repro (Bug) oder Spec-Lücke (Feature) klären, Fragen gebündelt in einer Nachricht.
- **Read**: grep zuerst. Bei erstmals genutzter externer API die Doku lesen (iHateMoney-Fork-API
  siehe `../ihatemoney-cat/README.md`); Gotchas in `docs/concept.md` festhalten.
- **Plan**: kurz posten (Scope, Schritte, Dateien), `docs/todos.md` updaten. Kein Code vor
  Bestätigung bei Multi-File-Änderungen.
- **Execute**: Plan befolgen, kein Scope Creep. Vor "fertig": Definition of Done aus
  `docs/ui-spec.md` §8 + Tests laufen lassen. Fertig → `docs/session_current.md` abhaken.

## Anti-Loop / Token Budget

- Kein Full-Dir-Scan — grep vor open, nur task-relevante Dateien
- Read mit `offset`+`limit` wenn Zeilenbereich bekannt
- `/compact` wenn Kontext >50%
- Stopp nach Task — keine ungebetenen Extras
- Keine Subagents, außer wenn unumgänglich

## Doc Maintenance

| Datei | Wann updaten |
|-------|-------------|
| `docs/concept.md` | Fachlogik-/API-Änderung, Gotchas aus dem Read-Schritt (lebendig) |
| `docs/ui-spec.md` | Neues UI-Pattern/Komponente, Token-Änderung, neuer Motion-Trigger (§4, §9) |
| `docs/todos.md` | Task eingereiht → add; erledigt → remove |
| `docs/session_current.md` | Start: Plan; während: abhaken; Ende: leeren |

## Wichtige Invarianten

- Bills tragen **nur `categoryid` (Int)**. `0`/`nil` = keine Kategorie. Negative IDs = Cospends
  globale Kategorien (Grocery −1, Bar/Party −2, Rent −3, Bill −4, Excursion/Culture −5, Health −6,
  Shopping −10, Restaurant −12, Accommodation −13, Transport −14, Sport −15); positive = eigene
  Projektkategorien. `−11` (Reimbursement) ist KEINE Kategorie.
- `Bill.paramsFor(.iHateMoney)` sendet aktuell KEIN `categoryid` — nur Cospend tut das. Beim
  iHateMoney-Fork darf `categoryid` nur gesendet werden, wenn der Server `categories` in
  `features` meldet; Stock-iHateMoney würde das Feld ggf. ablehnen/ignorieren.
- Stock-iHateMoney hat keine Kategorien (nur der Fork) → ohne Feature-Flag bleibt die UI wie
  bisher (Picker ist ohnehin nur sichtbar, wenn `project.categories` nicht leer ist).
- `Project` ist eine Klasse; Änderungen an `categories` sind erst nach Neu-Publizieren in
  `ProjectManager.currentProject` für SwiftUI sichtbar (siehe `loadTags`).
- Server-Antwort für `categories` kann Dictionary ODER Array sein (`CospendProjectTags.decodeList`
  deckt beides ab) — beibehalten.
- Auto-Kategorie-Reihenfolge (identisch zum Obsidian-Plugin): 1. exakter gelernter Treffer
  (neuester `updatedAt` gewinnt), 2. Fuzzy-Treffer (Overlap-Koeffizient ≥ 0,5, Zahl-Tokens
  vorher entfernt), 3. Schlüsselwörter (längste Phrase zuerst, wortgrenzensicher), 4. keine
  Kategorie. Normalisierung: lowercase, ä/ö/ü/ß → ae/oe/ue/ss, Nicht-Alphanumerisches → Leerzeichen.
- Vorschlag überschreibt NIE eine vom Nutzer manuell gewählte Kategorie.

## State (Stand 2026-10-14)

- Fork `modularidea/PayForMe` angelegt und nach `payforme-cat/` geklont, `upstream` gesetzt.
- Upstream hat bereits Kategorie-Anzeige/-Picker für **Cospend** (PR #103). Für **iHateMoney**
  fehlt: Tag-Laden (`loadTagsPublisher` guard), `categoryid` im Bill-Request, Feature-Erkennung.
- Noch nichts implementiert, nichts per Test/Build verifiziert. Nächster Schritt:
  `docs/concept.md` + `docs/ui-spec.md` mit dem Nutzer abstimmen (Offene Fragen), dann Plan.
