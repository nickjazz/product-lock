<p align="center">
  <a href="https://productlock.org">
    <img src="logo.png" alt="Product Lock" width="60">
  </a>
</p>

<h1 align="center">product.lock.json</h1>

<p align="center">
  Manage KI so, wie du Teams managst.
</p>

<p align="center">
  <a href="https://spec.productlock.org"><img src="https://img.shields.io/badge/Website-spec.productlock.org-blue?style=flat-square" alt="Website"></a>
  <a href="https://github.com/anthropics/product-lock/blob/main/LICENSE"><img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="License"></a>
  <a href="product-lock-spec.md"><img src="https://img.shields.io/badge/Spec-v0.1.0-orange?style=flat-square" alt="Spec Version"></a>
</p>

<p align="center">
  <a href="README.md">English</a> ·
  <a href="README-zh-TW.md">繁體中文</a> ·
  <a href="README-ja.md">日本語</a> ·
  <strong>Deutsch</strong>
</p>

---

**Deine Management-Erfahrung — Scope-Kontrolle, Review-Prozesse, Entscheidungs-Tracking — als Protokoll, dem KI folgen kann.**

product.lock.json definiert die Grenze eines Softwareprodukts — was es **ist** und was es **nicht ist**. Egal welche Sprache, welches Framework oder welche Architektur. Die Grenze bleibt gleich.

```
AI Worker schreibt Code  →  generiert Lock  →  Mensch genehmigt  →  AI Reviewer verifiziert
```

---

## Warum?

KI kann mittlerweile den Großteil des Codes schreiben. Das Problem: **Sie weiß nicht, wann sie aufhören soll.**

Du fragst nach einer Chat-App. Sie fügt hilfsbereit Emoji-Reaktionen, Sprachanrufe und Dateifreigabe hinzu. Nichts davon hast du verlangt. Aber die KI weiß das nicht, weil du nie „Nein" gesagt hast.

Traditionelle Entwicklung hat eine natürliche Grenze — menschliche Zeit und Aufwand. KI hat das nicht. Also muss die Grenze explizit festgeschrieben werden.

**product.lock.json ist diese Grenze.**

---

## Wie sieht das aus?

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

6 Felder. 6 Fragen:

| Feld | Frage |
|------|-------|
| `actors` | Wer nutzt es? |
| `entities` | Welche Daten werden gespeichert? |
| `features` | Was kann es? |
| `stories` | Wie interagieren die Teile? |
| `permissions` | Wer darf was? |
| `denied` | **Was darf es NICHT haben?** |

Das letzte ist das wichtigste. Im KI-Zeitalter ist es wichtiger zu definieren, was man **nicht** baut, als was man baut.

---

## Kernideen

### WAS, nicht WIE

Keine Routen, keine Frameworks, keine Abhängigkeiten, keine Dateistruktur im Lock.

Gleicher Lock, verschiedene Stacks. Gib ihn der KI mit „Bau das in Python + FastAPI" oder „Bau das in TypeScript + Next.js". Die Produktgrenze bleibt identisch.

### Mehr spezifizieren, strenger durchsetzen

```json
"entities": ["User"]                              // prüft nur, ob User existiert
"entities": { "User": [] }                        // dasselbe — prüft nur Existenz
"entities": { "User": ["id", "name", "email"] }   // prüft exakte Felder, nicht mehr, nicht weniger
```

Ein Feld weglassen = KI entscheidet frei. Du sperrst nur, was dir wichtig ist.

### Drei Rollen

| Rolle | Aufgabe |
|-------|---------|
| **Mensch** | Prüft den Lock, genehmigt oder lehnt ab, verwaltet die Denied-Liste |
| **AI Worker** | Schreibt Code, generiert den Lock |
| **AI Reviewer** | Prüft Code unabhängig gegen den Lock. Vertraut dem Worker NICHT |

