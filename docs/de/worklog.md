---
layout: default
title: Worklog
nav_order: 9
permalink: /de/worklog
lang: de
lang_label: Deutsch
page_id: worklog
---

# Product Lock Worklog-Spezifikation

### Version 0.1.0 — Februar 2026

> Ein strukturiertes Entwicklungsprotokoll, das Aenderungen relativ zu product.lock.json verfolgt.
>
> Lock definiert WAS. Plan definiert WIE. Score misst Komplexitaet. **Worklog zeichnet WARUM und WANN auf.**
>
> Zusammen rekonstruieren diese vier Dateien den vollstaendigen Projektkontext — ueber Sitzungen, ueber Agenten, ueber die Zeit hinweg.

Die vollstaendige Spezifikation findest du in der [Product Lock Spezifikation](https://spec.productlock.org).

---

## Inhaltsverzeichnis

1. [Zweck](#1-zweck)
2. [Der geschlossene Kreislauf](#2-der-geschlossene-kreislauf)
3. [Dateiformat](#3-dateiformat)
4. [Erforderliche Abschnitte](#4-erforderliche-abschnitte)
5. [Optionale Abschnitte](#5-optionale-abschnitte)
6. [Eintragslebenszyklus](#6-eintragslebenszyklus)
7. [Kontextrekonstruktion](#7-kontextrekonstruktion)
8. [Konventionen](#8-konventionen)

---

## 1. Zweck

KI-Sitzungen sind fluechtig. Kontextfenster komprimieren, Konversationen laufen ab, Agenten verlieren ihr Gedaechtnis. Aber Produktentscheidungen bestehen fort. Das Worklog erfasst, was zaehlt:

- **Was sich geaendert hat** — geaenderte Dateien, hinzugefuegte Entitaeten, implementierte Features
- **Warum es sich geaendert hat** — die Entscheidung, der Ausloser, die Einschraenkung
- **Was es bedeutet** — Lock-Aenderungen, Komplexitaetsdelta, Auswirkung auf den Umfang
- **Was erkannt wurde** — Reviewer-Ergebnisse, behobene Verstoesse, durchgesetzte Grenzen

Ohne Worklog startet jede KI-Sitzung bei Null. Mit Worklog kann jeder Agent Lock + Worklog lesen und die Projekttrajektorie rekonstruieren.

### Was das Worklog NICHT ist

- Kein Git-Log (Git verfolgt Dateiaenderungen, das Worklog verfolgt Produktentscheidungen)
- Kein Changelog (Changelog ist fuer Benutzer, Worklog ist fuer Entwickler und KI)
- Kein Tagebuch (keine Meinungen, kein Kommentar — nur strukturierte Daten)

---

## 2. Der geschlossene Kreislauf

Die vier Product-Lock-Dateien bilden einen geschlossenen Feedbackkreislauf:

```
product.lock.json ──→ product.plan.md ──→ Implementierung ──→ product.worklog.md
       ↑                                                            │
       └────────────────────────────────────────────────────────────┘
                         (Lock wird basierend auf Worklog-Erkenntnissen aktualisiert)
```

| Datei | Rolle | Beantwortet |
|------|------|---------|
| `product.lock.json` | Grenze | Was ist dieses Produkt? |
| `product.plan.md` | Blaupause | Wie bauen wir es? |
| `product-lock-scoring.md` | Messung | Wie komplex ist es? |
| `product.worklog.md` | Historie | Was ist passiert, wann und warum? |

Jede Entwicklungssitzung:

1. **Vorher**: Lock + letzten Worklog-Eintrag lesen → aktuellen Stand verstehen
2. **Waehrend**: Aenderungen, Entscheidungen, Lock-Modifikationen verfolgen
3. **Nachher**: Worklog-Eintrag schreiben → Reviewer ausfuehren → Ergebnisse festhalten
4. **Abschluss**: Lock bei Bedarf aktualisieren → Score neu berechnen → Delta notieren

Der Kreislauf ist geschlossen, wenn jede Aenderung nachvollziehbar ist: vom Lock, das sie autorisiert hat, ueber die Sitzung, die sie implementiert hat, bis zum Reviewer, der sie verifiziert hat.

---

## 3. Dateiformat

Worklogs werden in einem `worklogs/`-Verzeichnis im Projektstammverzeichnis gespeichert, neben `product.lock.json`. Jede Sitzung ist eine Datei.

```
project-root/
├── product.lock.json
├── product.plan.md
└── worklogs/
    ├── 2026-02-16-initial-release.md
    ├── 2026-02-17-add-payment-system.md
    ├── 2026-02-17T1430-fix-payment-webhook.md
    └── ...
```

### Dateibenennung

```
{date}-{kebab-case-title}.md
```

| Teil | Format | Beispiel |
|------|--------|---------|
| Datum | `YYYY-MM-DD` | `2026-02-17` |
| Datum (selber Tag) | `YYYY-MM-DDTHHmm` | `2026-02-17T1430` |
| Titel | kebab-case, unter 60 Zeichen | `add-payment-system` |

Beispiele:
```
2026-02-16-initial-release.md
2026-02-17-add-payment-system.md
2026-02-17T1430-fix-payment-webhook.md
2026-02-18-refactor-auth-middleware.md
```

### Dateilistenreihenfolge

Dateien sortieren sich natuerlich nach Namen (Datumspraefix stellt chronologische Reihenfolge sicher). Um den neuesten Eintrag zu finden, absteigend sortieren oder die letzte Datei anschauen.

### Jede Datei ist eine Sitzung

Eine Datei = eine Sitzung = ein fokussierter Arbeitsblock. Jede Datei folgt der untenstehenden Vorlage:

```markdown
# [{date}] {title}

### Context
- Branch: `feature/xxx`
- Agent: {wer die Arbeit gemacht hat}
- Lock version: {Version vor dieser Sitzung}
- PLS before: {Score}

### Goal
{Ein Satz: was diese Sitzung erreichen sollte.}

### Changes

| Action | Target | Detail |
|--------|--------|--------|
| added | Entity: Payment | 4 fields: id, amount, userId, createdAt |
| modified | Feature: checkout | Added Stripe integration |
| removed | Feature: manualInvoice | Replaced by automated billing |

### Lock Delta
{Was hat sich in product.lock.json geaendert, falls etwas.}

### Score Delta
- PLS: {vorher} → {nachher} ({+/-delta})
- Breakdown: D {vorher}→{nachher}, F {vorher}→{nachher}, I {vorher}→{nachher}, A {vorher}→{nachher}

### Review Findings
{KI-Reviewer-Ergebnisse nach dieser Sitzung.}

### Decisions
{Wichtige Entscheidungen und deren Begruendung.}

### Files
| Action | Path |
|--------|------|
| created | src/models/payment.ts |
| modified | src/routes/checkout.ts |
| deleted | src/routes/manual-invoice.ts |

### Next
{Was die naechste Sitzung aufgreifen sollte.}
```

---

## 4. Erforderliche Abschnitte

Jeder Worklog-Eintrag MUSS die folgenden Abschnitte enthalten.

### 4.1 Context

Sitzungsmetadaten. Legt den Ausgangszustand fest.

```markdown
### Context
- Branch: `feature/payment-system`
- Agent: Claude (Opus 4)
- Lock version: 1.2.0
- PLS before: 145
```

| Feld | Erforderlich | Beschreibung |
|-------|----------|-------------|
| Branch | Ja | Git-Branch fuer diese Sitzung |
| Agent | Ja | Wer die Arbeit gemacht hat (KI-Modell, menschlicher Name oder beides) |
| Lock version | Ja | Version von product.lock.json zu Sitzungsbeginn |
| PLS before | Ja | Product Lock Score zu Sitzungsbeginn |

### 4.2 Goal

Ein Satz, der das Sitzungsziel beschreibt. Dies ist der Vertrag — alles im Eintrag sollte sich auf dieses Ziel beziehen.

```markdown
### Goal
Add payment processing with Stripe so Members can pay for premium features.
```

Regeln:
- MUSS ein Satz sein
- MUSS spezifisch genug sein, damit die Fertigstellung verifizierbar ist
- SOLLTE Lock-Konzepte (entities, features, actors) referenzieren, wenn zutreffend

### 4.3 Changes

Eine Tabelle der produktebenen Aenderungen waehrend der Sitzung. Dies ist KEIN Datei-Diff — es ist ein Produktgrenzen-Diff.

```markdown
### Changes

| Action | Target | Detail |
|--------|--------|--------|
| added | Entity: Payment | id, amount, currency, userId, status, createdAt |
| added | Entity: Subscription | id, planId, userId, startDate, endDate |
| added | Feature: createPayment | Stripe checkout session creation |
| added | Feature: cancelSubscription | Self-service cancellation |
| modified | Feature: signup | Now creates free-tier Subscription |
| added | Actor: Subscriber | Paying member with premium access |
| added | Permission: Subscriber | createPayment, cancelSubscription, all Member perms |
| added | Denied: refundPayment | Refunds handled manually by admin |
```

Aktionen:
- `added` — neues Element in der Produktgrenze
- `modified` — bestehendes Element geaendert
- `removed` — Element aus der Produktgrenze entfernt
- `unchanged` — explizite Notiz, dass etwas NICHT geaendert wurde (wenn relevant)

Zielformat: `{Typ}: {Name}`, wobei Typ Entity, Feature, Actor, Permission, Denied, Story ist.

### 4.4 Review Findings

Ergebnisse des KI-Reviewers nach den Aenderungen der Sitzung. Wenn kein Review durchgefuehrt wurde, dies explizit angeben.

```markdown
### Review Findings

Reviewer: Claude (Sonnet 4.5)
Status: PASS (2 warnings)

- [PASS] Entity Payment — model exists with correct fields
- [PASS] Feature createPayment — POST /api/payments endpoint
- [WARN] Undeclared entity: PaymentLog (exists in code, not in lock)
- [WARN] Undeclared feature: retryPayment (exists in code, not in lock)
```

Oder:

```markdown
### Review Findings

No review conducted this session.
```

Regeln:
- MUSS den Reviewer benennen (KI-Modell oder Mensch)
- MUSS den Gesamtstatus enthalten: PASS, FAIL oder NOT RUN
- MUSS alle Verstoesse und Warnungen auflisten
- Bei FAIL: der Eintrag MUSS einen Nachfolgepunkt im Next-Abschnitt enthalten

### 4.5 Files

Dateien, die waehrend der Sitzung erstellt, geaendert oder geloescht wurden.

```markdown
### Files

| Action | Path |
|--------|------|
| created | prisma/migrations/003_add_payment.sql |
| created | src/models/payment.ts |
| created | src/models/subscription.ts |
| created | src/routes/payments.ts |
| modified | src/routes/auth.ts |
| modified | src/middleware/permissions.ts |
| modified | prisma/schema.prisma |
| deleted | src/routes/legacy-billing.ts |
```

Regeln:
- Jede angefasste Datei auflisten, nicht nur die wichtigen
- Relative Pfade vom Projektstamm verwenden
- Aktionen: `created`, `modified`, `deleted`
- Sortiert nach Aktion (created → modified → deleted), dann alphabetisch innerhalb jeder Gruppe

---

## 5. Optionale Abschnitte

Diese Abschnitte einbeziehen, wenn sie nuetzlichen Kontext liefern.

### 5.1 Lock Delta

Was sich waehrend dieser Sitzung in product.lock.json geaendert hat. Ueberspringen, wenn das Lock nicht geaendert wurde.

```markdown
### Lock Delta

Lock updated: 1.2.0 → 1.3.0

Added:
- Entity: Payment [id, amount, currency, userId, status, createdAt]
- Entity: Subscription [id, planId, userId, startDate, endDate]
- Feature: createPayment
- Feature: cancelSubscription
- Actor: Subscriber
- Denied: refundPayment — "Refunds handled manually by admin"

Modified:
- Feature: signup (now includes subscription creation)
- Permission: Member → added cancelSubscription

Removed:
- (none)
```

Regeln:
- MUSS die Versionsaenderung angeben
- Nach Added / Modified / Removed gruppieren
- Lock-Feldformat verwenden (Entitaetsnamen PascalCase, Features camelCase usw.)

### 5.2 Score Delta

Wie sich der Product Lock Score geaendert hat. Ueberspringen, wenn das Lock nicht geaendert wurde.

```markdown
### Score Delta

- PLS: 145 → 172 (+27)
- Level: Moderat → Komplex
- Breakdown:
  - D: 79.2 → 94.8 (+15.6) — 2 neue Entitaeten
  - F: 34 → 38 (+4) — 2 neue Features + 2 bestehende
  - I: 22.5 → 27.5 (+5) — Payment-Stories hinzugefuegt
  - A: 9 → 11.7 (+2.7) — Subscriber-Rolle hinzugefuegt
```

Wenn der Score eine Stufengrenze ueberschreitet (z.B. Moderat → Komplex), SOLLTE dies explizit hervorgehoben werden. Stufenuebergaenge sind Signale fuer eine Umfangspruefung.

### 5.3 Decisions

Wichtige Entscheidungen waehrend der Sitzung und deren Begruendung. Ueberspringen, wenn keine bedeutenden Entscheidungen getroffen wurden.

```markdown
### Decisions

1. **Stripe statt Paddle**: Stripe hat eine bessere API fuer Abonnementverwaltung. Paddle buendelt Steuerbehandlung, aber wir brauchen das fuer das MVP nicht.

2. **Keine Rueckerstattungen in v1**: Zur denied-Liste hinzugefuegt. Manuelle Rueckerstattung ueber das Stripe-Dashboard ist ausreichend. Automatisierte Rueckerstattungen fuegen Haftung und Randfaelle hinzu (Teilerstattungen, anteilige Berechnung).

3. **Subscription als separate Entitaet**: Haette ein Feld auf User sein koennen, aber Subscription hat einen eigenen Lebenszyklus (Start, Ende, Kuendigung, Verlaengerung), der eine separate Entitaet rechtfertigt.
```

Regeln:
- Jede Entscheidung nummerieren
- Den Entscheidungstitel fett setzen
- Begruendung (das WARUM) einschliessen
- Wenn eine Entscheidung die denied-Liste aendert, dies vermerken

### 5.4 Next

Was die naechste Sitzung aufgreifen sollte. Nur ueberspringen, wenn das Projekt abgeschlossen ist.

```markdown
### Next

- [ ] Webhook-Handler fuer Stripe-Events implementieren (payment_intent.succeeded, subscription.canceled)
- [ ] PaymentLog zum Lock hinzufuegen oder aus der Codebasis entfernen (Reviewer-Warnung)
- [ ] Stories fuer den Zahlungsablauf schreiben und zum Lock hinzufuegen
- [ ] Vollstaendigen Review nach Webhook-Implementierung durchfuehren
```

Regeln:
- Checkbox-Format fuer umsetzbare Punkte verwenden
- Reviewer-Ergebnisse referenzieren, wenn zutreffend
- Spezifisch genug sein, damit ein anderer Agent dies aufgreifen kann

### 5.5 Commits

Git-Commits waehrend der Sitzung. Ueberspringen, wenn keine Commits gemacht wurden (z.B. reine Planungssitzung).

```markdown
### Commits

| Hash | Message |
|------|---------|
| a1b2c3d | feat: add Payment and Subscription models |
| e4f5g6h | feat: implement Stripe checkout endpoint |
| i7j8k9l | fix: permission middleware for Subscriber role |
| m0n1o2p | chore: update product.lock.json to v1.3.0 |
```

---

## 6. Eintragslebenszyklus

Ein Worklog-Eintrag folgt diesem Lebenszyklus innerhalb einer Entwicklungssitzung:

```
Sitzungsbeginn
│
├─ 1. Letzten Worklog-Eintrag lesen → aktuellen Stand verstehen
├─ 2. Lock lesen → Produktgrenze verstehen
├─ 3. Plan lesen → Implementierungsentscheidungen verstehen
│
├─ 4. Die Arbeit erledigen
│   ├─ Aenderungen verfolgen (Entitaeten, Features, Akteure)
│   ├─ Geaenderte Dateien verfolgen
│   └─ Entscheidungen festhalten, sobald sie fallen
│
├─ 5. Lock aktualisieren, wenn sich die Produktgrenze geaendert hat
├─ 6. Score neu berechnen, wenn das Lock geaendert wurde
├─ 7. Reviewer gegen aktualisiertes Lock ausfuehren
│
├─ 8. Worklog-Eintrag schreiben
│   ├─ Context (Branch, Agent, Lock-Version, PLS)
│   ├─ Goal (was war das Ziel)
│   ├─ Changes (Aenderungen auf Produktebene)
│   ├─ Lock Delta (wenn Lock geaendert wurde)
│   ├─ Score Delta (wenn Score sich geaendert hat)
│   ├─ Review Findings (Reviewer-Ergebnisse)
│   ├─ Decisions (Schluesselentscheidungen + Begruendung)
│   ├─ Files (alle angefassten Dateien)
│   ├─ Commits (Git-Commits)
│   └─ Next (was als Naechstes aufzugreifen ist)
│
Sitzungsende
```

### Wann einen Eintrag erstellen

- Ein Eintrag pro Entwicklungssitzung (eine Sitzung = ein fokussierter Arbeitsblock)
- Eine Sitzung, die nur plant, aber keinen Code schreibt, bekommt trotzdem einen Eintrag (ohne Files-Abschnitt)
- Eine Sitzung, die nur prueft, aber nichts aendert, bekommt trotzdem einen Eintrag (mit Review Findings)
- Mehrere kleine Korrekturen in einer Sitzung = ein Eintrag
- Grosses Feature ueber mehrere Sitzungen = mehrere Eintraege

### Eintraggroesse

Eintraege praegnant halten. Ein typischer Eintrag umfasst 30–80 Zeilen. Wenn ein Eintrag 150 Zeilen ueberschreitet, war die Sitzung zu gross — naechstes Mal in mehrere Eintraege aufteilen.

---

## 7. Kontextrekonstruktion

Der primaere Wert des Worklogs: Jeder Agent kann den Projektkontext rekonstruieren, indem er strukturierte Dateien liest, anstatt Konversationsverlaeufe abzuspielen.

### Ohne Worklog

```
Neue KI-Sitzung → liest Codebasis (Tausende Dateien) → raet die Absicht → oft falsch
```

### Mit Worklog

```
Neue KI-Sitzung → liest Lock (Produktgrenze)
              → liest letztes Worklog (aktueller Stand, aktuelle Entscheidungen, naechste Schritte)
              → liest Plan (Implementierungsansatz)
              → beginnt mit vollstaendigem Kontext zu arbeiten
```

### Was jede Datei zur Rekonstruktion beitraegt

| Frage | Beantwortet durch |
|----------|-------------|
| Was ist dieses Produkt? | `product.lock.json` |
| Was sind die genauen Grenzen? | `product.lock.json` (denied) |
| Wie komplex ist es? | PLS-Score (im Worklog) |
| Wie ist es gebaut? | `product.plan.md` |
| Was ist kuerzlich passiert? | `product.worklog.md` (neueste Eintraege) |
| Warum wurde X entschieden? | `product.worklog.md` (Decisions) |
| Was sollte ich als Naechstes tun? | `product.worklog.md` (Next) |
| Was hat der Reviewer gefunden? | `product.worklog.md` (Review Findings) |
| Ist der Umfang am Ausufern? | `product.worklog.md` (Score Delta ueber Zeit) |

### Token-Effizienz

Eine 2-stuendige KI-Konversation kann 100K+ Token an Kontext verbrauchen. Der Worklog-Eintrag fuer dieselbe Sitzung umfasst ~50 Zeilen strukturiertes Markdown. Die naechste Sitzung liest 50 Zeilen statt 100K Token abzuspielen.

```
Konversationsverlauf: 100.000+ Token (komprimiert, verlustbehaftet, fluechtig)
Worklog-Eintrag:           ~800 Token (strukturiert, vollstaendig, permanent)
```

### Scope-Drift-Erkennung

Durch die Verfolgung des PLS ueber Eintraege hinweg wird Scope Creep sichtbar:

```
[2026-02-10] v1.0.0 — PLS: 85  (Moderat)
[2026-02-12] v1.1.0 — PLS: 92  (+7, kleinere Ergaenzungen)
[2026-02-14] v1.2.0 — PLS: 145 (+53, deutlicher Sprung ⚠️)
[2026-02-17] v1.3.0 — PLS: 172 (+27, in Komplex ueberschritten)
```

Ein Sprung von +50 oder mehr in einer einzelnen Sitzung oder ein Stufengrenzenueberschreitung SOLLTE eine Umfangspruefung ausloesen.

---

## 8. Konventionen

### 8.1 Verzeichnisstruktur

- Verzeichnis: `worklogs/` im Projektstamm
- Ein Verzeichnis pro Produkt (entspricht einem Lock pro Produkt)
- Jede Datei ist eine Sitzung

```
project-root/
├── product.lock.json
└── worklogs/
    ├── 2026-02-16-initial-release.md
    └── 2026-02-17-add-payment-system.md
```

### 8.2 Dateibenennung

```
{YYYY-MM-DD}-{kebab-case-title}.md
```

- Datumspraefix stellt chronologische Sortierung sicher
- Titel ist Imperativ, kebab-case, unter 60 Zeichen
- Wenn mehrere Sitzungen am selben Tag, `{YYYY-MM-DD}T{HHmm}-{title}.md` verwenden

### 8.3 Titel

Imperativ und praegnant:

| Gut | Schlecht |
|------|-----|
| `add-payment-system` | `added-the-payment-system-and-also-fixed-some-bugs` |
| `fix-auth-middleware` | `authentication-middleware-was-broken` |
| `remove-legacy-billing` | `cleaned-up-old-billing-code` |

### 8.4 Querverweise

Beim Referenzieren anderer Eintraege den Dateinamen verwenden:

```markdown
Continues from `2026-02-14-setup-stripe-integration.md`.
See `2026-02-12-define-payment-entities.md` for schema decisions.
```

### 8.5 Namenskonventionen

Lock-Konventionen innerhalb des Worklogs befolgen:

| Element | Format | Beispiel |
|------|--------|---------|
| Entitaetsnamen | PascalCase | `Entity: Payment` |
| Feature-Namen | camelCase | `Feature: createPayment` |
| Akteursnamen | PascalCase | `Actor: Subscriber` |
| Dateipfade | relativ vom Stamm | `src/models/payment.ts` |

### 8.6 Lesereihenfolge

Um den Kontext zu rekonstruieren, SOLLTE ein KI-Agent:

1. Die **neueste** Worklog-Datei lesen (letzte nach Dateinamensortierung)
2. Den **Next**-Abschnitt auf ausstehende Arbeit pruefen
3. Weitere aktuelle Dateien lesen, wenn mehr Kontext benoetigt wird
4. Das Lock fuer die Produktgrenze lesen

---

*Diese Spezifikation ist Teil der [Product Lock Spezifikation](https://spec.productlock.org).*
