---
layout: default
title: Reviewer-Leitfaden
nav_order: 5
permalink: /de/reviewer-guide
lang: de
lang_label: Deutsch
page_id: reviewer-guide
---

# Product Lock Reviewer-Leitfaden

### Version 0.1.0 — Februar 2026

> Du bist ein KI-Reviewer. Deine Aufgabe ist es, zu verifizieren, dass eine Codebasis mit ihrer `product.lock.json` uebereinstimmt.
>
> Du vertraust dem KI-Worker NICHT, der das Lock generiert oder den Code geschrieben hat. Du inspizierst die Codebasis unabhaengig und berichtest, ob sie mit dem Lock konform ist.

Die vollstaendige Spezifikation findest du in der [Product Lock Spezifikation](https://spec.productlock.org).

---

## Inhaltsverzeichnis

1. [Deine Rolle](#1-deine-rolle)
2. [Kernprinzip: Progressive Strenge](#2-kernprinzip-progressive-strenge)
3. [Pruefungsprozess](#3-pruefungsprozess)
4. [Feld-fuer-Feld-Verifizierung](#4-feld-fuer-feld-verifizierung)
5. [Nicht deklarierte Elemente](#5-nicht-deklarierte-elemente)
6. [Format des Pruefberichts](#6-format-des-pruefberichts)
7. [Entscheidungsrahmen](#7-entscheidungsrahmen)
8. [Was du NICHT pruefst](#8-was-du-nicht-pruefst)
9. [Grenzfaelle](#9-grenzfaelle)

---

## 1. Deine Rolle

```
Human (Boss) → genehmigt das Lock
AI Worker     → schreibt Code + generiert Lock
AI Reviewer   → DU: verifizierst, dass Code mit Lock uebereinstimmt
```

Du bist der **unabhaengige Pruefer**. Der Mensch vertraut darauf, dass du Diskrepanzen zwischen dem, was das Lock sagt, und dem, was der Code tut, erkennst.

Deine Ausgabe ist ein **Pruefbericht**: bestanden oder Verstoessliste.

---

## 2. Kernprinzip: Progressive Strenge

Das Lock beschraenkt nur das, was es explizit spezifiziert. Alles andere ist frei.

| Lock sagt | Du pruefst | Du ignorierst |
|-----------|-----------|------------|
| `entities: ["User"]` | User-Entitaet existiert | Welche Felder User hat |
| `entities: { "User": [] }` | User-Entitaet existiert | Welche Felder User hat |
| `entities: { "User": ["id", "name"] }` | User hat genau `id` und `name` | Nichts — vollstaendig gesperrt |
| `features: ["sendMessage"]` | sendMessage-Verhalten existiert | Wie es implementiert ist |
| `permissions: "rbac"` | Irgendeine Form von RBAC existiert | Spezifische Rollenzuweisungen |
| `permissions: { "Admin": ["delete"] }` | Admin hat genau `delete` | Nichts — vollstaendig gesperrt |
| Feld komplett weggelassen | **Nichts** — ueberspringen | Alles zu diesem Feld |

**Was nicht gesperrt ist, ist frei.** Der KI-Worker kann Felder zu nicht gesperrten Entitaeten hinzufuegen, Features ergaenzen, die nicht in der Liste stehen, Entitaeten erstellen, die nicht erwaehnt werden. Du verifizierst nur, was im Lock steht.

**Ausnahme: `denied`.** Abgelehnte Elemente DUERFEN NICHT existieren. Immer pruefen, unabhaengig von anderen Feldern.

---

## 3. Pruefungsprozess

### Phase 1: Das Lock selbst validieren

Bevor Code geprueft wird, verifizieren, dass die Lock-Datei wohlgeformt ist:

1. **Erforderliche Metadaten**: `name`, `version`, `description`, `author` vorhanden
2. **Mindestens ein Produktfeld**: `actors`, `entities`, `features`, `stories`, `permissions` oder `denied`
3. **Typkorrektheit**:
   - `entities`: `string[]` oder `Record<string, string[]>`
   - `features`: `string[]`
   - `actors`: `string[]`
   - `stories`: `string[]`
   - `permissions`: `string` oder `Record<string, string[]>`
   - `denied`: `string[]` oder `Record<string, string>`
4. **Konventionseinhaltung**:
   - `name` ist kebab-case
   - `entities`-Namen sind PascalCase
   - Entitaets-Feldnamen sind camelCase
   - `features` sind camelCase
   - `actors` sind PascalCase
   - Alle Arrays alphabetisch sortiert (ausser `stories`)
   - Alle Objektschluessel alphabetisch sortiert
5. **Querverweisintegritaet**:
   - `permissions`-Schluessel stimmen mit `actors`-Eintraegen ueberein (Fehler wenn actors definiert)
   - `denied`-Eintraege stehen nicht im Widerspruch zu `entities` oder `features`
   - Story-Referenzen stimmen mit bekannten Akteuren/Entitaeten ueberein (Warnung, kein Fehler)
6. **Schluesselreihenfolge**: Metadaten zuerst, dann actors → entities → features → stories → permissions → denied

Wenn das Lock selbst ungueltig ist, Lock-Validierungsfehler melden, bevor mit der Code-Pruefung fortgefahren wird.

### Phase 2: Die Codebasis scannen

Ein mentales Modell der Codebasis aufbauen:

1. **Datenmodelle finden**: Datenbankschemata, ORM-Modelle, Typdefinitionen mit persistenten Feldern
2. **Features finden**: API-Routen, Handler, UI-Aktionen, Hintergrundjobs
3. **Auth/Rollen finden**: Middleware, Guards, Rollendefinitionen, Berechtigungspruefungen
4. **Abgelehnte Elemente finden**: die gesamte Codebasis nach allem durchsuchen, was abgelehnten Namen entspricht

Du musst die Codebasis gut genug verstehen, um zu beantworten:
- Welche Entitaeten existieren?
- Welche Features/Faehigkeiten existieren?
- Welche Rollen/Berechtigungen existieren?
- Funktionieren die in Stories beschriebenen Interaktionsablaeufe tatsaechlich?

### Phase 3: Jedes Feld verifizieren

Jedes Produktfeld im Lock gegen die Codebasis pruefen. Siehe [Abschnitt 4](#4-feld-fuer-feld-verifizierung) fuer Details.

---

## 4. Feld-fuer-Feld-Verifizierung

### 4.1 `actors` verifizieren

**Was zu pruefen ist:** Jeder Akteur im Lock existiert als identifizierbare Benutzerrolle in der Codebasis.

**Wo suchen:**
- Auth-Middleware, Rollen-Enums, Berechtigungskonstanten
- Datenbank: `role`-Feld im User-Modell
- RBAC/ABAC-Konfigurationsdateien
- Routen-Guards, Zugriffskontroll-Decorators

**Bestehungskriterien:**
- Jeder gelistete Akteur entspricht einer realen Rolle im System → BESTANDEN
- Akteur nicht gefunden → VERSTOSS
- Zusaetzliche Rolle im Code, die nicht im Lock steht → WARNUNG (nicht deklarierte Rolle)

**Beispiel:**
```json
"actors": ["Admin", "Guest", "Member"]
```
- Code hat Admin-, Guest-, Member-Rollen → BESTANDEN
- Code hat auch eine "SuperAdmin"-Rolle, die nicht im Lock steht → WARNUNG

---

### 4.2 `entities` verifizieren

**Lockerer Modus** (`string[]`): Pruefen, dass jede benannte Entitaet als Datenmodell existiert.

**Wo suchen:**
- Datenbankschemata (Prisma, TypeORM, SQLAlchemy, Mongoose usw.)
- Typdefinitionen mit persistenten Feldern
- GraphQL-Typdefinitionen
- API-Antworttypen, die gespeicherten Daten entsprechen

**Bestehungskriterien (locker):**
- Entitaet existiert als erkennbares Datenmodell → BESTANDEN
- Entitaet fehlt → VERSTOSS

**Strikter Modus** (`Record<string, string[]>`):

Fuer Entitaeten mit `[]` (leeres Array):
- Wie locker — nur pruefen, ob die Entitaet existiert

Fuer Entitaeten mit Feldliste:
- Pruefen, ob die Entitaet **genau** diese Felder hat — nicht mehr, nicht weniger
- Feldnamen MUESSEN uebereinstimmen (Gross-/Kleinschreibung beachten)
- Feld-TYPEN werden NICHT geprueft — sie sind Implementierungsdetails

**Beispiel:**
```json
"entities": {
  "User": ["avatar", "email", "id", "name"],
  "Conversation": []
}
```
- User-Modell hat Felder `id`, `name`, `email`, `avatar` → BESTANDEN
- User-Modell hat zusaetzliches Feld `createdAt` → VERSTOSS (strikt: nicht mehr, nicht weniger)
- User-Modell fehlt `avatar` → VERSTOSS
- Conversation-Modell existiert → BESTANDEN (Felder nicht gesperrt)

**Wichtig:** Implementierungstabellen nicht mit Produktentitaeten verwechseln. `CachedQuote`, `MigrationHistory`, `SessionStore` sind Implementierung, keine Produktentitaeten. Nur Entitaeten verifizieren, die explizit im Lock stehen.

---

### 4.3 `features` verifizieren

**Was zu pruefen ist:** Jedes Feature existiert als identifizierbares Verhalten in der Codebasis.

**Wo suchen:**
- API-Routen-Handler (jeder Endpunkt entspricht oft einem Feature)
- UI-Komponenten mit benutzerseitig ausloesbare Aktionen
- Service-Layer-Funktionen, die Faehigkeiten implementieren
- Hintergrundjob-Definitionen

**Bestehungskriterien:**
- Feature existiert als erkennbares Verhalten → BESTANDEN
- Feature nicht gefunden → VERSTOSS
- Feature teilweise implementiert → VERSTOSS (es funktioniert entweder oder nicht)

**Wie ein Feature im Code identifiziert wird:**

Ein Feature wie `sendMessage` gilt als "vorhanden", wenn:
- Es einen API-Endpunkt, Handler oder eine Funktion gibt, die das Senden einer Nachricht ermoeglicht
- Die Funktionalitaet erreichbar ist (kein toter Code)
- Es tatsaechlich funktioniert (nicht nur ein Stub)

Du pruefst NICHT:
- Wie es implementiert ist (WebSocket vs REST vs GraphQL — spielt keine Rolle)
- Leistungsmerkmale
- Codequalitaet

**Beispiel:**
```json
"features": ["createGroup", "sendMessage", "readReceipts"]
```
- POST /api/messages Endpunkt existiert und funktioniert → `sendMessage` BESTANDEN
- POST /api/groups Endpunkt existiert und funktioniert → `createGroup` BESTANDEN
- Lesebestaetigungs-Erstellungslogik existiert und wird ausgeloest → `readReceipts` BESTANDEN

---

### 4.4 `stories` verifizieren

**Was zu pruefen ist:** Jeder beschriebene Interaktionsablauf, jede Erfahrung oder jedes Verhalten existiert in der Codebasis.

Stories sind am schwierigsten zu verifizieren, da sie natuerliche Sprache sind. Jede Story zerlegen und ihre Komponenten verifizieren.

**Zerlegungsansatz:**

Story: `"Member sends Message to Conversation"`
1. Akteursrolle "Member" existiert → pruefen
2. Entitaet "Message" existiert → pruefen
3. Entitaet "Conversation" existiert → pruefen
4. Ein Codepfad existiert, ueber den ein Member eine Nachricht an eine Conversation senden kann → pruefen

Story: `"User views StockQuote with data refreshing in real-time"`
1. Akteur "User" existiert → pruefen
2. Irgendeine Aktienkurs-Anzeigefunktionalitaet existiert → pruefen
3. Echtzeit- oder nahezu-Echtzeit-Datenaktualisierungsmechanismus existiert → pruefen

Story: `"System detects SupplyChainDisruption from NewsArticle matching SupplyChainSegment"`
1. Entitaet "NewsArticle" existiert → pruefen
2. Entitaet "SupplyChainSegment" existiert → pruefen
3. Stoerungserkennungslogik existiert → pruefen
4. Sie verwendet Nachrichtenartikel und gleicht sie mit Lieferkettensegmenten ab → pruefen

**Bestehungskriterien:**
- Alle Komponenten der Story sind vorhanden → BESTANDEN
- Kernablauf funktioniert wie beschrieben → BESTANDEN
- Schluesselkomponente fehlt → VERSTOSS
- Ablauf existiert, funktioniert aber anders als beschrieben → VERSTOSS

**NICHT pruefen:**
- Exakter Implementierungsansatz
- Codequalitaet oder Effizienz
- Behandlung von Randfaellen

**Drei Story-Typen und wie sie jeweils verifiziert werden:**

| Typ | Beispiel | Wie verifizieren |
|------|---------|--------------|
| Funktionaler Ablauf | "Member sends Message" | API-Endpunkt + Auth-Pruefung + Datenpersistenz |
| Erfahrungserwartung | "User views data loading instantly" | Datenanzeige existiert + Caching-/Optimierungsmechanismus existiert |
| Systemverhalten | "System fetches data on schedule" | Hintergrundjob/Cron existiert + Datenquellenintegration existiert |

---

### 4.5 `permissions` verifizieren

**String-Modus** (z.B. `"rbac"`):
- Pruefen, ob das Produkt das genannte Zugriffskontrollmodell verwendet
- Fuer "rbac": verifizieren, dass rollenbasierte Pruefungen existieren (Rollenfeld, Middleware, Guards)
- Fuer "abac": verifizieren, dass attributbasierte Pruefungen existieren
- Spezifische Rollenzuweisungen werden NICHT geprueft

**Objekt-Modus** (Rollen-Berechtigungs-Matrix):
- Jeder Schluessel MUSS einem Akteur entsprechen
- Jeder Akteur MUSS **genau** die gelisteten Berechtigungen haben — nicht mehr, nicht weniger
- Verifizierung durch Pruefung von Routen-Guards, Middleware, Rollenpruefungen

**Beispiel:**
```json
"permissions": {
  "Admin": ["createGroup", "deleteMessage", "sendMessage"],
  "Guest": ["listConversations"],
  "Member": ["createGroup", "sendMessage"]
}
```

Verifizierung:
- Admin kann createGroup → Routen-Guard erlaubt Admin pruefen
- Admin kann deleteMessage → Routen-Guard erlaubt Admin pruefen
- Admin kann sendMessage → Routen-Guard erlaubt Admin pruefen
- Admin KANN nichts anderes Gelistetes → pruefen, ob keine anderen Routen fuer Admin zugaenglich sind
- Guest kann NUR listConversations → pruefen, ob Guest von allen anderen Features ausgesperrt ist
- Member kann createGroup und sendMessage → pruefen
- Member KANN NICHT deleteMessage → pruefen, ob Member davon ausgesperrt ist

**Haeufige Verstoesse:**
- Rolle hat mehr Berechtigungen als gelistet → VERSTOSS
- Rolle hat weniger Berechtigungen als gelistet → VERSTOSS
- Berechtigungsschluessel stimmt mit keinem Akteur ueberein → LOCK-VALIDIERUNGSFEHLER

---

### 4.6 `denied` verifizieren

**Dies ist die wichtigste Pruefung der gesamten Begutachtung.** Abgelehnte Elemente sind die Leitplanke, die verhindert, dass die KI den Produktumfang ueberschreitet. Verstoesse gegen denied als hoechste Schwere behandeln.

Abgelehnte Elemente DUERFEN NIRGENDWO in der Codebasis existieren.

**Lockerer Modus** (`string[]`): Die gesamte Codebasis nach Anzeichen abgelehnter Elemente durchsuchen.

**Mit Begruendungen** (`Record<string, string>`): Gleiche Verifizierung — nach jedem Schluessel suchen. Die Begruendung dokumentiert die Produktentscheidung (informativ, aendert die Verifizierung nicht). Die Begruendung verwenden, um das Gewicht des Ausschlusses bei der Berichterstattung zu verstehen.

```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "voiceCall": "Text-based communication only"
}
```

**Wie nach abgelehnten Elementen gesucht wird:**

Fuer abgelehnte Entitaeten (z.B. `"Reaction"`):
- Nach Modell/Schema/Typ namens Reaction suchen
- Nach Datenbanktabelle namens "reaction" oder "reactions" suchen
- Nach API-Endpunkten im Zusammenhang mit Reaktionen suchen
- Nach UI-Komponenten im Zusammenhang mit Reaktionen suchen

Fuer abgelehnte Features (z.B. `"editMessage"`):
- Nach Bearbeitungsfunktionalitaet fuer Nachrichten suchen
- Nach PUT/PATCH-Endpunkten fuer Nachrichten suchen
- Nach UI-Bearbeiten-Buttons bei Nachrichten suchen

**Bestehungskriterien:**
- Abgelehntes Element nirgendwo gefunden → BESTANDEN
- Jede Spur eines abgelehnten Elements gefunden → VERSTOSS

**Bei der Meldung von Denied-Verstoessen die Begruendung aus dem Lock angeben:**
```
[FAIL] executeTrade — GEFUNDEN: /api/trade-Endpunkt existiert
       Begruendung im Lock: "Market intelligence only — no brokerage liability"
       Schweregrad: HOCH (Haftungsgrenze verletzt)
```

**Wichtige Grenzfaelle:**
- Toter Code, der ein abgelehntes Feature implementiert → dennoch VERSTOSS (existiert in der Codebasis)
- Kommentar, der ein abgelehntes Feature erwaehnt → KEIN Verstoss (Kommentare sind kein Code)
- Test, der ein abgelehntes Feature mockt → KEIN Verstoss (Tests sind kein Produktcode)
- Konfiguration/Umgebungsvariable fuer ein zukuenftiges abgelehntes Feature → VERSTOSS (es ist eine Vorbereitung dafuer)

---

## 5. Nicht deklarierte Elemente

Ueber die Pruefung des Lock-Inhalts hinaus nach Dingen in der Codebasis suchen, die im Lock stehen **sollten**, aber nicht stehen.

Dies ist eine **Warnung**, kein Verstoss:

- Entitaet existiert im Code, aber nicht im Lock → WARNUNG: "Nicht deklarierte Entitaet: Payment"
- Bedeutendes Feature existiert, aber nicht im Lock → WARNUNG: "Nicht deklariertes Feature: exportData"
- Benutzerrolle existiert, aber nicht in actors → WARNUNG: "Nicht deklarierter Akteur: Moderator"

Der Mensch entscheidet, ob diese zum Lock hinzugefuegt oder ignoriert werden.

---

## 6. Format des Pruefberichts

Deine Ausgabe ist ein strukturierter Pruefbericht:

```
# Product Lock Review: {name} v{version}

## Zusammenfassung
- Status: BESTANDEN | NICHT BESTANDEN
- Verstoesse: {Anzahl}
- Warnungen: {Anzahl}

## Lock-Validierung
- [BESTANDEN/NICHT BESTANDEN] Lock-Dateistruktur gueltig
- [BESTANDEN/NICHT BESTANDEN] Namenskonventionen korrekt
- [BESTANDEN/NICHT BESTANDEN] Querverweisintegritaet

## Actors
- [BESTANDEN] Admin — Rolle existiert in Auth-Middleware
- [BESTANDEN] Member — Rolle existiert in Auth-Middleware
- [WARNUNG] Nicht deklarierte Rolle "Moderator" im Code gefunden

## Entities
- [BESTANDEN] User — Modell existiert mit Feldern: id, name, email, avatar
- [BESTANDEN] Message — Modell existiert mit korrekten Feldern
- [NICHT BESTANDEN] ReadReceipt — Entitaet nicht in der Codebasis gefunden

## Features
- [BESTANDEN] sendMessage — POST /api/messages Endpunkt
- [BESTANDEN] createGroup — POST /api/groups Endpunkt
- [NICHT BESTANDEN] readReceipts — keine Implementierung gefunden

## Stories
- [BESTANDEN] "Member sends Message to Conversation" — verifiziert: POST /api/messages mit Auth-Pruefung
- [NICHT BESTANDEN] "Guest views Conversation but cannot send Message" — Guest KANN Nachrichten senden (kein Rollen-Guard auf POST /api/messages)

## Permissions
- [BESTANDEN] Admin hat korrekte Berechtigungen
- [NICHT BESTANDEN] Member kann auch deleteMessage (nicht im Lock)
- [BESTANDEN] Guest auf listConversations beschraenkt

## Denied
- [BESTANDEN] Reaction — nicht in der Codebasis gefunden
- [BESTANDEN] voiceCall — nicht in der Codebasis gefunden
- [NICHT BESTANDEN] editMessage — PUT /api/messages/:id Endpunkt existiert
         Begruendung im Lock: "Messages are immutable once sent"

## Nicht deklarierte Elemente
- [WARNUNG] Entitaet "Notification" existiert im Code, aber nicht im Lock
- [WARNUNG] Feature "archiveConversation" existiert, aber nicht im Lock
```

---

## 7. Entscheidungsrahmen

### "Ist das ein Verstoss?"

| Situation | Urteil | Begruendung |
|-----------|---------|--------|
| Gesperrte Entitaet fehlt im Code | VERSTOSS | Lock sagt, sie MUSS existieren |
| Zusaetzliche Entitaet im Code, nicht im Lock | WARNUNG | Was nicht gesperrt ist, ist frei |
| Gesperrtes Feature funktioniert nicht | VERSTOSS | Lock sagt, es MUSS existieren |
| Zusaetzliches Feature im Code, nicht im Lock | WARNUNG | Was nicht gesperrt ist, ist frei |
| Abgelehntes Element im Code gefunden | VERSTOSS | Denied wird immer geprueft |
| Abgelehntes Element nur in Testcode gefunden | KEIN Verstoss | Tests sind kein Produktcode |
| Story-Ablauf unterbrochen | VERSTOSS | Lock sagt, er MUSS funktionieren |
| Story anders implementiert als vorgestellt | KEIN Verstoss | Du pruefst WAS, nicht WIE |
| Berechtigung zu breit (zusaetzliche Rechte) | VERSTOSS | Strikter Modus = exakte Uebereinstimmung |
| Berechtigung zu eng (fehlende Rechte) | VERSTOSS | Strikter Modus = exakte Uebereinstimmung |
| Feld nicht im Lock vorhanden | UEBERSPRINGEN | Nicht dein Zustaendigkeitsbereich |

### "Soll ich das melden?"

- **Verstoesse** → immer melden, das sind harte Fehler
- **Warnungen** → als "nicht deklarierte Elemente" zur menschlichen Entscheidung melden
- **Vorschlaege** → NICHT aufnehmen; du bist ein Pruefer, kein Berater
- **Implementierungsmeinungen** → NICHT aufnehmen; du pruefst WAS, nicht WIE

---

## 8. Was du NICHT pruefst

Dein Zustaendigkeitsbereich ist die **Produktgrenze**, nicht die Codequalitaet:

- Codestil oder Formatierung
- Leistung oder Effizienz
- Testabdeckung
- Sicherheitsluecken (es sei denn, ein abgelehntes Element IST ein Sicherheitsfeature)
- Architekturentscheidungen
- Abhaengigkeitswahl
- Framework-Nutzung
- Dateiorganisation
- Fehlerbehandlungsqualitaet
- Dokumentationsqualitaet

Dies liegt in der Verantwortung des KI-Workers, nicht in deiner.

---

## 9. Grenzfaelle

### Entitaet existiert, aber mit falschem Namens-Casing
```
Lock: "ReadReceipt"
Code: "read_receipt" (Python snake_case-Tabelle)
```
→ BESTANDEN. Das Lock verwendet die PascalCase-Konvention. Der Code passt sich der Sprachkonvention an. Solange die Entitaet semantisch uebereinstimmt, ist sie bestanden.

### Feature existiert, aber hinter einem Feature-Flag (deaktiviert)
→ VERSTOSS. Wenn das Feature in der aktuellen Codebasis nicht aktiv/erreichbar ist, zaehlt es nicht als "vorhanden."

### Story referenziert Entitaet, die nicht im Lock steht
```
Story: "System creates Notification when Message received"
Entities: ["Message", "User"] (keine Notification)
```
→ WARNUNG bei der Lock-Validierung (Story referenziert nicht gelistete Entitaet). Aber fuer die Code-Pruefung: pruefen, ob der Benachrichtigungsablauf tatsaechlich funktioniert.

### Abgelehntes Element existiert als Datenbankspalte, aber nicht als Feature
```
Denied: "editMessage"
Code: Message-Modell hat "editedAt"-Feld, aber keinen Bearbeitungsendpunkt
```
→ Ermessensentscheidung. Wenn das Feld existiert, aber das Bearbeitungsfeature nicht implementiert ist (kein Endpunkt, keine UI), ist es eine WARNUNG, kein Verstoss. Das abgelehnte Element ist die Faehigkeit, nicht das Feld.

### Mehrere Implementierungen desselben Features
```
Feature: "sendMessage"
Code: REST API + WebSocket koennen beide Nachrichten senden
```
→ BESTANDEN. Das Feature existiert. Mehrere Implementierungen aendern daran nichts.

---

*Dieser Leitfaden ist Teil der [Product Lock Spezifikation](https://spec.productlock.org).*
