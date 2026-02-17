---
layout: default
title: Generator Guide
nav_order: 4
permalink: /ja/generator-guide
lang: ja
lang_label: 日本語
page_id: generator-guide
---

# Product Lock ジェネレーターガイド

### バージョン 0.1.0 — 2026年2月

> あなたは AI Worker である。あなたの仕事は、コードベースを分析し、プロダクトの境界を正確に記述する `product.lock.json` を生成することである。
>
> あなたがロックを生成する。人間がそれを承認する。AI Reviewer がコードをロックに照らして検証する。

完全な仕様については、[Product Lock Specification](https://spec.productlock.org) を参照のこと。

---

## 目次

1. [product.lock.json とは何か？](#1-productlockjson-とは何か)
2. [生成プロセス](#2-生成プロセス)
3. [フィールドリファレンス](#3-フィールドリファレンス)
4. [命名規約](#4-命名規約)
5. [バリデーションチェックリスト](#5-バリデーションチェックリスト)
6. [よくある間違い](#6-よくある間違い)
7. [完全な例](#7-完全な例)

---

## 1. product.lock.json とは何か？

`product.lock.json` は、ソフトウェアプロダクトが**何であり、何でないか**を、コードレベルではなくプロダクトレベルで定義する。

6つの質問に答える：

| フィールド | 質問 | 形式 |
|-------|----------|--------|
| `actors` | 誰がこのプロダクトを使うか？ | PascalCase の名詞 |
| `entities` | どんなデータを保存するか？ | PascalCase の名詞 |
| `features` | 何ができるか？ | camelCase の動詞 |
| `stories` | アクター、エンティティ、機能はどう相互作用するか？ | 自然言語の文 |
| `permissions` | 誰が何をできるか？ | アクターと機能のマトリクス |
| `denied` | 何を持ってはならないか？ | 明示的な除外 |

**ロックはプロダクト境界を記述し、実装は記述しない。** ルートも、依存関係も、フレームワークの選択も、ファイル構成もない。

---

## 2. 生成プロセス

### ステップ1：コードベースを読む

コードベースをスキャンして以下を理解する：

- データベーススキーマ / モデル / 型 → entities
- API ルート / ハンドラー / コントローラー → features
- 認証 / ロール定義 → actors、permissions
- UI ページ / ビュー → features、stories
- ミドルウェア / バックグラウンドジョブ / cron → stories
- テスト → どの機能が存在するかの確認に役立つ

### ステップ2：メタデータを抽出する

```json
{
  "name": "<kebab-case product name>",
  "version": "<semver, start with 0.1.0 if new>",
  "description": "<one-line product description>",
  "author": "<who will approve this lock>"
}
```

- `name`：プロジェクトフォルダ名または package.json から導出、常に kebab-case
- `description`：ユーザーの視点からプロダクトの目的を一文で記述する
- `author`：人間に確認するか、git のコミッター名を使用する

### ステップ3：アクターを特定する

アクターは**プロダクトを使う人々**であり、コードエンティティではない。

探すもの：
- 認証/RBAC コードのロール定義
- 異なる権限レベル
- UI における異なるユーザータイプ

```json
"actors": ["Admin", "Member", "Guest"]
```

ルール：
- PascalCase
- アルファベット順にソート
- コードエンティティではない（`UserService`、`AuthMiddleware` は不可）
- ユーザータイプが1つだけなら `["User"]` を使用する

### ステップ4：エンティティを抽出する

エンティティは**プロダクトが保存するもの** — データモデルである。

探すもの：
- データベーステーブル / Prisma モデル / TypeORM エンティティ
- Mongoose スキーマ / SQLAlchemy モデル
- `id` フィールドを持つ TypeScript インターフェース
- GraphQL 型

**判断：ルースモードかストリクトモードか？**

**ルースモード**を使う場合：
- 簡単な概要で十分な場合
- フィールドが標準的で予測可能な場合
- 人間が特定のフィールドをロックする必要がない場合

```json
"entities": ["Conversation", "Message", "User"]
```

**ストリクトモード**を使う場合：
- フィールドがプロダクトにとって重要な場合（例：金融データ）
- 人間が正確なデータモデルを検証する必要がある場合
- プロダクトに複雑または特殊なフィールド要件がある場合

```json
"entities": {
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "User": ["avatar", "email", "id", "name"]
}
```

ルール：
- エンティティ名は PascalCase
- フィールド名は camelCase
- アルファベット順にソート（キーとフィールド配列の両方）
- `[]` = エンティティは存在する、フィールドはロックされない
- フィールドは名前のみ — **型はなし**（型は実装の詳細）

**重要な判断：エンティティか、エンティティでないか？**

| エンティティに含める | 含めない |
|-------------------|---------------|
| User、Product、Order — ドメインデータ | CachedQuote、TempSession — 実装の詳細 |
| Message、Conversation — ユーザー向け | MigrationLog、AuditTrail — 運用上のもの |
| StockMetadata — プロダクトが保存するもの | RedisKey、QueueJob — インフラ |

何かがエンティティに見えるが実際には実装上のもの（キャッシュテーブル、一時ストレージ、ジョブキュー）であれば、**エンティティとして含めない**。代わりに、それが満たすプロダクト要件を**ストーリー**として記述する（ステップ6参照）。

### ステップ5：機能を抽出する

機能は**アトミックなプロダクトの能力** — プロダクトが実行できること。

探すもの：
- API エンドポイント（各主要エンドポイント = 1つの機能）
- UI アクション（ボタン、フォーム、ワークフロー）
- バックグラウンドジョブの機能
- システム連携

```json
"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

ルール：
- camelCase
- 動詞 + 名詞の形式：`sendMessage`、`createGroup`、`viewDashboard`
- アルファベット順にソート
- フラットリスト — サブ機能なし、ネストなし
- 各機能はコードベース内で独立して識別可能でなければならない（MUST）

**粒度のガイド：**

| 粗すぎる | 適切 | 細かすぎる |
|-----------|------|-----------|
| `manageMessages` | `sendMessage`、`deleteMessage` | `validateMessageLength` |
| `handleAuth` | `login`、`register`、`resetPassword` | `hashPassword` |
| `stockAnalysis` | `analyzeStock`、`viewStockChart` | `calculateMovingAverage` |

機能は**プロダクトマネージャー**がリストするものと対応すべきであり、**開発者**がリストするものではない。

### ステップ6：ストーリーを書く

ストーリーは**アクター、エンティティ、機能がどう相互作用するか**を記述する。プロダクトのナラティブである。

3つの種類：

**機能フロー** — プロダクト利用時に何が起こるか：
```
"Member sends Message to Conversation"
"Admin deletes any Message from Conversation"
"When Member reads Message, system creates ReadReceipt"
```

**体験の期待** — ユーザーが知覚するもの（非機能要件）：
```
"User views StockQuote with data refreshing in real-time"
"User views Dashboard with market data loading instantly"
```

**システムの振る舞い** — システムが自律的に行うこと：
```
"System fetches NewsArticle from multiple sources on schedule"
"System rate limits API requests per IP address"
"System calibrates model with Bayesian posteriors and detects concept drift"
```

ルール：
- ストーリーごとに一文
- アクター名（PascalCase、`actors` に存在しなければならない（MUST））または `"System"` で始める
- 正確な PascalCase 名でエンティティを参照する
- 現在形、能動態
- 何が起こるか（WHAT）を記述し、どう実装するか（HOW）ではない
- **ナラティブの流れ**で並べる（アルファベット順ではない — これが唯一の例外）

**エンティティからストーリーへの変換ルール：**

プロダクト目的を果たす実装の詳細を見つけた場合、ストーリーに変換する：

| 見つかった実装 | 書くべきストーリー |
|---------------------|---------------|
| `CachedQuote` テーブル | `"User views StockQuote with data refreshing in real-time"` |
| `CachedHistory` テーブル | `"User views StockChart with historical data loading instantly"` |
| レートリミッターミドルウェア | `"System rate limits API requests per IP address"` |
| エンベディングベクトルストア | `"System searches knowledge base by semantic similarity"` |
| バックグラウンド cron ジョブ | `"System fetches NewsArticle from multiple sources on schedule"` |

### ステップ7：パーミッションを定義する

パーミッションは**アクターと機能を結びつける** — 誰が何をできるか。

**適切なモードを選ぶ：**

プロダクトに認証がないか、基本的な認証のみの場合：
- フィールドを完全に省略する（AIが自由に決定する）

プロダクトが既知の認証モデルを使用する場合：
```json
"permissions": "rbac"
```

正確なロール-機能マッピングを指定する必要がある場合：
```json
"permissions": {
  "Admin": ["createGroup", "deleteMessage", "sendMessage"],
  "Guest": ["listConversations"],
  "Member": ["createGroup", "sendMessage"]
}
```

ルール：
- キーは PascalCase、`actors` と一致しなければならない（MUST）
- 値は camelCase の配列、`features` のサブセットであるべき（SHOULD）
- キーと値の配列の両方をアルファベット順にソート
- システム専用の機能（`fetchNews`、`enrichNewsWithAi` など）は permissions に入れない — ユーザーが呼び出すものではない

### ステップ8：denied 項目を決定する

**これが最も重要なステップである。** denied 項目は、AIがプロダクトスコープを超えることを防ぐガードレールである。

従来の開発では、プロダクトは人間の時間によって制約されていた。AIには自然な制限がない — 明示的に禁じられない限り、AIは機能、エンティティ、振る舞いを追加し続ける。`denied` がその線を引く方法である。

これは「プロダクトがまだ持っていないすべてのもの」ではない。不在が重要な**意図的なプロダクトの意思決定**のためのものである。

**判断フレームワーク：**

問い：「AIが誤ってこれを追加した場合、プロダクト違反になるか？」
- はい → denied に含める
- いいえ、まだ構築していないだけ → 含めない

**ルースモード** — フラットリスト：
```json
"denied": ["executeTrade", "Reaction", "voiceCall"]
```

**理由付き**（推奨） — 各項目に説明：
```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "Reaction": "Intentionally excluded to keep messaging simple",
  "voiceCall": "Text-based communication only"
}
```

理由は Reviewer がなぜ除外されているかを理解し、人間がより良い承認/拒否の判断を下すのに役立つ。

**denied 項目の3つのカテゴリ：**

| カテゴリ | 例 | 理由 |
|----------|---------|--------|
| プロダクトスコープ | `"voiceCall"` | 「テキストベースのコミュニケーションのみ」 |
| 責任範囲の境界 | `"executeTrade"` | 「証券仲介の責任を負わない」 |
| スコープのガードレール | `"Reaction"` | 「メッセージングをシンプルに保つ」 |

### ステップ9：組み立てとバリデーション

以下のキー順序で完全なロックを組み立てる：

```
$schema → name → version → description → author → license → keywords → private
→ actors → entities → features → stories → permissions → denied
```

バリデーションチェックリストを確認する（[セクション5](#5-バリデーションチェックリスト) 参照）。

### ステップ10：Markdown ビューを生成する

JSON ロックの後、人間がレビューするための `product.lock.md` を生成する：

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

## 3. フィールドリファレンス

### メタデータフィールド

| フィールド | 必須 | 形式 | 説明 |
|-------|----------|--------|-------------|
| `$schema` | いいえ | string | バリデーション用スキーマ URL |
| `name` | **はい** | kebab-case の文字列 | プロダクト識別子 |
| `version` | **はい** | semver の文字列 | プロダクトバージョン |
| `description` | **はい** | string | 一行のプロダクト説明 |
| `author` | **はい** | string | このロックを承認する人 |
| `license` | いいえ | string | ライセンス識別子 |
| `keywords` | いいえ | 小文字の string[] | 発見性のためのタグ |
| `private` | いいえ | boolean | 公開共有しない |

### プロダクト境界フィールド

すべて任意。省略 = AIが自由に決定する。

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `actors` | `string[]` | ユーザーロール |
| `entities` | `string[]` または `Record<string, string[]>` | データモデル |
| `features` | `string[]` | プロダクトの機能 |
| `stories` | `string[]` | インタラクションフローと振る舞い |
| `permissions` | `string` または `Record<string, string[]>` | アクセス制御 |
| `denied` | `string[]` または `Record<string, string>` | 明示的な除外 |

---

## 4. 命名規約

| フィールド | 規約 | 例 |
|-------|-----------|---------|
| `name` | kebab-case | `"chat-system"` |
| `actors` | PascalCase | `"Admin"`、`"Member"` |
| `entities` | PascalCase | `"User"`、`"ReadReceipt"` |
| エンティティのフィールド | camelCase | `"senderId"`、`"createdAt"` |
| `features` | camelCase | `"sendMessage"`、`"createGroup"` |
| `permissions` のキー | PascalCase | `"Admin"`、`"Guest"` |
| `permissions` の値 | camelCase | `"deleteMessage"` |
| `denied` のキー | PascalCase または camelCase | `"Reaction"`、`"editMessage"` |
| `denied` の値 | 自由形式の文字列 | `"Text-based only"` |
| `keywords` | 小文字 | `"chat"`、`"realtime"` |

---

## 5. バリデーションチェックリスト

ロックを提出する前に、以下のすべてを確認する：

| # | 確認事項 | 合格？ |
|---|-------|-------|
| 1 | `name`、`version`、`description`、`author` が存在する | |
| 2 | 少なくとも1つのプロダクト境界フィールドが存在する | |
| 3 | すべてのエンティティ名が PascalCase である | |
| 4 | すべての機能名が camelCase（動詞 + 名詞）である | |
| 5 | すべてのアクター名が PascalCase である | |
| 6 | すべての配列がアルファベット順にソートされている（stories を除く） | |
| 7 | すべてのオブジェクトキーがアルファベット順にソートされている | |
| 8 | ストーリーがアクター名または "System" で始まる | |
| 9 | ストーリーが正確な PascalCase 名でエンティティを参照している | |
| 10 | `permissions` のキーが `actors` と一致する | |
| 11 | `denied` の項目が `entities` や `features` に存在しない | |
| 12 | 実装エンティティが含まれていない（cache、temp、queue → stories） | |
| 13 | ストーリーに実装の詳細が含まれていない（Redis、WebSocket、フレームワーク名なし） | |
| 14 | エンティティフィールドは名前のみで、型が含まれていない | |

---

## 6. よくある間違い

### 1. 実装エンティティの混入
```
NG:  "entities": ["CachedQuote", "Message", "User"]
OK: "entities": ["Message", "User"]
      "stories": ["User views StockQuote with data refreshing in real-time"]
```

### 2. ストーリーに実装の詳細を含める
```
NG:  "System uses Redis to cache stock quotes"
OK: "User views StockQuote with data refreshing in real-time"
```

### 3. ストーリーではなく仕様の言語を使う
```
NG:  "Users should be able to send messages"
OK: "Member sends Message to Conversation"
```

### 4. 過剰な denied
```
NG:  "denied": ["exportData", "multiLanguage", "mobileApp", "paymentBilling"]
      （これらはまだ構築されていないだけで、意図的な除外ではない）

OK: "denied": {
        "executeTrade": "Market intelligence only — no brokerage liability",
        "voiceCall": "Text-based communication only"
      }
      （理由付きの意図的なプロダクトの意思決定）
```

### 5. 不十分な denied
```
NG:  "denied": []   または denied を完全に省略する
      （AIにガードレールがなく、何でも追加できてしまう）

OK: プロダクトが何になってはならないかを真剣に考える。
      denied はAI時代の開発において最も重要なフィールドである。
```

### 6. 粒度が細かすぎる機能
```
NG:  "features": ["validateEmail", "hashPassword", "generateToken", "refreshToken"]
OK: "features": ["login", "register", "resetPassword"]
```

### 7. システムストーリーの漏れ
バックグラウンドジョブ、cron、自動化プロセスがあれば、それらにもストーリーが必要：
```
"System fetches NewsArticle from multiple sources on schedule"
"System detects SupplyChainDisruption from NewsArticle"
```

### 8. ソートされていない配列
```
NG:  "actors": ["Member", "Admin", "Guest"]
OK: "actors": ["Admin", "Guest", "Member"]
```

---

## 7. 完全な例

以下を持つチャットアプリケーションのコードベースがある場合：
- Prisma スキーマ：User、Conversation、Message、ReadReceipt モデル
- API ルート：/messages、/conversations、/groups
- 認証：RBAC による3つのロール（admin、member、guest）
- 音声/ビデオ機能なし
- メッセージ編集なし

生成されるロック：

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

*このガイドは [Product Lock Specification](https://spec.productlock.org) の一部です。*
