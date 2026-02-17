<p align="center">
  <a href="https://productlock.org">
    <img src="logo.png" alt="Product Lock" width="60">
  </a>
</p>

<h1 align="center">product.lock.json</h1>

<p align="center">
  チームを管理するようにAIを管理する。
</p>

<p align="center">
  <a href="https://spec.productlock.org"><img src="https://img.shields.io/badge/Website-spec.productlock.org-blue?style=flat-square" alt="Website"></a>
  <a href="https://github.com/anthropics/product-lock/blob/main/LICENSE"><img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="License"></a>
  <a href="product-lock-spec.md"><img src="https://img.shields.io/badge/Spec-v0.1.0-orange?style=flat-square" alt="Spec Version"></a>
</p>

<p align="center">
  <a href="README.md">English</a> ·
  <a href="README-zh-TW.md">繁體中文</a> ·
  <strong>日本語</strong> ·
  <a href="README-de.md">Deutsch</a>
</p>

---

**あなたのマネジメント経験 — スコープ管理、レビュープロセス、意思決定の追跡 — をAIが従えるプロトコルに。**

product.lock.jsonはソフトウェアプロダクトの境界を定義する — それが**何であるか**、そして**何でないか**。言語、フレームワーク、アーキテクチャに関係なく、境界は同じ。

```
AI Worker がコードを書く  →  lock を生成  →  人間が承認  →  AI Reviewer が検証
```

---

## なぜ必要？

AIは今やコードの大部分を書ける。問題は：**いつ止めるべきか分からないこと。**

チャットアプリを頼む。AIは親切にも絵文字リアクション、音声通話、ファイル共有を追加する。どれも頼んでいない。でもAIは知らない。あなたが「ノー」と言わなかったから。

従来の開発には自然な境界がある — 人間の時間と労力。AIにはそれがない。だから境界を明示的に書き下す必要がある。

**product.lock.jsonがその境界。**

---

## どんな見た目？

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

6つのフィールド。6つの問い：

| フィールド | 問い |
|-----------|------|
| `actors` | 誰が使う？ |
| `entities` | どんなデータを保存する？ |
| `features` | 何ができる？ |
| `stories` | どう連携する？ |
| `permissions` | 誰が何をできる？ |
| `denied` | **何を持ってはいけない？** |

最後が最も重要。AI時代では、**作らないもの**を定義することが、作るものを定義するより重要。

---

## コアアイデア

### WHATを定義し、HOWは定義しない

lockにルート、フレームワーク、依存関係、ファイル構造は含まない。

同じlock、異なるスタック。AIに「Python + FastAPIで作って」でも「TypeScript + Next.jsで作って」でも渡せる。プロダクト境界は同一。

### 多く指定すれば、厳しく適用

```json
"entities": ["User"]                              // Userの存在だけ確認
"entities": { "User": [] }                        // 同上 — 存在だけ確認
"entities": { "User": ["id", "name", "email"] }   // 正確なフィールド、過不足なし
```

フィールドを省略 = AIが自由に決定。ロックするのは気にする部分だけ。

### 3つの役割

| 役割 | 仕事 |
|------|------|
| **人間** | lockを見て、承認または却下、deniedリストを管理 |
| **AI Worker** | コードを書き、lockを生成 |
| **AI Reviewer** | コードがlockに準拠しているか独立検証。Workerを信頼しない |

人間はコードを読まない。人間が読むのはlock（Markdownにレンダリング）。数時間ではなく、数分で済む。

### `denied`がガードレール

```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "voiceCall": "Text-based communication only"
}
```

AIが`denied`にあるものを誤って追加した場合、Reviewerが違反としてフラグを立てる。これが最も重要な防衛線。

テスト：「AIが勝手にこれを追加したら、プロダクト違反になるか？」はい → deniedに入れる。いいえ → それは単なるバックログ、入れない。

---

## ドキュメント

| ドキュメント | 対象 | 目的 |
|-------------|------|------|
| [仕様書](product-lock-spec.md) | 全員 | 完全仕様（英語） |
| [仕様書（中国語）](product-lock-spec-zh.md) | 全員 | 完全仕様（中国語） |
| [Generator ガイド](product-lock-generator-guide.md) | AI Worker | コードベースからlockを生成する方法 |
| [Reviewer ガイド](product-lock-reviewer-guide.md) | AI Reviewer | コードがlockに準拠しているか検証する方法 |
| [Plan 仕様](product-plan-spec.md) | AI Worker | product.plan.md形式（HOW） |
| [Scoring](product-lock-scoring.md) | 全員 | lockからプロダクト複雑度を定量化 |
| [Worklog](product-lock-worklog.md) | AI + 人間 | セッション横断で変更・決定・スコープを追跡 |

---

## クイックスタート

### 最もシンプルなlock

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

4つのメタデータフィールド + 少なくとも1つのプロダクトフィールド。以上。

### AIに生成させる

[Generator ガイド](product-lock-generator-guide.md)をAIに渡し、コードベースを指定：

> 「このガイドを読んで、このコードベースを分析して、product.lock.jsonを生成して。」

### AIにレビューさせる

[Reviewer ガイド](product-lock-reviewer-guide.md)を*別の*AIに渡し、lockとコードベースも一緒に：

> 「このガイドを読んで、このproduct.lock.jsonに対してこのコードベースをレビューして。」

2つのAI。互いを信頼しない。1つが生成し、1つが検証する。人間が最終判断。

---

## 命名規則

| フィールド | 形式 | 例 |
|-----------|------|-----|
| `name` | kebab-case | `"chat-system"` |
| `actors` | PascalCase | `"Admin"`, `"Member"` |
| `entities` | PascalCase | `"User"`, `"ReadReceipt"` |
| entity fields | camelCase | `"senderId"`, `"createdAt"` |
| `features` | camelCase | `"sendMessage"`, `"createGroup"` |
| `denied` | PascalCaseまたはcamelCase | `"Reaction"`, `"editMessage"` |

全配列はアルファベット順。唯一の例外：`stories`はナラティブフローに従う。

---

## FAQ

**なぜJSONでYAMLじゃない？**
JSONは決定論的 — 同じデータ、1つの書き方。AIが生成し、AIが検証する場合、決定論は利便性に勝る。人間はどうせレンダリングされたMarkdownを読む。

**マイクロサービスごとに1つのlock？**
プロダクトごとに1つのlock。マイクロサービスはHOWであり、WHATではない。

**`denied`を省略できる？**
できる。でもそれはAIに「何でも好きに追加していい」と言っていること。本当にいい？

**異なる言語で同じlockを実装できる？**
できる。それがポイント。

---

## ライセンス

MIT

---

<p align="center">
  <a href="https://productlock.org"><strong>productlock.org</strong></a>
</p>
