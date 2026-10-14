# UI/UX-Spec — PayForMe (Fork `payforme-cat`)

Quelle der Wahrheit für Look&Feel/Interaktion der Fork-Änderungen. Fachlogik/API stehen in
`docs/concept.md`. Nicht zutreffende Abschnitte sind als "n/a" markiert, nicht gelöscht.
Verhältnis zu GitHub Spec Kit: siehe `../templates/CLAUDE.template.md`.

## 1. Scope

- Plattform: iOS (SwiftUI), iPhone primär, iPad lauffähig
- Betroffen: Bill-Formular (Kategorie-Picker, Vorschlag), Bill-Liste (Kategorie-Anzeige, besteht
  schon), Einstellungen (neuer Schalter), Projekt-Verwaltung (Kategorien nur lesen)
- Plattform-Design-System: ja — Apple HIG / SwiftUI-Standardkomponenten. Kein eigenes Theme.

## 2. Design-Tokens

Plattform-eigenes System vorhanden → **nutzen, nicht duplizieren.** Keine eigenen Werte erfinden.

| Kategorie | Verwendete Plattform-Tokens |
|---|---|
| Spacing | SwiftUI-Standard (`Form`, `List`, `VStack` ohne explizite Zahl); falls nötig bestehende Werte der Nachbar-Views übernehmen |
| Radius | System-Standard von `Form`/`List`/`Button` |
| Schatten | keine neuen |
| Farbe | System-Farben (`.primary`, `.secondary`, `.accentColor`); Kategoriefarbe aus Server (`CospendTag.color`) nur als dezenter Akzent, falls überhaupt — Default: nicht anzeigen (upstream zeigt sie auch nicht) |
| Typografie | Dynamic-Type-Textstile (`.body`, `.caption`, `.footnote`) |
| Motion | SwiftUI-Standard (`withAnimation(.default)`); Dauer nicht überschreiben |

## 3. Layout-Grundraster & Navigationsmodell

- Navigation (besteht): `TabView` mit **Bills**, **Members**, **Projects** (Zahnrad-Icon).
- Screens, die sich ändern:
  - **Bill-Formular** (`BillDetailView`): Abschnitt "Category / Payment mode" erscheint, sobald
    `project.categories` nicht leer ist. Neu: bei aktivem Auto-Modus wird beim Tippen des Titels
    die vorgeschlagene Kategorie im Picker vorausgewählt.
  - **Einstellungen**: ⚠ OFFEN — die App hat noch keinen Settings-Screen. Vorschlag (Annahme,
    nicht abgestimmt): ein Abschnitt "Categories" in der Projekt-Tab-Ansicht (`ProjectList`) mit
    einem Toggle "Suggest category from title". Alternative: eigener Settings-Tab. Entscheidung
    mit dem Nutzer treffen, dann hier eintragen.
- Zustände (jede betroffene View muss sie behandeln):
  - Loading: Tags kommen asynchron nach den Bills → Picker erscheint erst, wenn Tags da sind; kein
    Spinner nötig (Formular bleibt benutzbar)
  - Empty: keine Kategorien am Projekt (Stock-iHateMoney) → Picker und Auto-Vorschlag ausgeblendet,
    Toggle in den Einstellungen deaktiviert mit Hinweistext "Server unterstützt keine Kategorien"
  - Error: Tag-Laden schlägt fehl → still wie bisher (`.empty`), kein Alert
  - Offline: wie bisher

## 4. Komponenten-Inventar (lebendes Dokument, Pflicht-Update)

| Datenmuster | Komponente/Funktion | Datei | Verwendet in |
|---|---|---|---|
| Kategorie (Icon + Name) | `CospendTag.label` | `Model/CospendTag.swift` | BillCell (Untertitel), BillDetailView (Picker) |
| Kategorie am Bill auflösen | `Project.category(for:)` | `Model/Project.swift` | BillCell |
| Picker-ID absichern (unbekannte ID → 0) | `Project.listedCategoryId(_:)` | `Model/Project.swift` | BillDetailViewModel |
| Titel → Kategorie-Vorschlag | `CategoryClassifier.classify` (neu) | `Util/CategoryClassifier.swift` | BillDetailViewModel |
| Text-Normalisierung/Token-Match | `TextMatchUtils` (neu) | `Util/TextMatchUtils.swift` | CategoryClassifier |

Regel: Kategorie-Darstellung nie von Hand aus `icon`+`name` bauen — immer `label`.

## 5. Interaktions- & Motion-Regeln

| Trigger | Übergang | Dauer/Easing |
|---|---|---|
| Vorschlag setzt Picker-Auswahl | Auswahl wechselt ohne Hervorhebung; kein Pop-up | System-Default |
| Nutzer wählt selbst eine Kategorie | Auto-Vorschlag ist ab jetzt für dieses Formular aus | sofort |
| Toggle in Einstellungen | Standard-Toggle-Animation | System-Default |

Architektur-Hinweis: SwiftUI diffed den View-Baum → Übergänge sind ohne Sonderlösung möglich;
n/a für DOM-Rebuild-Problematik.

## 6. Barrierefreiheit & Eingabe

- Touch-Ziele: System-Standard (≥ 44 pt) — keine eigenen Mini-Buttons
- Kontrast: System-Farben (Light/Dark automatisch)
- VoiceOver: Picker-Label "Category"; Toggle mit Erklärtext (`.accessibilityHint`)
- Dynamic Type: keine festen Schriftgrößen
- Eingabemodi: Touch; keine Gesten nötig

## 7. UX-Best-Practices-Checkliste

- [ ] Systemzustand sichtbar: Nutzer sieht, welche Kategorie gewählt ist (Picker zeigt Wert)
- [ ] Konsistenz: Kategorie überall über `label`
- [ ] Fehler verhindern: Vorschlag nur setzen, wenn die ID im Projekt existiert (`listedCategoryId`)
- [ ] Wiedererkennen: Kategorie des zuletzt gelernten gleichen Titels wird vorgeschlagen
- [ ] Nutzer behält Kontrolle: manuelle Wahl hat immer Vorrang, Schalter standardmäßig AUS
- [ ] Minimalismus: kein zusätzlicher Dialog; Vorschlag = vorbelegter Picker
- [ ] Hilfetexte in Nutzersprache, keine rohen API-Fehler

## 8. Definition of Done (Gate vor "fertig")

- [ ] Jeder Trigger aus §5 verhält sich wie spezifiziert
- [ ] Keine frei erfundenen Spacing-/Farbwerte (§2)
- [ ] Kategorie-Anzeige nutzt `label` (§4)
- [ ] Loading/Empty/Error aus §3 behandelt (v. a. Stock-iHateMoney ohne Kategorien)
- [ ] Neue Strings in allen sechs Sprachen
- [ ] Unit-Tests für Klassifizierer, Tag-Decoding (iHateMoney-Fork-Antwort), Bill-Request mit
  `categoryid`; bestehende Tests grün
- [ ] §7 durchgegangen
- [ ] Standardverhalten ohne Opt-in unverändert (Schalter AUS)

## 9. Änderungs-Log

- 2026-10-14 — Erstfassung aus Vorlage; Settings-Ort noch offen.
