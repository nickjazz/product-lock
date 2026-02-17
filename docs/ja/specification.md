---
layout: default
title: Specification
nav_order: 2
permalink: /ja/specification
lang: ja
lang_label: 日本語
page_id: specification
---

# Product Lock 仕様書

### バージョン 0.1.0 — 2026年2月

> 人間とAIのためのプロダクト境界仕様。
>
> product.lock.json は、ソフトウェアプロダクトが**何であるか**、そして**何でないか**を記述する。
> プロダクトがどのように構築されるかは記述しない。

この文書における「MUST」「MUST NOT」「SHOULD」「SHOULD NOT」「MAY」というキーワードは、[RFC 2119](https://tools.ietf.org/html/rfc2119) に従って解釈されるものとする。

---

## 目次

1. [はじめに](#1-はじめに)
2. [クイックスタート](#2-クイックスタート)
3. [設計原則](#3-設計原則)
4. [ファイル形式](#4-ファイル形式)
5. [メタデータフィールド](#5-メタデータフィールド)
6. [プロダクト境界フィールド](#6-プロダクト境界フィールド)
7. [段階的厳密性](#7-段階的厳密性)
8. [規約](#8-規約)
9. [バリデーションルール](#9-バリデーションルール)
10. [ロール](#10-ロール)
11. [ライフサイクル](#11-ライフサイクル)
12. [FAQ](#12-faq)
13. [参考文献](#13-参考文献)
14. [将来の検討事項](#14-将来の検討事項)

---

## 1. はじめに

### 1.1 課題

AIは今やソフトウェアプロダクトのコードの大部分を書くことができる。しかし、境界のないコード生成は新たな問題を生む：**スコープドリフト**である。AIは誰も求めていない機能を追加し、存在すべきでないエンティティを作成し、プロダクトラインを越える機能を構築する。

従来の開発では、プロダクトは人間の時間と労力によって自然に制約されていた。AIによってその制約は消える。プロダクト境界は明示的に定義されなければならない。

### 1.2 解決策

`product.lock.json` は、ソフトウェアプロダクトが何を含み、何を含んではならないかを定義する、機械可読かつ人間がレビュー可能なファイルである。3者間の契約として機能する：

- **人間** — 境界を承認する
- **AI Worker** — 境界の範囲内で構築する
- **AI Reviewer** — コードを境界に照らして検証する

### 1.3 スコープ

Product Lock は**プロダクト層**のみを定義する：

| スコープ内 | スコープ外 |
|----------|-------------|
| プロダクトの利用者（actors） | ルート、エンドポイント |
| 保存するデータ（entities） | 依存関係、パッケージ |
| 実行できること（features） | フレームワークの選択 |
| 相互作用の方法（stories） | ファイル構成 |
| 誰が何をできるか（permissions） | デプロイ設定 |
| 持ってはならないもの（denied） | コードスタイル、パターン |

### 1.4 フォーマット戦略

**JSON が信頼の源。Markdown はレンダリングビュー。**

`product.lock.json` が正規フォーマットである — 決定的で、スキーマ検証可能で、書き方は一つだけ。

人間に承認のために提示される際、ロックは可読性のために Markdown としてレンダリングされる。人間は生の JSON を読む必要がない。

```
AI Worker が JSON を書く  →  AI Reviewer が JSON を読む  →  人間が Markdown を読む
```

---

## 2. クイックスタート

### 2.1 最小限の例

最もシンプルな有効なロック：

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

4つのメタデータフィールドと、少なくとも1つのプロダクト境界フィールド。それだけである。

### 2.2 一般的な例

すべてのプロダクト境界フィールドを含むより完全なロック：

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

### 2.3 仕様としてのロック

ロックはポータブルである。誰かが `chat-system` のロックを共有し、それをAIに渡す：

> 「このプロダクトを Python で FastAPI と SQLAlchemy を使って構築して。」

同じロック、異なる技術スタック。プロダクト境界は同一。

---

## 3. 設計原則

### 3.1 プロダクトであって、コードではない

ロックはプロダクト境界を記述し、技術的な実装は記述しない。ルートも、依存関係も、フレームワークの選択もない。異なる技術スタックで構築された2つのプロダクトが同じロックを共有できる。

### 3.2 段階的厳密性

指定すればするほど、より厳密に検証される。フィールドを省略すれば、AIは自由に決定する。`entities: ["User"]` と書けば、Reviewer は User が存在することだけを確認する。`entities: { "User": ["id", "name"] }` と書けば、Reviewer は正確なフィールドを確認する。ロックは指定されたものだけを、それ以上でもそれ以下でもなく検証する。

### 3.3 一目で把握可能

人間はコードを読まなくても、ロックの構造を一覧して承認・拒否の判断を下せるべき（SHOULD）である。シンプルなプロダクトなら数秒、複雑なプロダクトなら数分。いずれにしても、ロックの確認はコードレビューより桁違いに速い。

### 3.4 言語非依存

同じロックが TypeScript、Python、Go、Java、その他どの言語でも機能する。ロックはプロダクトレベルの命名規約（PascalCase のエンティティ、camelCase の機能）を使用し、AI Worker がコード生成時にターゲット言語の規約に適応する。

### 3.5 仕様としてのロック

ロックは共有可能なプロダクト仕様である。誰かのロックを受け取ることは、そのプロダクト要件を受け取ることと同等である。他の仕様と同様に、バージョン管理、差分比較、共有が可能である。

### 3.6 境界とは除外するもの

AI時代の開発において、プロダクトが何をすべきでないかを定義することは、何をするかを定義することよりも重要である。AIは機能、エンティティ、振る舞いを際限なく追加できる。プロダクトスコープを維持する唯一の方法は、除外事項を明示的に宣言することである。`denied` フィールドがそのガードレールとなる。

---

## 4. ファイル形式

- ファイル名は `product.lock.json` でなければならない（MUST）
- ファイルは有効な JSON（[RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259)）でなければならない（MUST）
- ファイルは UTF-8 エンコーディングを使用しなければならない（MUST）
- ファイルはプロジェクトのルートディレクトリに配置されるべきである（SHOULD）
- 人間が読めるレンダリングビューとして、オプションで `product.lock.md` を生成してもよい（MAY）

---

## 5. メタデータフィールド

| フィールド | 型 | 必須 | 説明 |
|-------|------|----------|-------------|
| `$schema` | string | いいえ | エディタのバリデーションとオートコンプリートのためのスキーマ URL |
| `name` | string | **はい** | kebab-case のプロダクト識別子 |
| `version` | string | **はい** | プロダクトバージョン、[semver](https://semver.org/) に従うべき（SHOULD） |
| `description` | string | **はい** | 一行のプロダクト説明 |
| `author` | string | **はい** | このロックを承認した人 |
| `license` | string | いいえ | ライセンス識別子（例：`"MIT"`、`"UNLICENSED"`） |
| `keywords` | string[] | いいえ | ロック共有時の発見性を高めるタグ |
| `private` | boolean | いいえ | `true` の場合、ロックは公開共有を意図していない |

これらのフィールドは設計上、[package.json](https://docs.npmjs.com/cli/v10/configuring-npm/package-json) の規約に従っている。

**例：**
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

## 6. プロダクト境界フィールド

すべてのプロダクト境界フィールドは**任意**である。フィールドを省略すると、AIは自由に決定し、Reviewer はそれをチェックしない。

プロダクト境界フィールドは、存在する場合、以下の順序で記述しなければならない（MUST）：

```
actors → entities → features → stories → permissions → denied
```

これは概念的な流れに従う：誰が使うか → どんなデータか → どんな操作か → どう相互作用するか → 誰が何をできるか → 何が除外されるか。

---

### 6.1 `actors`

**目的：** プロダクトを使用する人々を定義する。これらはユーザーロールであり、コードエンティティではない。

**形式：** `string[]`

**例：**
```json
"actors": ["Admin", "Guest", "Member"]
```

**ルール：**
- エントリは PascalCase でなければならない（MUST）
- 配列はアルファベット順にソートされなければならない（MUST）
- 省略された場合、AIがすべてのユーザーロールを決定する

**反例：**
```json
"actors": ["admin", "UserService", "AuthMiddleware"]
```
`admin` は PascalCase ではない。`UserService` と `AuthMiddleware` はユーザーロールではなくコードエンティティである。

---

### 6.2 `entities`

**目的：** データモデルを定義する — プロダクトが保存するもの。

**ルースモード** — 名前の配列。Reviewer はエンティティの存在のみを確認する。

```json
"entities": ["Conversation", "Message", "ReadReceipt", "User"]
```

**ストリクトモード** — フィールドリスト付きのオブジェクト。Reviewer はエンティティとそのフィールドの両方を確認する。

```json
"entities": {
  "Conversation": [],
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "ReadReceipt": [],
  "User": ["avatar", "email", "id", "name"]
}
```

**ルール：**
- エンティティ名は PascalCase でなければならない（MUST）
- フィールド名は camelCase でなければならない（MUST）
- 空配列 `[]` はエンティティが存在しなければならない（MUST）が、フィールドはロックされないことを意味する
- 非空配列はこれらのフィールドが存在しなければならない（MUST）ことを意味する — それ以上でもそれ以下でもない
- フィールドは名前のみで、型はない（型は実装の詳細である）
- 配列とキーはアルファベット順にソートされなければならない（MUST）

**反例：**
```json
"entities": {
  "CachedQuote": ["symbol", "price", "cachedAt"],
  "User": ["id", "name"]
}
```
`CachedQuote` は実装の詳細（キャッシュテーブル）であり、プロダクトエンティティではない。それが満たすプロダクト要件は、代わりにストーリーとして記述されるべきである（SHOULD）：
```json
"stories": ["User views StockQuote with data refreshing in real-time"]
```

**重要なルール：** エンティティは「プロダクトは何を保存するか？」に答えるものである。インフラ的なもの（キャッシュ、キュー、一時テーブル、マイグレーションログ）であれば、エンティティには含めない。それが満たすプロダクト上のニーズをストーリーとして記述する。

---

### 6.3 `features`

**目的：** プロダクトの機能を定義する — プロダクトが実行できること。

**形式：** `string[]`

**例：**
```json
"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

**ルール：**
- 機能は camelCase でなければならない（MUST）
- 機能は動詞 + 名詞の形式に従うべきである（SHOULD）（`sendMessage`、`createGroup`、`viewDashboard`）
- 配列はアルファベット順にソートされなければならない（MUST）
- サブ機能なし — フラットに保つ
- 各機能はコードベース内で独立して識別可能でなければならない（MUST）

**粒度のガイド：**

| 粗すぎる | 適切 | 細かすぎる |
|-----------|------|-----------|
| `manageMessages` | `sendMessage`、`deleteMessage` | `validateMessageLength` |
| `handleAuth` | `login`、`register` | `hashPassword` |

機能は**プロダクトマネージャー**がリストするものであり、**開発者**がリストするものではない。

**反例：**
```json
"features": ["hashPassword", "validateEmail", "generateJwt"]
```
これらは実装の詳細である。プロダクトの機能は `login`、`register`、`resetPassword` である。

---

### 6.4 `stories`

**目的：** インタラクションフロー、体験の期待、システムの振る舞いを定義する。ストーリーはプロダクトのナラティブであり、アクター、エンティティ、機能を意味のあるシーケンスに結びつける。

**形式：** `string[]`

**例：**
```json
"stories": [
  "Member sends Message to Conversation",
  "Member creates Conversation as Group and invites other Members",
  "When Member reads Message, system creates ReadReceipt",
  "Admin deletes any Message from Conversation",
  "Guest views Conversation but cannot send Message"
]
```

**3種類のストーリー：**

| 種類 | 目的 | 例 |
|------|---------|---------|
| 機能フロー | プロダクト利用時に何が起こるか | `"Member sends Message to Conversation"` |
| 体験の期待 | ユーザーが知覚するもの（非機能要件） | `"User views Dashboard with data loading instantly"` |
| システムの振る舞い | システムが自律的に行うこと | `"System fetches NewsArticle from multiple sources on schedule"` |

**ルール：**
- 各ストーリーは一文でなければならない（MUST）
- ストーリーはアクター名（PascalCase、`actors` と一致）または `"System"` で始めなければならない（MUST）
- ストーリーは正確な PascalCase 名でエンティティを参照しなければならない（MUST）
- ストーリーは現在形、能動態を使用しなければならない（MUST）
- ストーリーは何が起こるか（WHAT）を記述し、どのように実装されるか（HOW）を記述してはならない（MUST）
- ストーリーはアルファベット順ではなく、**ナラティブの流れ**の順序で並べる（これがアルファベット順ソートルールの唯一の例外である）

**反例：**
```
"Messages are sent via WebSocket"          ← 実装の詳細（WebSocket）
"Use Redis to cache stock quotes"          ← 実装の詳細（Redis）
"The user should be able to send messages" ← 仕様の言語であり、ストーリーではない
"member sends message to conversation"     ← ケーシングが間違っている
```

**実装からストーリーへの変換：** 何かがエンティティのように見えるがインフラである場合、それが満たすプロダクト要件をストーリーとして記述する：

| 実装の詳細 | ストーリー |
|----------------------|-------|
| CachedQuote テーブル | `"User views StockQuote with data refreshing in real-time"` |
| レートリミッターミドルウェア | `"System rate limits API requests per IP address"` |
| エンベディングベクトルストア | `"System searches knowledge base by semantic similarity"` |

---

### 6.5 `permissions`

**目的：** アクセス制御を定義する — 誰が何をできるか。

**ルースモード** — モデル名のみ。AIが詳細を自由に実装する。

```json
"permissions": "rbac"
```

有効なモデル名：`"rbac"`、`"abac"`、`"acl"`。

**ストリクトモード** — ロール-パーミッションマトリクス。Reviewer は各ロールが正確にこれらの権限を持つことを確認する。

```json
"permissions": {
  "Admin": ["createGroup", "deleteMessage", "listConversations", "removeMember", "sendMessage"],
  "Guest": ["listConversations"],
  "Member": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
}
```

**ルール：**
- オブジェクトの場合：キーは `actors` のエントリと一致しなければならない（MUST）
- オブジェクトの場合：値は `features`（またはフィーチャーライクなアクション）のサブセットでなければならない（MUST）
- キーと値の配列はアルファベット順にソートされなければならない（MUST）
- システム専用の機能（例：`fetchNews`、`enrichData`）は permissions に含めない — ユーザーが呼び出すものではない
- 省略された場合、AIがすべてのアクセス制御を決定する

**反例：**
```json
"permissions": {
  "admin": ["send-message"]
}
```
`admin` は PascalCase ではない（actors と一致しなければならない（MUST））。`send-message` は camelCase ではない（features と一致しなければならない（MUST））。

---

### 6.6 `denied`

**目的：** 明示的な除外を定義する — プロダクトが持ってはならないもの。

**これはAI時代の開発において最も重要なフィールドである。**

従来の開発では、プロダクトは人間の時間と労力によって自然に制約されていた。AIには自然な制限がない — AIは機能、エンティティ、振る舞いを際限なく追加できる。プロダクトスコープを維持する唯一の方法は、存在してはならないものを明示的に宣言することである。他のフィールドはプロダクトが何であるかを定義する。`denied` はプロダクトが何でないかを定義する。`denied` がなければ、AIにはガードレールがない。

**ルースモード** — フラットリスト。

```json
"denied": ["Reaction", "editMessage", "voiceCall"]
```

**理由付き**（推奨） — 説明付きの項目。理由は Reviewer が各除外の重みを理解するのに役立ち、人間が情報に基づいた判断を下すのに役立つ。

```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "Reaction": "Intentionally excluded to keep messaging simple",
  "voiceCall": "Text-based communication only"
}
```

**ルール：**
- Reviewer は denied の項目がコードベースに存在しないことを確認しなければならない（MUST）
- 見つかった場合、それは違反である
- denied のキーは PascalCase（エンティティ的な場合）または camelCase（機能的な場合）でなければならない（MUST）
- キーはアルファベット順にソートされなければならない（MUST）
- 理由は情報提供用であり、検証の動作を変更しない

**denied に含めるべきもの：**

| カテゴリ | 例 | 理由 |
|----------|---------|--------|
| プロダクトスコープ | `"voiceCall"` | 「テキストベースのコミュニケーションのみ」 |
| 責任範囲の境界 | `"executeTrade"` | 「証券仲介の責任を負わない」 |
| スコープのガードレール | `"Reaction"` | 「メッセージングをシンプルに保つ」 |

**denied に含めるべきでないもの：**
- まだ構築されていない機能（それはバックログであり、プロダクトの意思決定ではない）
- 実装アプローチ（`"don't use Redis"` — それは HOW であり、WHAT ではない）
- 次のバージョンで追加される予定の一時的な制限

**反例：**
```json
"denied": ["exportData", "multiLanguage", "mobileApp", "paymentBilling"]
```
これらはまだ構築されていない機能であり、意図的なプロダクトの除外ではない。問い：「AIが誤ってこれを追加した場合、プロダクト違反になるか？」いいえなら、denied に含めるべきではない。

---

## 7. 段階的厳密性

ロックは指定されたものだけを正確に検証する。それ以上のものはない。

| 記述内容 | Reviewer が確認すること |
|----------------|------------------------|
| フィールドが完全に省略されている | 何もしない — AIが自由に決定する |
| `"entities": ["User"]` | User エンティティが存在する |
| `"entities": { "User": [] }` | User エンティティが存在する（上記と同じ） |
| `"entities": { "User": ["id", "name"] }` | User が正確に `id` と `name` を持つ、それ以上でもそれ以下でもない |
| `"features": ["sendMessage"]` | sendMessage の振る舞いがコードベースに存在する |
| `"stories": [...]` | 記述された各フローがコードベースに存在する |
| `"permissions": "rbac"` | 何らかの形の RBAC が存在する |
| `"permissions": { "Admin": ["delete"] }` | Admin が正確にこれらの権限を持つ |
| `"denied": ["Reaction"]` | Reaction がどこにも存在しない |
| `"denied": { "Reaction": "reason" }` | 同じチェック、理由は意思決定を文書化する |

**ルール：ロックされていないものは自由。** AI はロックされていないエンティティにフィールドを追加したり、リストにない機能を追加したり、言及されていないエンティティを作成したりしてよい（MAY）。Reviewer はロックに記載されているもののみを確認する。

**例外：`denied`。** denied の項目は、他のフィールドに関係なく、常に不在が確認される。これはAIがプロダクトスコープを超えることを防ぐガードレールである。

---

## 8. 規約

これらの規約は、誰が — あるいはどのAIが — 生成しても、すべてのロックが同じ見た目になることを保証する。

### 8.1 命名規則

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

ロックは**プロダクト名**を定義し、コード名ではない。コード生成時、AI Worker は命名をターゲット言語の規約に適応させる（例：ロックの `sendMessage` は Python では `send_message` になる）。

### 8.2 並び順

すべての配列とオブジェクトキーはアルファベット順にソートされなければならない（MUST）。

```json
"actors": ["Admin", "Guest", "Member"]

"entities": {
  "Conversation": [],
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "User": ["avatar", "email", "id", "name"]
}

"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

**例外：** `stories` はアルファベット順ではなく、ナラティブの流れの順序で並べる。シーケンスがストーリーを語る。

### 8.3 キー順序

トップレベルのキーは以下の順序で記述しなければならない（MUST）：

```
$schema → name → version → description → author → license → keywords → private
→ actors → entities → features → stories → permissions → denied
```

メタデータが先、次にプロダクト境界フィールドが概念的な順序で続く。

### 8.4 ストーリーの形式

- ストーリーごとに一文
- アクター名または `"System"` で始める
- 正確な PascalCase 名でエンティティを参照する
- 現在形、能動態
- 3つの種類：**機能フロー**、**体験の期待**、**システムの振る舞い**

---

## 9. バリデーションルール

有効な `product.lock.json` は以下のすべてを満たさなければならない（MUST）：

| # | ルール | 重大度 |
|---|------|----------|
| 1 | `name`、`version`、`description`、`author` が存在しなければならない（MUST） | エラー |
| 2 | 少なくとも1つのプロダクト境界フィールドが存在しなければならない（MUST） | エラー |
| 3 | `entities` は `string[]` または `Record<string, string[]>` でなければならない（MUST） | エラー |
| 4 | `features` は `string[]` でなければならない（MUST） | エラー |
| 5 | `actors` は `string[]` でなければならない（MUST） | エラー |
| 6 | `stories` は `string[]` でなければならない（MUST） | エラー |
| 7 | `permissions` は `string` または `Record<string, string[]>` でなければならない（MUST） | エラー |
| 8 | `denied` は `string[]` または `Record<string, string>` でなければならない（MUST） | エラー |
| 9 | `permissions` のキーは `actors` のエントリと一致しなければならない（MUST）（両方が定義されている場合） | エラー |
| 10 | `denied` の項目は `entities` または `features` に存在してはならない（MUST NOT） | エラー |
| 11 | すべての配列とオブジェクトキーはアルファベット順にソートされなければならない（MUST）（`stories` を除く） | エラー |
| 12 | `name` は kebab-case、`entities` は PascalCase、`features` は camelCase、`actors` は PascalCase でなければならない（MUST） | エラー |
| 13 | `stories` で参照されるエンティティ/機能/アクター名は、それぞれのフィールドに存在すべきである（SHOULD） | 警告 |
| 14 | `version` は semver 形式に従うべきである（SHOULD） | 警告 |

---

## 10. ロール

### 10.1 人間（ボス）

- ロック（Markdown としてレンダリング）を確認する
- 承認または拒否する
- `denied` リストを修正する
- コードを読まない、実装をレビューしない

### 10.2 AI Worker

1. コードを書く
2. コードから `product.lock.json` を生成する
3. エビデンスを準備する（内部用、人間には見せない）
4. ロック + エビデンスを Reviewer に提出する

### 10.3 AI Reviewer

1. Worker からロックを受け取る
2. コードベースを独立して検査する（Worker のエビデンスを信頼しない）
3. 確認事項：
   - すべてのロックされたエンティティが存在する（ストリクトモードの場合は正確なフィールドも）
   - すべてのロックされた機能が識別可能な振る舞いとして存在する
   - すべてのストーリーのフローがコード内に存在する
   - すべての権限が正しく適用されている
   - すべての denied 項目が存在しない
   - ロックに記載されるべき未宣言のエンティティや機能が存在しないか
4. 報告：**合格**または**違反リスト**

---

## 11. ライフサイクル

```
1. 人間が意図を記述する（自然言語）
        ↓
2. AI Worker がプロダクトを構築する
        ↓
3. AI Worker がコードから product.lock.json を生成する
        ↓
4. 人間がロック（Markdown ビュー）を確認する → 承認 / 修正 / 拒否
        ↓
5. ロックが凍結される
        ↓
6. AI Worker がロック境界の範囲内で開発を継続する
        ↓
7. AI Reviewer がコードをロックに照らして定期的にチェックする
        ↓
8. 違反が報告される → 人間がアクションを決定する
        ↓
9. 次のイテレーション → バージョンバンプ → ステップ4から繰り返し
```

---

## 12. FAQ

### なぜ YAML ではなく JSON なのか？

JSON は決定的である — 同じデータを表現する方法がちょうど一つしかない。YAML には複数の同等な表現がある（インデントスタイル、フローとブロック、クォーティングルール）。AIが生成しAIが検証する仕様にとって、決定性は人間の書きやすさよりも重要である。人間はいずれにしても Markdown ビューを読む。

### ルースモードとストリクトモードはいつ使い分けるべきか？

全体的な構造が重要で詳細が重要でない場合は**ルースモード**を使用する。特定のフィールドがプロダクトにとって重要な場合（例：金融データモデル、コンプライアンスに敏感なエンティティ）は**ストリクトモード**を使用する。まずはルースで始め、必要に応じてストリクトにする。

### `denied` に入れるものと「まだ構築していないもの」の違いは？

問い：「AIが誤ってこれを追加した場合、プロダクト違反になるか？」はいなら、`denied` に含める。いいえなら、それはバックログに過ぎない。

### 同じロックから異なるコードベースを生成できるか？

はい。ロックは WHAT を定義し、HOW は定義しない。同じロックに異なる技術スタックを組み合わせると、同じプロダクト境界を実装する構造的に異なるコードが生成される。HOW も指定するには、ロックと `product.plan.md` を組み合わせる。

### マイクロサービスの場合は？サービスごとに1つのロック？

ロックは**サービスごと**ではなく、**プロダクトごと**に1つ。マイクロサービスが合わせて1つのプロダクトを構成するなら、1つのロックを共有する。独立したプロダクトであれば、それぞれが独自のロックを持つ。

### カスタムフィールドでロックを拡張できるか？

v0.1 ではできない。将来のバージョンでは `x-` プレフィックスパターンによる拡張フィールドをサポートする可能性がある。現時点では、定義されたフィールドのみを使用する。

---

## 13. 参考文献

- [RFC 8259 — The JavaScript Object Notation (JSON) Data Interchange Format](https://datatracker.ietf.org/doc/html/rfc8259)
- [RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [package.json — npm documentation](https://docs.npmjs.com/cli/v10/configuring-npm/package-json)

---

## 14. 将来の検討事項

以下の機能は v0.1 には含まれないが、将来のバージョンで追加される可能性がある：

- `extends` — 別のロックを継承し、特定のフィールドをオーバーライドする
- `milestones` — フェーズタグ付け（MVP / v2 / 将来）
- `contributors` — 複数の著者
- `repository` — ソースコードの場所
- `changelog` — ロックバージョンの差分履歴
- `x-` プレフィックスによる拡張フィールド

---

*Product Lock Specification は [MIT License](https://opensource.org/licenses/MIT) の下でライセンスされています。*
*仕様のソース：[productlock.org](https://spec.productlock.org)*
