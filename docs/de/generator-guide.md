---
layout: default
title: Generator-Leitfaden
nav_order: 4
permalink: /de/generator-guide
lang: de
lang_label: Deutsch
page_id: generator-guide
---

# Product Lock Generator-Leitfaden

### Version 0.1.0 — Februar 2026

> Du bist ein KI-Worker. Deine Aufgabe ist es, eine Codebasis zu analysieren und eine `product.lock.json` zu generieren, die die Produktgrenze praezise beschreibt.
>
> Du generierst das Lock. Ein Mensch genehmigt es. Ein KI-Reviewer verifiziert den Code dagegen.

Die vollstaendige Spezifikation findest du in der [Product Lock Spezifikation](https://spec.productlock.org).

---

## Inhaltsverzeichnis

1. [Was ist product.lock.json?](#1-was-ist-productlockjson)
2. [Generierungsprozess](#2-generierungsprozess)
3. [Feldreferenz](#3-feldreferenz)
4. [Namenskonventionen](#4-namenskonventionen)
5. [Validierungscheckliste](#5-validierungscheckliste)
6. [Haeufige Fehler](#6-haeufige-fehler)
7. [Vollstaendiges Beispiel](#7-vollstaendiges-beispiel)

---

## 1. Was ist product.lock.json?

Eine `product.lock.json` definiert, **was ein Softwareprodukt ist und nicht ist** — auf Produktebene, nicht auf Code-Ebene.

Sie beantwortet sechs Fragen:

| Feld | Frage | Format |
|-------|----------|--------|
| `actors` | Wer nutzt dieses Produkt? | PascalCase-Substantive |
| `entities` | Welche Daten speichert es? | PascalCase-Substantive |
| `features` | Was kann es tun? | camelCase-Verben |
| `stories` | Wie interagieren Akteure, Entitaeten und Features? | Saetze in natuerlicher Sprache |
| `permissions` | Wer darf was tun? | Akteur-zu-Feature-Matrix |
| `denied` | Was darf es NICHT haben? | Explizite Ausschluesse |

**Das Lock beschreibt die Produktgrenze, nicht die Implementierung.** Keine Routen, keine Abhaengigkeiten, keine Framework-Entscheidungen, keine Dateistruktur.

---

## 2. Generierungsprozess

### Schritt 1: Die Codebasis lesen

Die Codebasis scannen, um zu verstehen:

- Datenbankschemata / Modelle / Typen → entities
- API-Routen / Handler / Controller → features
- Auth / Rollendefinitionen → actors, permissions
- UI-Seiten / Ansichten → features, stories
- Middleware / Hintergrundjobs / Cron → stories
- Tests → helfen zu bestaetigen, welche Features existieren

### Schritt 2: Metadaten extrahieren

```json
{
  "name": "<kebab-case product name>",
  "version": "<semver, start with 0.1.0 if new>",
  "description": "<one-line product description>",
  "author": "<who will approve this lock>"
}
```

- `name`: abgeleitet vom Projektordnernamen oder package.json, immer kebab-case
- `description`: den Zweck des Produkts in einem Satz beschreiben, aus der Perspektive des Benutzers
- `author`: den Menschen fragen oder den Git-Committer-Namen verwenden

### Schritt 3: Akteure identifizieren

Akteure sind **die Personen, die das Produkt nutzen**, keine Code-Entitaeten.

Suche nach:
- Rollendefinitionen in Auth/RBAC-Code
- Verschiedene Berechtigungsstufen
- Unterschiedliche Benutzertypen in der UI

```json
"actors": ["Admin", "Member", "Guest"]
```

Regeln:
- PascalCase
- Alphabetisch sortiert
- KEINE Code-Entitaeten (kein `UserService`, `AuthMiddleware`)
- Wenn nur ein Benutzertyp existiert, `["User"]` verwenden

### Schritt 4: Entitaeten extrahieren

Entitaeten sind **was das Produkt speichert** — das Datenmodell.

Suche nach:
- Datenbanktabellen / Prisma-Modelle / TypeORM-Entitaeten
- Mongoose-Schemata / SQLAlchemy-Modelle
- TypeScript-Interfaces mit `id`-Feldern
- GraphQL-Typen

**Entscheidung: Lockerer oder Strikter Modus?**

Den **lockeren Modus** verwenden, wenn:
- Ein schneller Ueberblick ausreicht
- Felder standardmaessig und vorhersehbar sind
- Der Mensch keine bestimmten Felder sperren muss

```json
"entities": ["Conversation", "Message", "User"]
```

Den **strikten Modus** verwenden, wenn:
- Felder fuer das Produkt kritisch sind (z.B. Finanzdaten)
- Der Mensch das exakte Datenmodell verifizieren muss
- Das Produkt komplexe oder unuebliche Feldanforderungen hat

```json
"entities": {
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "User": ["avatar", "email", "id", "name"]
}
```

Regeln:
- PascalCase fuer Entitaetsnamen
- camelCase fuer Feldnamen
- Alphabetisch sortiert (sowohl Schluessel als auch Feld-Arrays)
- `[]` = Entitaet existiert, Felder nicht gesperrt
- Felder sind nur Namen — **keine Typen** (Typ ist Implementierungsdetail)

**Kritische Beurteilung: Entitaet oder KEINE Entitaet?**

| Als Entitaet aufnehmen | NICHT aufnehmen |
|-------------------|---------------|
| User, Product, Order — Domaenendaten | CachedQuote, TempSession — Implementierungsdetail |
| Message, Conversation — benutzersichtbar | MigrationLog, AuditTrail — operativ |
| StockMetadata — Produkt speichert dies | RedisKey, QueueJob — Infrastruktur |

Wenn etwas wie eine Entitaet aussieht, aber eigentlich Implementierung ist (Cache-Tabellen, temporaerer Speicher, Job-Queues), **NICHT als Entitaet aufnehmen**. Stattdessen die Produktanforderung, die es erfuellt, als **Story** erfassen (siehe Schritt 6).

### Schritt 5: Features extrahieren

Features sind **atomare Produktfaehigkeiten** — was das Produkt kann.

Suche nach:
- API-Endpunkte (jeder wichtige Endpunkt = ein Feature)
- UI-Aktionen (Buttons, Formulare, Workflows)
- Hintergrundjob-Faehigkeiten
- Systemintegrationen

```json
"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

Regeln:
- camelCase
- Verb + Substantiv-Format: `sendMessage`, `createGroup`, `viewDashboard`
- Alphabetisch sortiert
- Flache Liste — keine Sub-Features, keine Verschachtelung
- Jedes Feature MUSS in der Codebasis eigenstaendig identifizierbar sein

**Granularitaetsleitfaden:**

| Zu breit | Gut | Zu granular |
|-----------|------|-----------|
| `manageMessages` | `sendMessage`, `deleteMessage` | `validateMessageLength` |
| `handleAuth` | `login`, `register`, `resetPassword` | `hashPassword` |
| `stockAnalysis` | `analyzeStock`, `viewStockChart` | `calculateMovingAverage` |

Features sollten das abbilden, was ein **Produktmanager** auflisten wuerde, nicht was ein **Entwickler** auflisten wuerde.

### Schritt 6: Stories schreiben

Stories beschreiben, **wie Akteure, Entitaeten und Features interagieren**. Sie sind die Erzaehlung des Produkts.

Drei Typen:

**Funktionale Ablaeufe** — was passiert, wenn jemand das Produkt nutzt:
```
"Member sends Message to Conversation"
"Admin deletes any Message from Conversation"
"When Member reads Message, system creates ReadReceipt"
```

**Erfahrungserwartungen** — was der Benutzer wahrnimmt (nicht-funktionale Anforderungen):
```
"User views StockQuote with data refreshing in real-time"
"User views Dashboard with market data loading instantly"
```

**Systemverhalten** — was das System autonom tut:
```
"System fetches NewsArticle from multiple sources on schedule"
"System rate limits API requests per IP address"
"System calibrates model with Bayesian posteriors and detects concept drift"
```

Regeln:
- Ein Satz pro Story
- Beginnt mit Akteursname (PascalCase, MUSS in `actors` sein) oder `"System"`
- Referenziert Entitaeten mit exaktem PascalCase-Namen
- Praesens, Aktiv
- Beschreibt WAS passiert, nicht WIE es implementiert wird
- Geordnet nach **erzaehlerischem Ablauf** (NICHT alphabetisch — dies ist die einzige Ausnahme)

**Die Entitaet-zu-Story-Regel:**

Wenn Implementierungsdetails gefunden werden, die einem Produktzweck dienen, als Stories umwandeln:

| Implementierung gefunden | Zu schreibende Story |
|---------------------|---------------|
| `CachedQuote`-Tabelle | `"User views StockQuote with data refreshing in real-time"` |
| `CachedHistory`-Tabelle | `"User views StockChart with historical data loading instantly"` |
| Rate-Limiter-Middleware | `"System rate limits API requests per IP address"` |
| Embedding-Vektorspeicher | `"System searches knowledge base by semantic similarity"` |
| Hintergrund-Cron-Job | `"System fetches NewsArticle from multiple sources on schedule"` |

### Schritt 7: Berechtigungen definieren

Berechtigungen verbinden **Akteure mit Features** — wer was tun darf.

**Den richtigen Modus waehlen:**

Wenn das Produkt keine Auth oder nur einfache Auth hat:
- Das Feld komplett weglassen (KI entscheidet frei)

Wenn das Produkt ein bekanntes Auth-Modell verwendet:
```json
"permissions": "rbac"
```

Wenn die exakte Rollen-Feature-Zuordnung angegeben werden muss:
```json
"permissions": {
  "Admin": ["createGroup", "deleteMessage", "sendMessage"],
  "Guest": ["listConversations"],
  "Member": ["createGroup", "sendMessage"]
}
```

Regeln:
- Schluessel sind PascalCase, MUESSEN mit `actors` uebereinstimmen
- Werte sind camelCase-Arrays, SOLLTEN Teilmengen von `features` sein
- Sowohl Schluessel als auch Wert-Arrays alphabetisch sortiert
- System-exklusive Features (wie `fetchNews`, `enrichNewsWithAi`) gehoeren NICHT in Berechtigungen — sie sind nicht vom Benutzer aufrufbar

### Schritt 8: Abgelehnte Elemente bestimmen

**Dies ist der wichtigste Schritt.** Abgelehnte Elemente sind die Leitplanke, die verhindert, dass die KI den Produktumfang ueberschreitet.

In der traditionellen Entwicklung werden Produkte durch menschliche Zeit begrenzt. Mit KI gibt es keine natuerliche Grenze — KI wird Features, Entitaeten und Verhaltensweisen hinzufuegen, sofern nicht explizit davon abgehalten. `denied` ist die Methode, diese Grenze zu ziehen.

Dies ist NICHT "alles, was das Produkt noch nicht hat." Es ist fuer **absichtliche Produktentscheidungen**, bei denen die Abwesenheit wichtig ist.

**Entscheidungsrahmen:**

Frage: "Wenn eine KI dies versehentlich hinzufuegen wuerde, waere es ein Produktverstoss?"
- Ja → in denied aufnehmen
- Nein, es ist nur noch nicht gebaut → NICHT aufnehmen

**Lockerer Modus** — flache Liste:
```json
"denied": ["executeTrade", "Reaction", "voiceCall"]
```

**Mit Begruendungen** (empfohlen) — jedes Element mit Erklaerung:
```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "Reaction": "Intentionally excluded to keep messaging simple",
  "voiceCall": "Text-based communication only"
}
```

Begruendungen helfen dem Reviewer zu verstehen, WARUM etwas ausgeschlossen ist, und helfen dem Menschen, bessere Genehmigen/Ablehnen-Entscheidungen zu treffen.

**Drei Kategorien abgelehnter Elemente:**

| Kategorie | Beispiel | Begruendung |
|----------|---------|--------|
| Produktumfang | `"voiceCall"` | "Nur textbasierte Kommunikation" |
| Haftungsgrenze | `"executeTrade"` | "Keine Brokerhaftung" |
| Scope-Leitplanke | `"Reaction"` | "Messaging einfach halten" |

### Schritt 9: Zusammensetzen und validieren

Das vollstaendige Lock in dieser Schluesselreihenfolge zusammensetzen:

```
$schema → name → version → description → author → license → keywords → private
→ actors → entities → features → stories → permissions → denied
```

Die Validierungscheckliste durchgehen (siehe [Abschnitt 5](#5-validierungscheckliste)).

### Schritt 10: Markdown-Ansicht generieren

Nach dem JSON-Lock eine `product.lock.md` fuer die menschliche Pruefung generieren:

```markdown
# {name} v{version}

{description}

**Author:** {author}

---

## Actors

- Actor1
- Actor2

---

## Entities

Entity1, Entity2, Entity3

---

## Features

- feature1
- feature2

---

## Stories

- Story 1
- Story 2

---

## Permissions

### Actor1
feature1, feature2, feature3

### Actor2
feature1

---

## Denied

- item1 — reason1
- item2 — reason2
```

---

## 3. Feldreferenz

### Metadatenfelder

| Feld | Erforderlich | Format | Beschreibung |
|-------|----------|--------|-------------|
| `$schema` | Nein | string | Schema-URL fuer Validierung |
| `name` | **Ja** | kebab-case string | Produktbezeichner |
| `version` | **Ja** | semver string | Produktversion |
| `description` | **Ja** | string | Einzeilige Produktbeschreibung |
| `author` | **Ja** | string | Wer dieses Lock genehmigt |
| `license` | Nein | string | Lizenzbezeichner |
| `keywords` | Nein | lowercase string[] | Tags fuer Auffindbarkeit |
| `private` | Nein | boolean | Nicht zum oeffentlichen Teilen gedacht |

### Produktgrenzenfelder

Alle optional. Weggelassen = KI entscheidet frei.

| Feld | Typ | Beschreibung |
|-------|------|-------------|
| `actors` | `string[]` | Benutzerrollen |
| `entities` | `string[]` oder `Record<string, string[]>` | Datenmodell |
| `features` | `string[]` | Produktfaehigkeiten |
| `stories` | `string[]` | Interaktionsablaeufe und Verhaltensweisen |
| `permissions` | `string` oder `Record<string, string[]>` | Zugriffskontrolle |
| `denied` | `string[]` oder `Record<string, string>` | Explizite Ausschluesse |

---

## 4. Namenskonventionen

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

---

## 5. Validierungscheckliste

Vor dem Einreichen des Locks alle folgenden Punkte pruefen:

| # | Pruefpunkt | Bestanden? |
|---|-------|-------|
| 1 | `name`, `version`, `description`, `author` vorhanden | |
| 2 | Mindestens ein Produktgrenzenfeld vorhanden | |
| 3 | Alle Entitaetsnamen PascalCase | |
| 4 | Alle Feature-Namen camelCase (Verb + Substantiv) | |
| 5 | Alle Akteursnamen PascalCase | |
| 6 | Alle Arrays alphabetisch sortiert (ausser stories) | |
| 7 | Alle Objektschluessel alphabetisch sortiert | |
| 8 | Stories beginnen mit Akteursname oder "System" | |
| 9 | Stories referenzieren Entitaeten mit exaktem PascalCase-Namen | |
| 10 | `permissions`-Schluessel stimmen mit `actors` ueberein | |
| 11 | `denied`-Eintraege erscheinen nicht in `entities` oder `features` | |
| 12 | Keine Implementierungsentitaeten (Cache, Temp, Queue → stories) | |
| 13 | Keine Implementierungsdetails in Stories (kein Redis, WebSocket, Framework-Namen) | |
| 14 | Entitaetsfelder sind nur Namen, keine Typen | |

---

## 6. Haeufige Fehler

### 1. Implementierungsentitaeten aufnehmen
```
FALSCH:  "entities": ["CachedQuote", "Message", "User"]
RICHTIG: "entities": ["Message", "User"]
         "stories": ["User views StockQuote with data refreshing in real-time"]
```

### 2. Implementierungsdetails in Stories
```
FALSCH:  "System uses Redis to cache stock quotes"
RICHTIG: "User views StockQuote with data refreshing in real-time"
```

### 3. Spezifikationssprache statt Stories
```
FALSCH:  "Users should be able to send messages"
RICHTIG: "Member sends Message to Conversation"
```

### 4. Zu viel ablehnen
```
FALSCH:  "denied": ["exportData", "multiLanguage", "mobileApp", "paymentBilling"]
         (Das sind nur noch nicht gebaute Features, keine absichtlichen Ausschluesse)

RICHTIG: "denied": {
           "executeTrade": "Market intelligence only — no brokerage liability",
           "voiceCall": "Text-based communication only"
         }
         (Bewusste Produktentscheidungen mit Begruendungen)
```

### 5. Zu wenig ablehnen
```
FALSCH:  "denied": []   oder denied komplett weglassen
         (KI hat keine Leitplanke, kann alles hinzufuegen)

RICHTIG: Gruendlich darueber nachdenken, was das Produkt NICHT werden soll.
         denied ist das wichtigste Feld in der KI-Aera der Entwicklung.
```

### 6. Zu granulare Features
```
FALSCH:  "features": ["validateEmail", "hashPassword", "generateToken", "refreshToken"]
RICHTIG: "features": ["login", "register", "resetPassword"]
```

### 7. System-Stories vergessen
Wenn es Hintergrundjobs, Crons oder automatisierte Prozesse gibt, brauchen auch diese Stories:
```
"System fetches NewsArticle from multiple sources on schedule"
"System detects SupplyChainDisruption from NewsArticle"
```

### 8. Unsortierte Arrays
```
FALSCH:  "actors": ["Member", "Admin", "Guest"]
RICHTIG: "actors": ["Admin", "Guest", "Member"]
```

---

## 7. Vollstaendiges Beispiel

Gegeben eine Chat-Anwendungs-Codebasis mit:
- Prisma-Schema: User, Conversation, Message, ReadReceipt-Modelle
- API-Routen: /messages, /conversations, /groups
- Auth: drei Rollen (admin, member, guest) via RBAC
- Keine Sprach-/Video-Features
- Kein Bearbeiten von Nachrichten

Generiertes Lock:

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

---

*Dieser Leitfaden ist Teil der [Product Lock Spezifikation](https://spec.productlock.org).*
