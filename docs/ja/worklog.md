---
layout: default
title: Worklog
nav_order: 8
permalink: /ja/worklog
lang: ja
lang_label: 日本語
page_id: worklog
---

# Product Lock ワークログ仕様書

### バージョン 0.1.0 — 2026年2月

> product.lock.json に対する変更を追跡する構造化された開発ログ。
>
> Lock は WHAT を定義する。Plan は HOW を定義する。Score は複雑度を測定する。**Worklog は WHY と WHEN を記録する。**
>
> この4つのファイルが、セッションをまたぎ、エージェントをまたぎ、時間をまたいで、完全なプロジェクトコンテキストを再構築する。

完全な仕様については、[Product Lock Specification](https://spec.productlock.org) を参照のこと。

---

## 目次

1. [目的](#1-目的)
2. [クローズドループ](#2-クローズドループ)
3. [ファイル形式](#3-ファイル形式)
4. [必須セクション](#4-必須セクション)
5. [任意セクション](#5-任意セクション)
6. [エントリのライフサイクル](#6-エントリのライフサイクル)
7. [コンテキストの再構築](#7-コンテキストの再構築)
8. [規約](#8-規約)

---

## 1. 目的

AIセッションは一時的である。コンテキストウィンドウは圧縮され、会話は期限切れになり、エージェントは記憶を失う。しかしプロダクトの意思決定は永続する。ワークログは重要なことを記録する：

- **何が変わったか** — 変更されたファイル、追加されたエンティティ、実装された機能
- **なぜ変わったか** — 意思決定、トリガー、制約
- **それが何を意味するか** — ロックの変更、複雑度の差分、スコープへの影響
- **何が検出されたか** — レビュワーの発見事項、修正された違反、適用された境界

ワークログがなければ、各AIセッションはゼロから始まる。ワークログがあれば、任意のエージェントがロック + ワークログを読んでプロジェクトの軌跡を再構築できる。

### ワークログでないもの

- git ログではない（git はファイル変更を追跡し、ワークログはプロダクトの意思決定を追跡する）
- チェンジログではない（チェンジログはユーザー向け、ワークログは開発者とAI向け）
- 日記ではない（意見もコメントもなし — 構造化データのみ）

---

## 2. クローズドループ

Product Lock の4つのファイルはクローズドなフィードバックループを形成する：

```
product.lock.json ──→ product.plan.md ──→ 実装 ──→ product.worklog.md
       ↑                                                            │
       └────────────────────────────────────────────────────────────┘
                         （ワークログの発見事項に基づいてロックが更新される）
```

| ファイル | 役割 | 答える問い |
|------|------|---------|
| `product.lock.json` | 境界 | このプロダクトは何か？ |
| `product.plan.md` | ブループリント | どう構築するか？ |
| `product-lock-scoring.md` | 測定 | どのくらい複雑か？ |
| `product.worklog.md` | 履歴 | 何が、いつ、なぜ起きたか？ |

各開発セッション：

1. **開始前**：ロック + 最新のワークログエントリを読む → 現在の状態を理解する
2. **実施中**：変更、意思決定、ロックの変更を追跡する
3. **完了後**：ワークログエントリを書く → レビュワーを実行する → 発見事項を記録する
4. **クローズ**：必要に応じてロックを更新する → スコアを再計算する → 差分を記録する

すべての変更が追跡可能なとき、ループは閉じられる：それを承認したロックから、実装したセッションを経て、検証したレビュワーまで。

---

## 3. ファイル形式

ワークログはプロジェクトルートの `worklogs/` ディレクトリに `product.lock.json` と並べて保存する。各セッションが1ファイル。

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

### ファイル命名

```
{date}-{kebab-case-title}.md
```

| 要素 | フォーマット | 例 |
|------|--------|---------|
| 日付 | `YYYY-MM-DD` | `2026-02-17` |
| 日付（同日） | `YYYY-MM-DDTHHmm` | `2026-02-17T1430` |
| タイトル | kebab-case、60文字以内 | `add-payment-system` |

例：
```
2026-02-16-initial-release.md
2026-02-17-add-payment-system.md
2026-02-17T1430-fix-payment-webhook.md
2026-02-18-refactor-auth-middleware.md
```

### ファイルの並び順

ファイルは名前で自然にソートされる（日付プレフィックスが時系列順を保証する）。最新のエントリを見つけるには、降順ソートするか最後のファイルを確認する。

### 1ファイル = 1セッション

1ファイル = 1セッション = 1つの焦点を絞った作業ブロック。各ファイルは以下のテンプレートに従う：

```markdown
# [{date}] {title}

### Context
- Branch: `feature/xxx`
- Agent: {作業を行った者}
- Lock version: {このセッション前のバージョン}
- PLS before: {スコア}

### Goal
{一文：このセッションの目的。}

### Changes

| Action | Target | Detail |
|--------|--------|--------|
| added | Entity: Payment | 4 fields: id, amount, userId, createdAt |
| modified | Feature: checkout | Added Stripe integration |
| removed | Feature: manualInvoice | Replaced by automated billing |

### Lock Delta
{product.lock.json に何が変わったか（変わった場合）。}

### Score Delta
- PLS: {before} → {after} ({+/-delta})
- Breakdown: D {before}→{after}, F {before}→{after}, I {before}→{after}, A {before}→{after}

### Review Findings
{このセッション後の AI Reviewer の結果。}

### Decisions
{重要な意思決定とその理由。}

### Files
| Action | Path |
|--------|------|
| created | src/models/payment.ts |
| modified | src/routes/checkout.ts |
| deleted | src/routes/manual-invoice.ts |

### Next
{次のセッションで取り組むべきこと。}
```

---

## 4. 必須セクション

すべてのワークログエントリは以下のセクションを含まなければならない（MUST）。

### 4.1 Context

セッションのメタデータ。開始状態を確立する。

```markdown
### Context
- Branch: `feature/payment-system`
- Agent: Claude (Opus 4)
- Lock version: 1.2.0
- PLS before: 145
```

| フィールド | 必須 | 説明 |
|-------|----------|-------------|
| Branch | はい | このセッションの Git ブランチ |
| Agent | はい | 作業を行った者（AIモデル、人間名、または両方） |
| Lock version | はい | セッション開始時の product.lock.json のバージョン |
| PLS before | はい | セッション開始時の Product Lock Score |

### 4.2 Goal

セッションの目的を記述する一文。これが契約であり、エントリ内のすべてはこの目標に関連すべきである。

```markdown
### Goal
Add payment processing with Stripe so Members can pay for premium features.
```

ルール：
- 一文でなければならない（MUST）
- 完了を検証できるほど具体的でなければならない（MUST）
- 該当する場合、ロックの概念（entities、features、actors）を参照すべきである（SHOULD）

### 4.3 Changes

セッション中に行われたプロダクトレベルの変更のテーブル。これはファイル差分ではなく、プロダクト境界の差分である。

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

アクション：
- `added` — プロダクト境界の新規項目
- `modified` — 既存項目の変更
- `removed` — プロダクト境界からの削除
- `unchanged` — 変更されなかったことの明示的な記録（関連する場合）

ターゲットの形式：`{type}: {name}` ここで type は Entity、Feature、Actor、Permission、Denied、Story のいずれか。

### 4.4 Review Findings

セッションの変更後の AI Reviewer の結果。レビューが実行されなかった場合は、その旨を明記する。

```markdown
### Review Findings

Reviewer: Claude (Sonnet 4.5)
Status: PASS (2 warnings)

- [PASS] Entity Payment — model exists with correct fields
- [PASS] Feature createPayment — POST /api/payments endpoint
- [WARN] Undeclared entity: PaymentLog (exists in code, not in lock)
- [WARN] Undeclared feature: retryPayment (exists in code, not in lock)
```

または：

```markdown
### Review Findings

No review conducted this session.
```

ルール：
- レビュワーを明記しなければならない（MUST）（AIモデルまたは人間）
- 全体のステータスを含めなければならない（MUST）：PASS、FAIL、または NOT RUN
- すべての違反と警告をリストしなければならない（MUST）
- FAIL の場合：エントリは Next セクションにフォローアップを含めなければならない（MUST）

### 4.5 Files

セッション中に作成、変更、削除されたファイル。

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

ルール：
- 重要なものだけでなく、触れたすべてのファイルをリストする
- プロジェクトルートからの相対パスを使用する
- アクション：`created`、`modified`、`deleted`
- アクション順にソート（created → modified → deleted）、各グループ内はアルファベット順

---

## 5. 任意セクション

有用なコンテキストを提供する場合にこれらのセクションを含める。

### 5.1 Lock Delta

このセッション中に product.lock.json に何が変わったか。ロックが変更されなかった場合はスキップする。

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

ルール：
- バージョン変更を明記しなければならない（MUST）
- Added / Modified / Removed でグループ化する
- ロックのフィールド形式を使用する（エンティティ名は PascalCase、機能は camelCase など）

### 5.2 Score Delta

Product Lock Score がどう変わったか。ロックが変更されなかった場合はスキップする。

```markdown
### Score Delta

- PLS: 145 → 172 (+27)
- Level: Moderate → Complex
- Breakdown:
  - D: 79.2 → 94.8 (+15.6) — 2 new entities
  - F: 34 → 38 (+4) — 2 new features + 2 existing
  - I: 22.5 → 27.5 (+5) — added payment stories
  - A: 9 → 11.7 (+2.7) — Subscriber role added
```

スコアがレベル境界を越えた場合（例：Moderate → Complex）、明示的に記載すべきである（SHOULD）。レベルの遷移はスコープレビューのシグナルである。

### 5.3 Decisions

セッション中に行われた重要な意思決定とその根拠。重要な意思決定がなかった場合はスキップする。

```markdown
### Decisions

1. **Stripe over Paddle**: Stripe has better API for subscription management. Paddle bundles tax handling but we don't need it for MVP.

2. **No refunds in v1**: Added to denied list. Manual refund via Stripe dashboard is sufficient. Automated refunds add liability and edge cases (partial refunds, proration).

3. **Subscription as separate entity**: Could have been a field on User, but Subscription has its own lifecycle (start, end, cancel, renew) that justifies a separate entity.
```

ルール：
- 各意思決定に番号を付ける
- 意思決定のタイトルを太字にする
- 根拠（WHY）を含める
- 意思決定が denied リストを変更する場合はその旨を記載する

### 5.4 Next

次のセッションで取り組むべきこと。プロジェクトが完了した場合のみスキップする。

```markdown
### Next

- [ ] Implement webhook handler for Stripe events (payment_intent.succeeded, subscription.canceled)
- [ ] Add PaymentLog to lock or remove from codebase (reviewer warning)
- [ ] Write stories for payment flow and add to lock
- [ ] Run full review after webhook implementation
```

ルール：
- アクション可能な項目にはチェックボックス形式を使用する
- 該当する場合、レビュワーの発見事項を参照する
- 別のエージェントがこれを引き継げるほど具体的にする

### 5.5 Commits

セッション中に行われた Git コミット。コミットがなかった場合（例：計画のみのセッション）はスキップする。

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

## 6. エントリのライフサイクル

ワークログエントリは開発セッション内で以下のライフサイクルに従う：

```
セッション開始
│
├─ 1. 最新のワークログエントリを読む → 現在の状態を理解する
├─ 2. ロックを読む → プロダクト境界を理解する
├─ 3. Plan を読む → 実装の判断を理解する
│
├─ 4. 作業を行う
│   ├─ 変更を追跡する（entities、features、actors）
│   ├─ 変更されたファイルを追跡する
│   └─ 意思決定が発生したら記録する
│
├─ 5. プロダクト境界が変わった場合はロックを更新する
├─ 6. ロックが変わった場合はスコアを再計算する
├─ 7. 更新されたロックに対してレビュワーを実行する
│
├─ 8. ワークログエントリを書く
│   ├─ Context（ブランチ、エージェント、ロックバージョン、PLS）
│   ├─ Goal（目的は何だったか）
│   ├─ Changes（プロダクトレベルの変更）
│   ├─ Lock Delta（ロックが変更された場合）
│   ├─ Score Delta（スコアが変わった場合）
│   ├─ Review Findings（レビュワーの結果）
│   ├─ Decisions（重要な意思決定 + 根拠）
│   ├─ Files（触れたすべてのファイル）
│   ├─ Commits（git コミット）
│   └─ Next（次に取り組むべきこと）
│
セッション終了
```

### エントリを作成するタイミング

- 開発セッションごとに1エントリ（セッション = 1つの焦点を絞った作業ブロック）
- 計画のみでコードを書かないセッションもエントリを作成する（Files セクションなし）
- レビューのみで何も変更しないセッションもエントリを作成する（Review Findings あり）
- 1回の作業で複数の小さな修正 = 1エントリ
- 複数の作業にまたがる大きな機能 = 複数のエントリ

### エントリのサイズ

エントリは簡潔に保つ。典型的なエントリは30-80行。エントリが150行を超える場合、セッションが大きすぎたため、次回は複数のエントリに分割する。

---

## 7. コンテキストの再構築

ワークログの主な価値：任意のエージェントが、会話履歴を再生する代わりに構造化ファイルを読むことでプロジェクトコンテキストを再構築できる。

### ワークログなしの場合

```
新しいAIセッション → コードベースを読む（数千のファイル） → 意図を推測する → しばしば間違える
```

### ワークログありの場合

```
新しいAIセッション → ロックを読む（プロダクト境界）
                  → 最新のワークログを読む（現在の状態、最近の意思決定、次のステップ）
                  → Plan を読む（実装アプローチ）
                  → 完全なコンテキストで作業を開始する
```

### 各ファイルが再構築に貢献するもの

| 問い | 答えるファイル |
|----------|-------------|
| このプロダクトは何か？ | `product.lock.json` |
| 正確な境界は？ | `product.lock.json`（denied） |
| どのくらい複雑か？ | PLS スコア（ワークログ内） |
| どう構築されているか？ | `product.plan.md` |
| 最近何があったか？ | `product.worklog.md`（最新のエントリ） |
| なぜ X が決定されたか？ | `product.worklog.md`（Decisions） |
| 次に何をすべきか？ | `product.worklog.md`（Next） |
| レビュワーは何を発見したか？ | `product.worklog.md`（Review Findings） |
| スコープクリープしていないか？ | `product.worklog.md`（時系列の Score Delta） |

### トークン効率

2時間のAI会話は100K以上のトークンのコンテキストを消費する可能性がある。同じセッションのワークログエントリは約50行の構造化 Markdown である。次のセッションは100Kトークンの再生の代わりに50行を読む。

```
会話履歴:     100,000+ トークン（圧縮、非可逆、一時的）
ワークログエントリ:   ~800 トークン（構造化、完全、永続的）
```

### スコープドリフトの検知

エントリをまたいで PLS を追跡することで、スコープクリープが可視化される：

```
[2026-02-10] v1.0.0 — PLS: 85  (中程度)
[2026-02-12] v1.1.0 — PLS: 92  (+7, 小規模な追加)
[2026-02-14] v1.2.0 — PLS: 145 (+53, 大幅な増加 ⚠️)
[2026-02-17] v1.3.0 — PLS: 172 (+27, 複雑レベルに突入)
```

1セッションで +50 以上の増加、またはレベル境界の越境は、スコープレビューをトリガーすべきである（SHOULD）。

---

## 8. 規約

### 8.1 ディレクトリ構成

- ディレクトリ：プロジェクトルートの `worklogs/`
- プロダクトごとに1ディレクトリ（プロダクトごとに1ロックと対応）
- 各ファイルが1セッション

```
project-root/
├── product.lock.json
└── worklogs/
    ├── 2026-02-16-initial-release.md
    └── 2026-02-17-add-payment-system.md
```

### 8.2 ファイル命名

```
{YYYY-MM-DD}-{kebab-case-title}.md
```

- 日付プレフィックスが時系列ソートを保証する
- タイトルは命令形、kebab-case、60文字以内
- 同日に複数セッションがある場合は `{YYYY-MM-DD}T{HHmm}-{title}.md` を使用する

### 8.3 タイトル

命令形で簡潔に：

| 良い例 | 悪い例 |
|------|-----|
| `add-payment-system` | `added-the-payment-system-and-also-fixed-some-bugs` |
| `fix-auth-middleware` | `authentication-middleware-was-broken` |
| `remove-legacy-billing` | `cleaned-up-old-billing-code` |

### 8.4 相互参照

他のエントリを参照する場合はファイル名を使用する：

```markdown
Continues from `2026-02-14-setup-stripe-integration.md`.
See `2026-02-12-define-payment-entities.md` for schema decisions.
```

### 8.5 命名規約

ワークログ内ではロックの規約に従う：

| 項目 | 形式 | 例 |
|------|--------|---------|
| エンティティ名 | PascalCase | `Entity: Payment` |
| 機能名 | camelCase | `Feature: createPayment` |
| アクター名 | PascalCase | `Actor: Subscriber` |
| ファイルパス | ルートからの相対パス | `src/models/payment.ts` |

### 8.6 読む順序

コンテキストを再構築するために、AIエージェントは以下の順序で読むべきである（SHOULD）：

1. **最新の**ワークログファイルを読む（ファイル名ソートで最後のもの）
2. **Next** セクションで保留中の作業を確認する
3. 追加のコンテキストが必要な場合は最近のファイルをさらに読む
4. プロダクト境界のためにロックを読む

---

*この仕様は [Product Lock Specification](https://spec.productlock.org) の一部です。*
