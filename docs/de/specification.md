---
layout: default
title: Spezifikation
nav_order: 2
permalink: /de/specification
lang: de
lang_label: Deutsch
page_id: specification
---

# Product Lock Spezifikation

### Version 0.1.0 — Februar 2026

> Eine Produktgrenzen-Spezifikation fuer Menschen und KI.
>
> product.lock.json beschreibt, was ein Softwareprodukt **ist** und **nicht ist**.
> Sie beschreibt nicht, wie das Produkt gebaut wird.

Die Schluesselwoerter "MUSS", "DARF NICHT", "SOLLTE", "SOLLTE NICHT" und "KANN" in diesem Dokument sind wie in [RFC 2119](https://tools.ietf.org/html/rfc2119) beschrieben zu interpretieren.

---

## Inhaltsverzeichnis

1. [Einfuehrung](#1-einfuehrung)
2. [Schnellstart](#2-schnellstart)
3. [Designprinzipien](#3-designprinzipien)
4. [Dateiformat](#4-dateiformat)
5. [Metadatenfelder](#5-metadatenfelder)
6. [Produktgrenzenfelder](#6-produktgrenzenfelder)
7. [Progressive Strenge](#7-progressive-strenge)
8. [Konventionen](#8-konventionen)
9. [Validierungsregeln](#9-validierungsregeln)
10. [Rollen](#10-rollen)
11. [Lebenszyklus](#11-lebenszyklus)
12. [FAQ](#12-faq)
13. [Referenzen](#13-referenzen)
14. [Zukuenftige Erweiterungen](#14-zukuenftige-erweiterungen)

---

## 1. Einfuehrung

### 1.1 Problem

KI kann inzwischen den Grossteil des Codes in einem Softwareprodukt schreiben. Aber Code-Generierung ohne Grenzen schafft ein neues Problem: **Scope Drift**. KI fuegt Funktionen hinzu, die niemand angefordert hat, erstellt Entitaeten, die nicht existieren sollten, und baut Faehigkeiten, die Produktgrenzen ueberschreiten.

In der traditionellen Entwicklung werden Produkte natuerlich durch menschliche Zeit und Aufwand begrenzt. Mit KI faellt diese Beschraenkung weg. Die Produktgrenze muss explizit definiert werden.

### 1.2 Loesung

`product.lock.json` ist eine maschinenlesbare, von Menschen ueberpruefbare Datei, die definiert, was ein Softwareprodukt enthaelt und was es nicht enthalten darf. Sie dient als Vertrag zwischen drei Parteien:

- **Mensch** — genehmigt die Grenze
- **KI-Worker** — baut innerhalb der Grenze
- **KI-Reviewer** — verifiziert Code gegen die Grenze

### 1.3 Geltungsbereich

Product Lock definiert ausschliesslich die **Produktebene**:

| Im Geltungsbereich | Ausserhalb des Geltungsbereichs |
|----------|-------------|
| Wer das Produkt nutzt (actors) | Routen, Endpunkte |
| Welche Daten es speichert (entities) | Abhaengigkeiten, Pakete |
| Was es kann (features) | Framework-Entscheidungen |
| Wie Dinge interagieren (stories) | Dateistruktur |
| Wer was tun darf (permissions) | Deployment-Konfiguration |
| Was es nicht haben darf (denied) | Code-Stil, Patterns |

### 1.4 Formatstrategie

**JSON ist die Quelle der Wahrheit. Markdown ist die gerenderte Ansicht.**

`product.lock.json` ist das kanonische Format — deterministisch, schema-validierbar, mit nur einer Schreibweise.

Wenn es einem Menschen zur Genehmigung vorgelegt wird, wird das Lock als Markdown fuer bessere Lesbarkeit gerendert. Der Mensch muss niemals rohes JSON lesen.

```
AI Worker schreibt JSON  →  AI Reviewer liest JSON  →  Mensch liest Markdown
```

---

## 2. Schnellstart

### 2.1 Minimales Beispiel

Das einfachste gueltige Lock:

```json
{
  "name": "todo-app",
  "version": "0.1.0",
  "description": "Simple to-do list application",
  "author": "kim",

  "entities": ["Todo", "User"],
  "features": ["completeTodo", "createTodo", "deleteTodo"]
}
```

Vier Metadatenfelder und mindestens ein Produktgrenzenfeld. Das ist alles.

### 2.2 Typisches Beispiel

Ein vollstaendigeres Lock mit allen Produktgrenzenfeldern:

```json
{
  "name": "chat-system",
  "version": "1.0.0",
  "description": "Real-time chat with group conversations and read receipts",
  "author": "kim",

  "actors": ["Admin", "Guest", "Member"],

  "entities": {
    "Conversation": [],
    "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
    "ReadReceipt": [],
    "User": ["avatar", "email", "id", "name"]
  },

  "features": ["createGroup", "listConversations", "readReceipts", "sendMessage"],

  "stories": [
    "Member sends Message to Conversation",
    "Member creates Conversation as Group and invites other Members",
    "When Member reads Message, system creates ReadReceipt",
    "Admin deletes any Message from Conversation",
    "Guest views Conversation but cannot send Message"
  ],

  "permissions": {
    "Admin": ["createGroup", "deleteMessage", "listConversations", "removeMember", "sendMessage"],
    "Guest": ["listConversations"],
    "Member": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
  },

  "denied": {
    "Reaction": "Keep messaging simple, no emoji reactions",
    "deleteAccount": "Account lifecycle managed by admin only",
    "editMessage": "Messages are immutable once sent",
    "videoCall": "Text-based communication only",
    "voiceCall": "Text-based communication only"
  }
}
```

### 2.3 Lock als Spezifikation

Ein Lock ist portabel. Jemand teilt sein `chat-system`-Lock; man gibt es einer KI:

> "Baue dieses Produkt in Python mit FastAPI und SQLAlchemy."

Gleiches Lock, anderer Stack. Produktgrenze identisch.

---

## 3. Designprinzipien

### 3.1 Produkt, nicht Code

Lock beschreibt Produktgrenzen, nicht technische Implementierung. Keine Routen, keine Abhaengigkeiten, keine Framework-Entscheidungen. Zwei Produkte, die mit verschiedenen Stacks gebaut werden, koennen dasselbe Lock teilen.

### 3.2 Progressive Strenge

Mehr spezifizieren, mehr erzwingen. Ein Feld weglassen, und die KI entscheidet frei. `entities: ["User"]` schreiben, und der Reviewer prueft nur, dass User existiert. `entities: { "User": ["id", "name"] }` schreiben, und der Reviewer prueft die exakten Felder. Das Lock erzwingt genau das, was angegeben wird, nicht mehr.

### 3.3 Uebersichtlich

Ein Mensch SOLLTE in der Lage sein, die Struktur des Locks zu ueberfliegen und eine Genehmigen/Ablehnen-Entscheidung zu treffen, ohne Code zu lesen. Einfache Produkte brauchen Sekunden; komplexe Produkte brauchen Minuten. In jedem Fall ist das Ueberfliegen eines Locks um Groessenordnungen schneller als Code-Review.

### 3.4 Sprachunabhaengig

Dasselbe Lock funktioniert fuer TypeScript, Python, Go, Java oder jede andere Sprache. Das Lock verwendet Namenskonventionen auf Produktebene (PascalCase-Entitaeten, camelCase-Features), die KI-Worker bei der Codegenerierung an die Konventionen der Zielsprache anpassen.

### 3.5 Lock als Spezifikation

Ein Lock ist eine teilbare Produktspezifikation. Das Lock einer anderen Person zu erhalten entspricht dem Erhalt ihrer Produktanforderungen. Es kann versioniert, verglichen und geteilt werden wie jede andere Spezifikation.

### 3.6 Die Grenze ist das, was man ausschliesst

In der KI-Aera der Entwicklung ist es wichtiger zu definieren, was ein Produkt NICHT tun darf, als zu definieren, was es tut. KI kann unbegrenzt Funktionen, Entitaeten und Verhaltensweisen hinzufuegen. Der einzige Weg, den Produktumfang zu wahren, ist, Ausschluesse explizit zu deklarieren. Das Feld `denied` ist die Leitplanke.

---

## 4. Dateiformat

- Die Datei MUSS `product.lock.json` heissen
- Die Datei MUSS gueltiges JSON sein ([RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259))
- Die Datei MUSS UTF-8-Kodierung verwenden
- Die Datei SOLLTE im Projektstammverzeichnis platziert werden
- Eine optionale `product.lock.md` KANN als menschenlesbare gerenderte Ansicht generiert werden

---

## 5. Metadatenfelder

| Feld | Typ | Erforderlich | Beschreibung |
|-------|------|----------|-------------|
| `$schema` | string | Nein | Schema-URL fuer Editor-Validierung und Autovervollstaendigung |
| `name` | string | **Ja** | Produktbezeichner in kebab-case |
| `version` | string | **Ja** | Produktversion, SOLLTE [semver](https://semver.org/) folgen |
| `description` | string | **Ja** | Einzeilige Produktbeschreibung |
| `author` | string | **Ja** | Wer dieses Lock genehmigt hat |
| `license` | string | Nein | Lizenzbezeichner (z.B. `"MIT"`, `"UNLICENSED"`) |
| `keywords` | string[] | Nein | Tags fuer Auffindbarkeit beim Teilen von Locks |
| `private` | boolean | Nein | Wenn `true`, ist das Lock nicht zum oeffentlichen Teilen gedacht |

Diese Felder folgen absichtlich den [package.json](https://docs.npmjs.com/cli/v10/configuring-npm/package-json)-Konventionen.

**Beispiel:**
```json
{
  "$schema": "https://productlock.org/schema/v1.json",
  "name": "chat-system",
  "version": "1.0.0",
  "description": "Real-time chat with group conversations and read receipts",
  "author": "kim",
  "license": "MIT",
  "keywords": ["chat", "group-messaging", "realtime"],
  "private": false
}
```

---

## 6. Produktgrenzenfelder

Alle Produktgrenzenfelder sind **optional**. Ein weggelassenes Feld bedeutet, dass die KI frei entscheidet und der Reviewer es nicht prueft.

Produktgrenzenfelder MUESSEN in dieser Reihenfolge erscheinen, wenn vorhanden:

```
actors → entities → features → stories → permissions → denied
```

Dies folgt einem konzeptionellen Ablauf: Wer nutzt es → welche Daten → welche Aktionen → wie sie interagieren → wer was darf → was ausgeschlossen ist.

---

### 6.1 `actors`

**Zweck:** Die Personen definieren, die das Produkt nutzen. Dies sind Benutzerrollen, keine Code-Entitaeten.

**Format:** `string[]`

**Beispiel:**
```json
"actors": ["Admin", "Guest", "Member"]
```

**Regeln:**
- Eintraege MUESSEN PascalCase sein
- Das Array MUSS alphabetisch sortiert sein
- Wenn weggelassen, entscheidet die KI alle Benutzerrollen

**Gegenbeispiel:**
```json
"actors": ["admin", "UserService", "AuthMiddleware"]
```
`admin` ist nicht PascalCase. `UserService` und `AuthMiddleware` sind Code-Entitaeten, keine Benutzerrollen.

---

### 6.2 `entities`

**Zweck:** Das Datenmodell definieren — was das Produkt speichert.

**Lockerer Modus** — Array von Namen. Der Reviewer prueft nur, ob die Entitaeten existieren.

```json
"entities": ["Conversation", "Message", "ReadReceipt", "User"]
```

**Strikter Modus** — Objekt mit Feldlisten. Der Reviewer prueft Entitaeten UND ihre Felder.

```json
"entities": {
  "Conversation": [],
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "ReadReceipt": [],
  "User": ["avatar", "email", "id", "name"]
}
```

**Regeln:**
- Entitaetsnamen MUESSEN PascalCase sein
- Feldnamen MUESSEN camelCase sein
- Leeres Array `[]` bedeutet, die Entitaet MUSS existieren, aber Felder sind nicht gesperrt
- Nicht-leeres Array bedeutet, diese Felder MUESSEN existieren — nicht mehr, nicht weniger
- Felder sind nur Namen, keine Typen (Typen sind Implementierungsdetails)
- Arrays und Schluessel MUESSEN alphabetisch sortiert sein

**Gegenbeispiel:**
```json
"entities": {
  "CachedQuote": ["symbol", "price", "cachedAt"],
  "User": ["id", "name"]
}
```
`CachedQuote` ist ein Implementierungsdetail (Cache-Tabelle), keine Produktentitaet. Die Produktanforderung, die sie erfuellt, SOLLTE stattdessen als Story erfasst werden:
```json
"stories": ["User views StockQuote with data refreshing in real-time"]
```

**Schluesselregel:** Entitaeten beantworten "Was speichert das Produkt?" Wenn etwas Infrastruktur ist (Caches, Queues, temporaere Tabellen, Migrationslogs), gehoert es nicht in entities. Erfasse stattdessen den Produktbedarf als Story.

---

### 6.3 `features`

**Zweck:** Produktfaehigkeiten definieren — was das Produkt kann.

**Format:** `string[]`

**Beispiel:**
```json
"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

**Regeln:**
- Features MUESSEN camelCase sein
- Features SOLLTEN dem Format Verb + Substantiv folgen (`sendMessage`, `createGroup`, `viewDashboard`)
- Das Array MUSS alphabetisch sortiert sein
- Keine Sub-Features — flach halten
- Jedes Feature MUSS in der Codebasis eigenstaendig identifizierbar sein

**Granularitaetsleitfaden:**

| Zu breit | Gut | Zu granular |
|-----------|------|-----------|
| `manageMessages` | `sendMessage`, `deleteMessage` | `validateMessageLength` |
| `handleAuth` | `login`, `register` | `hashPassword` |

Features sind das, was ein **Produktmanager** auflisten wuerde, nicht was ein **Entwickler** auflisten wuerde.

**Gegenbeispiel:**
```json
"features": ["hashPassword", "validateEmail", "generateJwt"]
```
Dies sind Implementierungsdetails. Die Produktfeatures sind `login`, `register`, `resetPassword`.

---

### 6.4 `stories`

**Zweck:** Interaktionsablaeufe, Erfahrungserwartungen und Systemverhalten definieren. Stories sind die Erzaehlung des Produkts — sie verbinden Akteure, Entitaeten und Features zu sinnvollen Sequenzen.

**Format:** `string[]`

**Beispiel:**
```json
"stories": [
  "Member sends Message to Conversation",
  "Member creates Conversation as Group and invites other Members",
  "When Member reads Message, system creates ReadReceipt",
  "Admin deletes any Message from Conversation",
  "Guest views Conversation but cannot send Message"
]
```

**Drei Arten von Stories:**

| Typ | Zweck | Beispiel |
|------|---------|---------|
| Funktionaler Ablauf | Was passiert, wenn jemand das Produkt nutzt | `"Member sends Message to Conversation"` |
| Erfahrungserwartung | Was der Benutzer wahrnimmt (nicht-funktionale Anforderungen) | `"User views Dashboard with data loading instantly"` |
| Systemverhalten | Was das System autonom tut | `"System fetches NewsArticle from multiple sources on schedule"` |

**Regeln:**
- Jede Story MUSS ein Satz sein
- Stories MUESSEN mit einem Akteursnamen (PascalCase, passend zu `actors`) oder `"System"` beginnen
- Stories MUESSEN Entitaeten mit exaktem PascalCase-Namen referenzieren
- Stories MUESSEN Praesens und Aktiv verwenden
- Stories MUESSEN beschreiben, WAS passiert, nicht WIE es implementiert wird
- Stories werden nach **erzaehlerischem Ablauf** geordnet, NICHT alphabetisch (dies ist die einzige Ausnahme von der alphabetischen Sortierregel)

**Gegenbeispiele:**
```
"Messages are sent via WebSocket"          ← Implementierungsdetail (WebSocket)
"Use Redis to cache stock quotes"          ← Implementierungsdetail (Redis)
"The user should be able to send messages" ← Spezifikationssprache, keine Story
"member sends message to conversation"     ← falsche Gross-/Kleinschreibung
```

**Implementierung-zu-Story-Konvertierung:** Wenn etwas wie eine Entitaet wirkt, aber eigentlich Infrastruktur ist, erfasse die Produktanforderung, die es erfuellt, als Story:

| Implementierungsdetail | Story |
|----------------------|-------|
| CachedQuote-Tabelle | `"User views StockQuote with data refreshing in real-time"` |
| Rate-Limiter-Middleware | `"System rate limits API requests per IP address"` |
| Embedding-Vektorspeicher | `"System searches knowledge base by semantic similarity"` |

---

### 6.5 `permissions`

**Zweck:** Zugriffskontrolle definieren — wer was tun darf.

**Lockerer Modus** — nur der Modellname. Die KI implementiert Details frei.

```json
"permissions": "rbac"
```

Gueltige Modellnamen: `"rbac"`, `"abac"`, `"acl"`.

**Strikter Modus** — Rollen-Berechtigungs-Matrix. Der Reviewer prueft, ob jede Rolle genau diese Berechtigungen hat.

```json
"permissions": {
  "Admin": ["createGroup", "deleteMessage", "listConversations", "removeMember", "sendMessage"],
  "Guest": ["listConversations"],
  "Member": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
}
```

**Regeln:**
- Bei Objekt: Schluessel MUESSEN mit `actors`-Eintraegen uebereinstimmen
- Bei Objekt: Werte MUESSEN Teilmengen von `features` sein (oder feature-aehnliche Aktionen)
- Schluessel und Wert-Arrays MUESSEN alphabetisch sortiert sein
- System-exklusive Features (z.B. `fetchNews`, `enrichData`) erscheinen NICHT in Berechtigungen — sie sind nicht vom Benutzer aufrufbar
- Wenn weggelassen, entscheidet die KI die gesamte Zugriffskontrolle

**Gegenbeispiel:**
```json
"permissions": {
  "admin": ["send-message"]
}
```
`admin` ist nicht PascalCase (MUSS zu actors passen). `send-message` ist nicht camelCase (MUSS zu features passen).

---

### 6.6 `denied`

**Zweck:** Explizite Ausschluesse definieren — was das Produkt NICHT haben darf.

**Dies ist das wichtigste Feld in der KI-Aera der Entwicklung.**

In der traditionellen Entwicklung werden Produkte natuerlich durch menschliche Zeit und Aufwand begrenzt. Mit KI gibt es keine natuerliche Grenze — KI kann unbegrenzt Funktionen, Entitaeten und Verhaltensweisen hinzufuegen. Der einzige Weg, den Produktumfang zu wahren, ist explizit zu deklarieren, was NICHT existieren darf. Andere Felder definieren, was das Produkt IST. `denied` definiert, was das Produkt NICHT IST. Ohne `denied` hat die KI keine Leitplanke.

**Lockerer Modus** — flache Liste.

```json
"denied": ["Reaction", "editMessage", "voiceCall"]
```

**Mit Begruendungen** (empfohlen) — Eintraege mit Erklaerung. Begruendungen helfen dem Reviewer, das Gewicht jedes Ausschlusses zu verstehen, und helfen dem Menschen, fundierte Entscheidungen zu treffen.

```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "Reaction": "Intentionally excluded to keep messaging simple",
  "voiceCall": "Text-based communication only"
}
```

**Regeln:**
- Der Reviewer MUSS pruefen, dass abgelehnte Elemente NICHT in der Codebasis existieren
- Wenn gefunden, ist es ein Verstoss
- Denied-Schluessel MUESSEN PascalCase (wenn entitaetsaehnlich) oder camelCase (wenn feature-aehnlich) sein
- Schluessel MUESSEN alphabetisch sortiert sein
- Begruendungen sind informativ — sie aendern das Verifizierungsverhalten nicht

**Was in denied gehoert:**

| Kategorie | Beispiel | Begruendung |
|----------|---------|--------|
| Produktumfang | `"voiceCall"` | "Nur textbasierte Kommunikation" |
| Haftungsgrenze | `"executeTrade"` | "Keine Brokerhaftung" |
| Scope-Leitplanke | `"Reaction"` | "Messaging einfach halten" |

**Was NICHT in denied gehoert:**
- Features, die noch nicht gebaut wurden (das ist nur das Backlog, keine Produktentscheidung)
- Implementierungsansaetze (`"kein Redis verwenden"` — das ist WIE, nicht WAS)
- Temporaere Einschraenkungen, die in der naechsten Version hinzugefuegt werden

**Gegenbeispiel:**
```json
"denied": ["exportData", "multiLanguage", "mobileApp", "paymentBilling"]
```
Dies sind Features, die noch nicht gebaut wurden, keine absichtlichen Produktausschluesse. Frage: "Wenn eine KI dies versehentlich hinzufuegen wuerde, waere es ein Produktverstoss?" Wenn nein, gehoert es nicht in denied.

---

## 7. Progressive Strenge

Das Lock erzwingt genau das, was angegeben wird. Nicht mehr.

| Was man schreibt | Was der Reviewer prueft |
|----------------|------------------------|
| Feld komplett weggelassen | Nichts — KI entscheidet frei |
| `"entities": ["User"]` | User-Entitaet existiert |
| `"entities": { "User": [] }` | User-Entitaet existiert (wie oben) |
| `"entities": { "User": ["id", "name"] }` | User hat genau `id` und `name`, nicht mehr, nicht weniger |
| `"features": ["sendMessage"]` | sendMessage-Verhalten existiert in der Codebasis |
| `"stories": [...]` | Jeder beschriebene Ablauf existiert in der Codebasis |
| `"permissions": "rbac"` | Irgendeine Form von RBAC existiert |
| `"permissions": { "Admin": ["delete"] }` | Admin hat genau diese Berechtigungen |
| `"denied": ["Reaction"]` | Reaction existiert NIRGENDWO |
| `"denied": { "Reaction": "reason" }` | Gleiche Pruefung, Begruendung dokumentiert die Entscheidung |

**Regel: Was nicht gesperrt ist, ist frei.** KI KANN Felder zu nicht gesperrten Entitaeten hinzufuegen, Features ergaenzen, die nicht in der Liste stehen, Entitaeten erstellen, die nicht erwaehnt werden. Der Reviewer prueft nur, was im Lock steht.

**Ausnahme: `denied`.** Abgelehnte Elemente werden immer auf Abwesenheit geprueft, unabhaengig von anderen Feldern. Dies ist die Leitplanke, die verhindert, dass die KI den Produktumfang ueberschreitet.

---

## 8. Konventionen

Diese Konventionen stellen sicher, dass jedes Lock gleich aussieht, unabhaengig davon, wer — oder welche KI — es generiert.

### 8.1 Benennung

| Feld | Konvention | Beispiel |
|-------|-----------|---------|
| `name` | kebab-case | `"chat-system"` |
| `actors` | PascalCase | `"Admin"`, `"Member"` |
| `entities` | PascalCase | `"User"`, `"ReadReceipt"` |
| Entitaetsfelder | camelCase | `"senderId"`, `"createdAt"` |
| `features` | camelCase | `"sendMessage"`, `"createGroup"` |
| `permissions`-Schluessel | PascalCase | `"Admin"`, `"Guest"` |
| `permissions`-Werte | camelCase | `"deleteMessage"` |
| `denied`-Schluessel | PascalCase oder camelCase | `"Reaction"`, `"editMessage"` |
| `denied`-Werte | Freitext | `"Text-based only"` |
| `keywords` | Kleinbuchstaben | `"chat"`, `"realtime"` |

Das Lock definiert **Produktnamen**, keine Codenamen. Bei der Codegenerierung passen KI-Worker die Benennung an die Konventionen der Zielsprache an (z.B. `sendMessage` im Lock wird zu `send_message` in Python).

### 8.2 Reihenfolge

Alle Arrays und Objektschluessel MUESSEN alphabetisch sortiert sein.

```json
"actors": ["Admin", "Guest", "Member"]

"entities": {
  "Conversation": [],
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "User": ["avatar", "email", "id", "name"]
}

"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

**Ausnahme:** `stories` werden nach erzaehlerischem Ablauf geordnet, nicht alphabetisch. Die Reihenfolge erzaehlt eine Geschichte.

### 8.3 Schluesselreihenfolge

Schluessel auf oberster Ebene MUESSEN in dieser Reihenfolge erscheinen:

```
$schema → name → version → description → author → license → keywords → private
→ actors → entities → features → stories → permissions → denied
```

Metadaten zuerst, dann Produktgrenzenfelder in konzeptioneller Reihenfolge.

### 8.4 Stories-Format

- Ein Satz pro Story
- Beginnt mit einem Akteursnamen oder `"System"`
- Referenziert Entitaeten mit exaktem PascalCase-Namen
- Praesens, Aktiv
- Drei Typen: **funktionale Ablaeufe**, **Erfahrungserwartungen**, **Systemverhalten**

---

## 9. Validierungsregeln

Eine gueltige `product.lock.json` MUSS alle folgenden Regeln erfuellen:

| # | Regel | Schweregrad |
|---|------|----------|
| 1 | `name`, `version`, `description`, `author` MUESSEN vorhanden sein | Fehler |
| 2 | Mindestens ein Produktgrenzenfeld MUSS vorhanden sein | Fehler |
| 3 | `entities` MUSS `string[]` oder `Record<string, string[]>` sein | Fehler |
| 4 | `features` MUSS `string[]` sein | Fehler |
| 5 | `actors` MUSS `string[]` sein | Fehler |
| 6 | `stories` MUSS `string[]` sein | Fehler |
| 7 | `permissions` MUSS `string` oder `Record<string, string[]>` sein | Fehler |
| 8 | `denied` MUSS `string[]` oder `Record<string, string>` sein | Fehler |
| 9 | `permissions`-Schluessel MUESSEN mit `actors`-Eintraegen uebereinstimmen (wenn beide definiert) | Fehler |
| 10 | `denied`-Eintraege DUERFEN NICHT in `entities` oder `features` erscheinen | Fehler |
| 11 | Alle Arrays und Objektschluessel MUESSEN alphabetisch sortiert sein (ausser `stories`) | Fehler |
| 12 | `name` MUSS kebab-case sein; `entities` MUSS PascalCase sein; `features` MUSS camelCase sein; `actors` MUSS PascalCase sein | Fehler |
| 13 | Entitaets-/Feature-/Akteursnamen, die in `stories` referenziert werden, SOLLTEN in ihren jeweiligen Feldern existieren | Warnung |
| 14 | `version` SOLLTE dem semver-Format folgen | Warnung |

---

## 10. Rollen

### 10.1 Mensch (Chef)

- Ueberfliegt das Lock (gerendert als Markdown)
- Genehmigt oder lehnt ab
- Aendert die `denied`-Liste
- Liest KEINEN Code, prueft KEINE Implementierung

### 10.2 KI-Worker

1. Schreibt Code
2. Generiert `product.lock.json` aus dem Code
3. Bereitet Nachweise vor (intern, wird dem Menschen nicht gezeigt)
4. Reicht Lock + Nachweise beim Reviewer ein

### 10.3 KI-Reviewer

1. Empfaengt Lock vom Worker
2. Inspiziert die Codebasis unabhaengig (vertraut den Nachweisen des Workers NICHT)
3. Prueft:
   - Jede gesperrte Entitaet existiert (mit korrekten Feldern im strikten Modus)
   - Jedes gesperrte Feature existiert als identifizierbares Verhalten
   - Jeder Story-Ablauf existiert im Code
   - Jede Berechtigung wird korrekt durchgesetzt
   - Jedes abgelehnte Element existiert NICHT
   - Keine nicht deklarierten Entitaeten oder Features existieren, die im Lock stehen sollten
4. Berichtet: **bestanden** oder **Verstoessliste**

---

## 11. Lebenszyklus

```
1. Mensch beschreibt Absicht (natuerliche Sprache)
        ↓
2. KI-Worker baut das Produkt
        ↓
3. KI-Worker generiert product.lock.json aus dem Code
        ↓
4. Mensch ueberfliegt Lock (Markdown-Ansicht) → genehmigen / aendern / ablehnen
        ↓
5. Lock wird eingefroren
        ↓
6. KI-Worker entwickelt innerhalb der Lock-Grenze weiter
        ↓
7. KI-Reviewer prueft Code periodisch gegen das Lock
        ↓
8. Verstoesse werden gemeldet → Mensch entscheidet ueber Massnahmen
        ↓
9. Naechste Iteration → Versionsanhebung → Wiederholung ab Schritt 4
```

---

## 12. FAQ

### Warum JSON und nicht YAML?

JSON ist deterministisch — es gibt genau eine Art, dieselben Daten darzustellen. YAML hat mehrere aequivalente Darstellungen (Einrueckungsstile, Flow- vs. Block-Format, Anfuehrungsregeln). Fuer eine Spezifikation, die KI generiert und KI validiert, ist Determinismus wichtiger als menschliche Schreibfreundlichkeit. Der Mensch liest ohnehin die Markdown-Ansicht.

### Wann sollte man den lockeren Modus vs. den strikten Modus verwenden?

Den **lockeren Modus** verwenden, wenn die allgemeine Struktur wichtig ist, aber Details nicht. Den **strikten Modus** verwenden, wenn bestimmte Felder fuer das Produkt kritisch sind (z.B. Finanzdatenmodelle, compliance-sensible Entitaeten). Locker beginnen, bei Bedarf verschaerfen.

### Was gehoert in `denied` vs. was ist nur "noch nicht gebaut"?

Frage: "Wenn eine KI dies versehentlich hinzufuegen wuerde, waere es ein Produktverstoss?" Wenn ja, gehoert es in `denied`. Wenn nein, ist es nur das Backlog.

### Kann dasselbe Lock verschiedene Codebasen erzeugen?

Ja. Das Lock definiert WAS, nicht WIE. Dasselbe Lock mit verschiedenen Tech-Stacks erzeugt strukturell unterschiedlichen Code, der dieselbe Produktgrenze implementiert. Das Lock mit einer `product.plan.md` kombinieren, um auch das WIE festzulegen.

### Was ist mit Microservices? Ein Lock pro Service?

Ein Lock pro **Produkt**, nicht pro Service. Wenn Microservices zusammen ein Produkt bilden, teilen sie ein Lock. Wenn sie unabhaengige Produkte sind, bekommt jedes sein eigenes Lock.

### Kann ich das Lock mit benutzerdefinierten Feldern erweitern?

Nicht in v0.1. Zukuenftige Versionen koennten Erweiterungsfelder mit einem `x-`-Praefix-Muster unterstuetzen. Vorerst nur die definierten Felder verwenden.

---

## 13. Referenzen

- [RFC 8259 — The JavaScript Object Notation (JSON) Data Interchange Format](https://datatracker.ietf.org/doc/html/rfc8259)
- [RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [package.json — npm documentation](https://docs.npmjs.com/cli/v10/configuring-npm/package-json)

---

## 14. Zukuenftige Erweiterungen

Diese Features sind nicht Teil von v0.1, koennten aber in zukuenftigen Versionen hinzugefuegt werden:

- `extends` — von einem anderen Lock erben und bestimmte Felder ueberschreiben
- `milestones` — Phasen-Tagging (MVP / v2 / Zukunft)
- `contributors` — mehrere Autoren
- `repository` — Quellcode-Speicherort
- `changelog` — Lock-Versions-Diff-Verlauf
- Erweiterungsfelder mit `x-`-Praefix

---

*Die Product Lock Spezifikation steht unter der [MIT-Lizenz](https://opensource.org/licenses/MIT).*
*Spezifikationsquelle: [productlock.org](https://spec.productlock.org)*
