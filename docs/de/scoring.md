---
layout: default
title: Bewertung
nav_order: 7
permalink: /de/scoring
lang: de
lang_label: Deutsch
page_id: scoring
---

# Product Lock Bewertung

### Version 0.1.0 — Februar 2026

> Ein Komplexitaetsbewertungssystem, abgeleitet aus product.lock.json.
>
> Gleiches Lock, gleicher Score. Der Score misst Produktkomplexitaet, nicht Codekomplexitaet.

Die vollstaendige Spezifikation findest du in der [Product Lock Spezifikation](https://spec.productlock.org).

---

## Inhaltsverzeichnis

1. [Was gemessen wird](#1-was-gemessen-wird)
2. [Vier Dimensionen](#2-vier-dimensionen)
3. [Formel](#3-formel)
4. [Score-Stufen](#4-score-stufen)
5. [Berechnungsregeln](#5-berechnungsregeln)
6. [Benchmark](#6-benchmark)
7. [Anwendungsfaelle](#7-anwendungsfaelle)

---

## 1. Was gemessen wird

Product Lock Score (PLS) misst die **Produktgrenzen-Komplexitaet** eines Softwareprodukts. Er beantwortet: "Wie komplex ist dieses Produkt aus Produktperspektive?"

Was PLS misst:
- Wie viele Daten das Produkt verwaltet
- Wie viele Faehigkeiten es hat
- Wie vernetzt seine Teile sind
- Wie komplex seine Zugriffskontrolle ist

Was PLS NICHT misst:
- Codequalitaet oder Codeumfang
- Technische Schwierigkeit der Implementierung
- Leistungsanforderungen
- Infrastrukturkomplexitaet

**Der Score wird vollstaendig aus product.lock.json abgeleitet.** Gleiches Lock, gleicher Score. Sprache, Framework und Architektur beeinflussen den Score nicht.

---

## 2. Vier Dimensionen

Produktkomplexitaet laesst sich in vier unabhaengige Dimensionen aufteilen:

### D — Datenkomplexitaet

Wie viel das Produkt speichert. Mehr Entitaeten mit mehr Feldern = hoehere Datenkomplexitaet.

```
D = entity_count × avg_fields_per_entity × 0.3
```

Ein Produkt mit 45 Entitaeten und durchschnittlich 14 Feldern (Sentry) hat grundlegend mehr Datenkomplexitaet als eines mit 7 Entitaeten und durchschnittlich 6 Feldern (Hacker News).

### F — Funktionale Komplexitaet

Was das Produkt kann. Jedes Feature ist eine eigenstaendig identifizierbare Faehigkeit.

```
F = feature_count × 1.0
```

Gewichtet mit 1.0, weil Features das direkteste Mass fuer das sind, was ein Produkt tut. Ein Produkt mit 180 Features (GitLab) tut mehr als eines mit 12 Features (Hacker News).

### I — Interaktionskomplexitaet

Wie die Teile zusammenhaengen. Stories beschreiben entitaetsuebergreifende Ablaeufe — je mehr Stories, desto vernetzter das Produkt.

```
I = story_count × 0.5
```

Gewichtet mit 0.5, weil Stories erzaehlerisch sind (eine Story kann mehrere Interaktionen abdecken) und ihre Anzahl weniger praezise ist als die von Entitaeten oder Features.

### A — Zugriffskomplexitaet

Wer was tun darf. Mehr Akteure mit mehr unterschiedlichen Berechtigungssets = hoehere Zugriffskontrollkomplexitaet.

```
A = permission_matrix_size × 0.1
```

Berechtigungsmatrixgroesse = Anzahl sinnvoller Akteur-Feature-Kombinationen. Gewichtet mit 0.1, weil Zugriffskontrolle organisatorische Komplexitaet hinzufuegt, aber nicht aendert, was das Produkt tut.

---

## 3. Formel

```
PLS = D + F + I + A

Wobei:
  D = entity_count × avg_fields_per_entity × 0.3
  F = feature_count × 1.0
  I = story_count × 0.5
  A = permission_matrix_size × 0.1
```

### Wie jedes Feld aus product.lock.json gelesen wird

| Variable | Quelle | Wie zaehlen |
|----------|--------|-------------|
| `entity_count` | `entities` | Anzahl der Entitaetsschluessel (lockerer oder strikter Modus) |
| `avg_fields_per_entity` | `entities` | Durchschnittliche Laenge der Feld-Arrays (nur strikter Modus) |
| `feature_count` | `features` | Laenge des Features-Arrays |
| `story_count` | `stories` | Laenge des Stories-Arrays |
| `permission_matrix_size` | `permissions` | Summe aller Berechtigungs-Arrays ueber alle Akteure |

### Umgang mit fehlenden Feldern

| Situation | Regel |
|-----------|------|
| `entities` im lockeren Modus (`string[]`) | Standard verwenden: 8 Felder pro Entitaet |
| `entities` im strikten Modus mit `[]` | Als 0 Felder fuer diese Entitaet zaehlen |
| `features` weggelassen | F = 0 |
| `stories` weggelassen | I = 0 |
| `permissions` weggelassen | A = 0 |
| `permissions` als String (`"rbac"`) | Standard verwenden: actors × features × 0.6 |
| `actors` weggelassen | 1 verwenden (einzelner Benutzertyp) |
| `denied` | Nicht im Score enthalten (Grenzenreife, nicht Komplexitaet) |

### Warum `denied` ausgeschlossen wird

`denied` misst Grenzenreife, nicht Produktkomplexitaet. Ein Produkt mit 50 abgelehnten Elementen ist nicht komplexer als eines mit 0 — es ist staerker eingeschraenkt. Einschraenkung ist das Gegenteil von Komplexitaet.

---

## 4. Score-Stufen

| PLS-Bereich | Stufe | Beschreibung | Beispiele |
|-----------|-------|-------------|----------|
| < 50 | **Einfach** | Einzelzweck-Tool, minimales Datenmodell, wenige Features | Hacker News (32), Excalidraw (44) |
| 50–150 | **Moderat** | Fokussiertes Produkt mit klarer Domaene, moderates Datenmodell | Plausible (58), Ghost (145) |
| 150–300 | **Komplex** | Multi-Feature-Plattform, reiches Datenmodell, entitaetsuebergreifende Interaktionen | Cal.com (162), Discourse (170), Supabase (250) |
| 300–500 | **Sehr komplex** | Enterprise-Plattform, umfangreiches Datenmodell, tiefes Berechtigungssystem | Elasticsearch (287), Sentry (325) |
| 500+ | **Massiv** | Multi-Produkt-Plattform, Hunderte von Entitaeten und Features | GitLab (1037) |

### Was die Stufen in der Praxis bedeuten

| Stufe | Menschliche Pruefzeit | KI-Generierungsumfang | Typische Lock-Groesse |
|-------|------------------|--------------------| ------------------|
| Einfach | Sekunden | Einzelner Prompt | < 30 Zeilen |
| Moderat | 1–3 Minuten | Einzelne Sitzung | 30–80 Zeilen |
| Komplex | 3–10 Minuten | Mehrere Sitzungen | 80–200 Zeilen |
| Sehr komplex | 10–30 Minuten | Phasenweise Generierung | 200–500 Zeilen |
| Massiv | 30+ Minuten | Team von KI-Agenten | 500+ Zeilen |

---

## 5. Berechnungsregeln

### Regel 1: Nur Produktentitaeten zaehlen

Nur Entitaeten zaehlen, die im `entities`-Feld erscheinen. Implementierungstabellen (Caches, Queues, Migrationen) sollten nicht im Lock sein und beeinflussen daher den Score nicht.

### Regel 2: Strikter Modus ergibt hoehere Scores

Der strikte Modus sperrt mehr Felder und erzeugt einen hoeheren (und genaueren) D-Score. Dies ist beabsichtigt — ein Produkt, das exakte Felder spezifiziert, hat ein definierteres (und damit messbares) Datenmodell.

```json
// Locker: entity_count=3, avg_fields=8 (Standard)
// D = 3 × 8 × 0.3 = 7.2
"entities": ["User", "Post", "Comment"]

// Strikt: entity_count=3, avg_fields=10
// D = 3 × 10 × 0.3 = 9.0
"entities": {
  "User": ["avatar", "email", "id", "name", "role"],
  "Post": ["authorId", "content", "createdAt", "id", "slug", "status", "tags", "title"],
  "Comment": ["authorId", "content", "createdAt", "id", "postId", "replyToId", "status"]
}
```

### Regel 3: Berechtigungsmatrix zaehlt tatsaechliche Eintraege

Wenn permissions ein Objekt ist, die Gesamtzahl der Berechtigungseintraege zaehlen:

```json
"permissions": {
  "Admin": ["createPost", "deletePost", "manageUsers", "publishPost"],
  "Editor": ["createPost", "publishPost"],
  "Viewer": ["viewPost"]
}
// permission_matrix_size = 4 + 2 + 1 = 7
```

Wenn permissions ein String wie `"rbac"` ist, schaetzen: `actors × features × 0.6`.

### Regel 4: Score ist deterministisch

Gleiches Lock erzeugt immer denselben Score. Kein Zufall, keine externen Daten, kein subjektives Urteil. Die Formel verwendet ausschliesslich Daten aus der Lock-Datei.

### Regel 5: Auf Ganzzahl runden

Der endgueltige PLS wird auf die naechste Ganzzahl gerundet.

---

## 6. Benchmark

Validiert anhand von 10 bekannten Open-Source-Produkten. Alle Scores stimmen mit der oeffentlich akzeptierten Komplexitaetsrangfolge ueberein.

| # | Produkt | Entities | AvgFields | Features | Stories | PermMatrix | D | F | I | A | **PLS** | Stufe |
|---|---------|----------|-----------|----------|---------|------------|---|---|---|---|---------|-------|
| 1 | Hacker News | 7 | 6 | 12 | 10 | 26 | 12.6 | 12 | 5 | 2.6 | **32** | Einfach |
| 2 | Excalidraw | 8 | 10 | 14 | 10 | 10 | 24 | 14 | 5 | 1 | **44** | Einfach |
| 3 | Plausible | 12 | 8 | 18 | 15 | 36 | 28.8 | 18 | 7.5 | 3.6 | **58** | Moderat |
| 4 | Ghost | 22 | 12 | 34 | 45 | 90 | 79.2 | 34 | 22.5 | 9 | **145** | Moderat |
| 5 | Cal.com | 26 | 11 | 38 | 55 | 105 | 85.8 | 38 | 27.5 | 10.5 | **162** | Komplex |
| 6 | Discourse | 28 | 10 | 42 | 60 | 135 | 84 | 42 | 30 | 13.5 | **170** | Komplex |
| 7 | Supabase | 40 | 11 | 78 | 35 | 220 | 132 | 78 | 17.5 | 22 | **250** | Komplex |
| 8 | Elasticsearch | 35 | 12 | 90 | 45 | 480 | 126 | 90 | 22.5 | 48 | **287** | Komplex |
| 9 | Sentry | 45 | 14 | 75 | 60 | 310 | 189 | 75 | 30 | 31 | **325** | Sehr komplex |
| 10 | GitLab | 120 | 18 | 180 | 250 | 840 | 648 | 180 | 125 | 84 | **1037** | Massiv |

### Validierung

Die Bewertung erzeugt eine Rangfolge, die mit allgemein akzeptierten Komplexitaetswahrnehmungen uebereinstimmt:

```
Hacker News (32) < Excalidraw (44) < Plausible (58) < Ghost (145) < Cal.com (162)
< Discourse (170) < Supabase (250) < Elasticsearch (287) < Sentry (325) < GitLab (1037)
```

Wichtige Beobachtungen aus dem Benchmark:

- **Einfache Produkte** (< 50): Einzelzweck, < 10 Entitaeten, < 15 Features
- **Moderate Produkte** (50–150): fokussierte Domaene, 10–25 Entitaeten, 15–35 Features
- **Komplexe Produkte** (150–300): Multi-Feature-Plattformen, 25–40 Entitaeten, 35–80 Features
- **Sehr komplexe Produkte** (300–500): Enterprise-Plattformen, 35–50 Entitaeten, 75–90 Features, tiefe Berechtigungsmodelle
- **Massive Produkte** (500+): Multi-Produkt-Plattformen, 100+ Entitaeten, 150+ Features, 200+ Stories

---

## 7. Anwendungsfaelle

### Produkte vergleichen

> "Ist unser Produkt komplexer als Ghost?" Beide Produkte locken. Scores vergleichen.

### Komplexitaetswachstum verfolgen

> Version 1.0: PLS 85 (Moderat)
> Version 2.0: PLS 160 (Komplex)
> Version 3.0: PLS 340 (Sehr komplex)

Wenn der Score unerwartet springt, kann das auf Scope Creep hindeuten. Den Diff zwischen Lock-Versionen pruefen, um herauszufinden, was hinzugefuegt wurde.

### Aufwand schaetzen

Der Score korreliert mit dem Implementierungsaufwand. Ein komplexes Produkt (150–300) erfordert deutlich mehr Arbeit als ein einfaches (< 50). Die Benchmark-Produkte als Referenzpunkte verwenden:

- "Unser Produkt hat Score 170, aehnlich wie Discourse. Vergleichbarer Umfang zu erwarten."
- "Unser Produkt hat Score 45, aehnlich wie Excalidraw. Dies ist ein fokussiertes Tool."

### Umfangskontrolle

Einen Ziel-PLS fuer eine Produktversion setzen:

> "MVP muss unter PLS 100 bleiben. Alles, was uns darueber bringt, wird auf v2 verschoben."

Wenn eine Feature-Ergaenzung den Score ueber das Ziel hinaus erhoeht, ist das ein Signal, den Umfang zu ueberdenken.

### Lock-Versionen vergleichen

```
v1.0.0 → v1.1.0: PLS 120 → 128 (+8)     // Kleinere Feature-Ergaenzungen
v1.1.0 → v2.0.0: PLS 128 → 195 (+67)     // Groessere Umfangserweiterung
v2.0.0 → v2.1.0: PLS 195 → 210 (+15)     // Moderate Ergaenzungen
v2.1.0 → v3.0.0: PLS 210 → 340 (+130)    // Warnsignal: Umfang koennte ausufern
```

---

*Dieses Bewertungssystem ist Teil der [Product Lock Spezifikation](https://spec.productlock.org).*