Der Mensch liest nie Code. Der Mensch liest den Lock (als Markdown gerendert). Dauert Minuten, nicht Stunden.

### `denied` ist die Leitplanke

```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "voiceCall": "Text-based communication only"
}
```

Wenn KI versehentlich etwas aus `denied` hinzufügt, markiert der Reviewer es als Verstoß. Das ist die wichtigste Verteidigungslinie.

Der Test: „Wenn KI das eigenmächtig hinzugefügt hat — wäre das ein Produktverstoß?" Ja → in denied aufnehmen. Nein → das ist nur Backlog, weglassen.

---

## Dokumentation

| Dokument | Zielgruppe | Zweck |
|----------|-----------|-------|
| [Spezifikation](product-lock-spec.md) | Alle | Vollständige Spezifikation (Englisch) |
| [Spezifikation (Chinesisch)](product-lock-spec-zh.md) | Alle | Vollständige Spezifikation (Chinesisch) |
| [Generator-Leitfaden](product-lock-generator-guide.md) | AI Worker | Wie man einen Lock aus einer Codebasis generiert |
| [Reviewer-Leitfaden](product-lock-reviewer-guide.md) | AI Reviewer | Wie man Code gegen einen Lock verifiziert |
| [Plan-Spezifikation](product-plan-spec.md) | AI Worker | product.plan.md Format (das WIE) |
| [Scoring](product-lock-scoring.md) | Alle | Produktkomplexität aus einem Lock quantifizieren |
| [Worklog](product-lock-worklog.md) | KI + Mensch | Änderungen, Entscheidungen und Umfang über Sitzungen hinweg verfolgen |

---

## Schnellstart

### Einfachster möglicher Lock

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

4 Metadaten-Felder + mindestens 1 Produktfeld. Das war's.

### KI einen generieren lassen

Gib den [Generator-Leitfaden](product-lock-generator-guide.md) einer KI und zeig auf deine Codebasis:

> „Lies diesen Leitfaden, analysiere dann diese Codebasis und generiere eine product.lock.json."

### KI dagegen prüfen lassen

Gib den [Reviewer-Leitfaden](product-lock-reviewer-guide.md) einer *anderen* KI, zusammen mit dem Lock und der Codebasis:

> „Lies diesen Leitfaden, dann überprüfe diese Codebasis gegen diese product.lock.json."

Zwei KIs. Sie vertrauen einander nicht. Eine generiert, eine verifiziert. Der Mensch trifft die endgültige Entscheidung.

---

## Namenskonventionen

| Feld | Format | Beispiel |
|------|--------|----------|
| `name` | kebab-case | `"chat-system"` |
| `actors` | PascalCase | `"Admin"`, `"Member"` |
| `entities` | PascalCase | `"User"`, `"ReadReceipt"` |
| Entity-Felder | camelCase | `"senderId"`, `"createdAt"` |
| `features` | camelCase | `"sendMessage"`, `"createGroup"` |
| `denied` | PascalCase oder camelCase | `"Reaction"`, `"editMessage"` |

Alle Arrays alphabetisch sortiert. Einzige Ausnahme: `stories` folgen dem narrativen Ablauf.

---

## FAQ

**Warum JSON und nicht YAML?**
JSON ist deterministisch — gleiche Daten, eine Schreibweise. Wenn KI generiert und KI validiert, schlägt Determinismus Bequemlichkeit. Menschen lesen sowieso das gerenderte Markdown.

**Ein Lock pro Microservice?**
Ein Lock pro Produkt. Microservices sind WIE, nicht WAS.

**Kann ich `denied` weglassen?**
Kannst du. Aber das sagt der KI: „Füg hinzu, was du willst." Bist du sicher?

**Können verschiedene Sprachen denselben Lock implementieren?**
Ja. Genau darum geht's.

---

## Lizenz

MIT

---

<p align="center">
  <a href="https://productlock.org"><strong>productlock.org</strong></a>
</p>
