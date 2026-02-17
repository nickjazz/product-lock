---
layout: default
title: Plan-Eintrags-Spezifikation
nav_order: 7
permalink: /de/plan-entry-spec
lang: de
lang_label: Deutsch
page_id: plan-entry-spec
---

# Product Plan Eintrags-Spezifikation

### Version 0.1.0 — Februar 2026

> Plan-Eintraege zeichnen Implementierungsplaene vor und waehrend der Entwicklung auf.
>
> Alle Plaene befinden sich im `plans/`-Verzeichnis und ersetzen die einzelne `product.plan.md`-Datei.
>
> Plan = WIE & WARUM (was KI vorhat zu tun). Worklog = WAS & WANN (was KI tatsaechlich getan hat).

Die vollstaendige Lock-Spezifikation findest du in der [Product Lock Spezifikation](https://spec.productlock.org).

---

## Inhaltsverzeichnis

1. [Beziehung zu anderen Dateien](#1-beziehung-zu-anderen-dateien)
2. [Designprinzipien](#2-designprinzipien)
3. [Verzeichnis und Benennung](#3-verzeichnis-und-benennung)
4. [Format](#4-format)
5. [Erforderliche Abschnitte](#5-erforderliche-abschnitte)
6. [Optionale Abschnitte — Allgemein](#6-optionale-abschnitte--allgemein)
7. [Optionale Abschnitte — Vollstaendige Blaupause](#7-optionale-abschnitte--vollstaendige-blaupause)
8. [Plan vs Worklog](#8-plan-vs-worklog)

---

## 1. Beziehung zu anderen Dateien

```
plans/*.md        →  Plaene (WIE & WARUM — was KI vorhat zu tun)
worklogs/*.md     →  Aufzeichnungen (WAS & WANN — was KI tatsaechlich getan hat)
product.lock.json →  Grenze (WAS — was das Produkt ist)
```

Plan und Worklog sind unabhaengig, aber komplementaer:
- **Plan = vorher** — zeichnet Absicht, Ansatz, Schritte, Risiken auf
- **Worklog = nachher** — zeichnet tatsaechliche Aenderungen, Entscheidungen, Entdeckungen auf
- Lose korreliert durch Datum/Titel, keine erzwungene 1:1-Zuordnung

Das `plans/`-Verzeichnis ersetzt die einzelne `product.plan.md`-Datei. Alle Plaene — von vollstaendigen Produktblaupausen bis hin zu leichtgewichtigen Sitzungsplaenen — befinden sich im selben Verzeichnis, unterschieden durch Inhalt und Benennung.

---

## 2. Designprinzipien

1. **Umsetzbar** — Ein Plan ist ein konkreter Leitfaden, dem KI zur Implementierung folgen kann
2. **Begrenzt** — Jeder Plan hat ein klares Ziel und eine klare Grenze
3. **Klartext** — Markdown-Format, menschenlesbar
4. **Flexible Granularitaet** — Von vollstaendigen Produktblaupausen bis zu Einzelsitzungs-Aufgabenplaenen
5. **Nicht-blockierend** — Plaene benoetigen keine Genehmigung um zu existieren; sie dokumentieren Absichten

---

## 3. Verzeichnis und Benennung

Plaene MUESSEN im `plans/`-Verzeichnis im Projektstammverzeichnis platziert werden.

**Namenskonvention:** `{YYYY-MM-DD}-{kebab-title}.md`

Beispiele:
- `plans/2026-02-17-add-payment-entity.md`
- `plans/2026-02-18-full-product-blueprint.md`
- `plans/2026-02-19-refactor-auth-flow.md`

```
project-root/
├── product.lock.json
├── plans/
│   ├── 2026-02-17-add-payment-entity.md
│   ├── 2026-02-18-full-product-blueprint.md
│   └── 2026-02-19-refactor-auth-flow.md
└── worklogs/
    └── ...
```

---

## 4. Format

```markdown
# Plan: {title}

### Context
- Date: {YYYY-MM-DD}
- Branch: `{branch}`
- Agent: {wer diesen Plan erstellt hat}
- Lock version: {Version}
- PLS: {Score}
- Scope: {Scope-Beschreibung}

### Goal
{Was dieser Plan erreichen soll — 1-3 Saetze.}

### Approach
{Uebergeordnete Strategie — wie das Ziel erreicht wird.}

### Steps
1. {Erster Schritt}
2. {Zweiter Schritt}
3. ...
```

---

## 5. Erforderliche Abschnitte

Jeder Plan-Eintrag MUSS die folgenden Abschnitte enthalten.

### 5.1 Context

Metadaten darueber, wann und wo dieser Plan erstellt wurde.

| Feld | Erforderlich | Beschreibung |
|------|-------------|--------------|
| Date | Ja | `YYYY-MM-DD` |
| Branch | Ja | Git-Branch-Name |
| Agent | Nein | Wer den Plan erstellt hat (Standard: „AI") |
| Lock version | Nein | Aktuelle Lock-Version |
| PLS | Nein | Aktueller PLS-Score |
| Scope | Nein | Kurze Scope-Beschreibung |

### 5.2 Goal

Ein bis drei Saetze, die beschreiben, was dieser Plan erreichen soll. Dies ist das Erfolgskriterium — wenn das Ziel erreicht ist, ist der Plan abgeschlossen.

Regeln:
- MUSS spezifisch und messbar sein
- SOLLTE Lock-Konzepte (Entities, Features) referenzieren, wenn zutreffend
- Maximal ein bis drei Saetze

### 5.3 Approach

Uebergeordnete Strategie, die das WIE und WARUM erklaert. Erwaehnt betrachtete Alternativen, wenn relevant.

### 5.4 Steps

Geordnete, nummerierte Liste von Implementierungsschritten.

Regeln:
- Nummeriert, geordnet
- Jeder Schritt SOLLTE wenn moeglich unabhaengig verifizierbar sein
- Ziel: 3–15 Schritte
- Imperativform verwenden: „X hinzufuegen", „Y aktualisieren", „Z konfigurieren"

---

## 6. Optionale Abschnitte — Allgemein

Diese Abschnitte einbeziehen, wenn sie fuer den Plan relevant sind.

### 6.1 Background

Zusaetzlicher Kontext: Vorarbeiten, verwandte Issues, Benutzeranfragen.

### 6.2 Constraints

Bekannte Einschraenkungen oder Anforderungen, die den Ansatz formen.

```markdown
### Constraints
- Bestehende API darf nicht beschaedigt werden
- Antwortzeit unter 200ms
- Muss mit Node 18+ funktionieren
```

### 6.3 Risks

Potenzielle Probleme und deren Minderung.

```markdown
### Risks
- Stripe-Webhook-Zustellung kann verzoegert sein — Idempotenz-Schluessel implementieren
- Datenbankmigration grosser Tabelle — waehrend Schwachlastzeiten ausfuehren
```

### 6.4 Expected Outcome

Wie die Codebasis nach der Ausfuehrung des Plans aussehen soll. Nuetzlich fuer die Verifizierung durch den Reviewer.

### 6.5 References

Links zu Issues, PRs, Spezifikationen oder externen Ressourcen.

---

## 7. Optionale Abschnitte — Vollstaendige Blaupause

Fuer umfassende Produktplaene (als Ersatz fuer die alte `product.plan.md`) koennen diese zusaetzlichen Abschnitte enthalten sein:

| Abschnitt | Wann einzubeziehen |
|-----------|-------------------|
| Stack | Technologiewahl auf jeder Ebene |
| Schemas | Vollstaendige Entitaetsdefinitionen mit Typen, Einschraenkungen, Relationen |
| Endpoints | API-Vertraege fuer jedes Feature |
| Architecture | Nicht-triviale Systemmuster (Microservices, CQRS usw.) |
| Auth & Permissions | Auth-Methode, Sitzungsverwaltung, Rollenzuordnung, RLS |
| File Structure | Nicht-standardmaessiges Projektlayout |

Diese Abschnitte folgen demselben Format wie in der [Plan-Spezifikation](/de/plan-spec) beschrieben.

---

## 8. Plan vs Worklog

| Aspekt | Plan | Worklog |
|--------|------|---------|
| Zeitpunkt | Vor/waehrend der Implementierung | Nach der Implementierung |
| Zweck | Absicht und Ansatz (WIE & WARUM) | Tatsaechliche Aenderungen (WAS & WANN) |
| Inhalt | Ziel, Ansatz, Schritte, Risiken | Aenderungen, Dateien, Entscheidungen |
| Granularitaet | Flexibel (Blaupause bis Aufgabe) | Pro Sitzung |
| Verifizierung | Reviewer prueft ob Code mit Plan uebereinstimmt | Reviewer prueft ob Worklog mit Aenderungen uebereinstimmt |
| Verzeichnis | `plans/` | `worklogs/` |
| Benennung | `{YYYY-MM-DD}-{kebab-title}.md` | `{YYYY-MM-DD}-{kebab-title}.md` |

### Reviewer-Integration

Bei der Ueberpruefung eines Produkts vergleicht der KI-Reviewer Plaene mit der Codebasis und berichtet:

| Status | Bedeutung |
|--------|-----------|
| MATCH | Implementierung stimmt mit dem Plan ueberein |
| DEVIATION | Implementierung weicht erheblich vom Plan ab |
| INCOMPLETE | Plan ist nur teilweise implementiert |
| UNPLANNED | Bedeutender Code existiert ohne entsprechenden Plan |

---

*Diese Spezifikation ist Teil der [Product Lock Spezifikation](https://spec.productlock.org).*
