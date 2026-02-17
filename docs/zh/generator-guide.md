---
layout: default
title: 生成器指南
nav_order: 4
permalink: /zh/generator-guide
lang: zh
lang_label: 繁體中文
page_id: generator-guide
---

# Product Lock 生成器指南

### 版本 0.1.0 — 2026 年 2 月

> 你是一個 AI Worker。你的工作是分析程式碼庫，並生成一份能準確描述產品邊界的 `product.lock.json`。
>
> 你負責生成 lock 檔案。由人類審核批准。再由 AI Reviewer 驗證程式碼是否符合規範。

完整規格請參閱 [Product Lock 規格書](https://spec.productlock.org)。

---

## 目錄

1. [什麼是 product.lock.json？](#1-什麼是-productlockjson)
2. [生成流程](#2-生成流程)
3. [欄位參考](#3-欄位參考)
4. [命名慣例](#4-命名慣例)
5. [驗證清單](#5-驗證清單)
6. [常見錯誤](#6-常見錯誤)
7. [完整範例](#7-完整範例)

---

## 1. 什麼是 product.lock.json？

`product.lock.json` 定義了**一個軟體產品是什麼、不是什麼** — 這是在產品層級，而非程式碼層級。

它回答六個問題：

| 欄位 | 問題 | 格式 |
|------|------|------|
| `actors` | 誰使用這個產品？ | PascalCase 名詞 |
| `entities` | 它儲存哪些資料？ | PascalCase 名詞 |
| `features` | 它能做什麼？ | camelCase 動詞 |
| `stories` | actors、entities 和 features 如何互動？ | 自然語言句子 |
| `permissions` | 誰能做什麼？ | Actor 對 feature 的矩陣 |
| `denied` | 它絕對不能有什麼？ | 明確排除項目 |

**Lock 描述的是產品邊界，而非實作細節。** 不包含路由、相依套件、框架選擇或檔案結構。

---

## 2. 生成流程

### 步驟 1：閱讀程式碼庫

掃描程式碼庫以了解：

- 資料庫 schema / 模型 / 型別 → entities
- API 路由 / handler / controller → features
- 認證 / 角色定義 → actors、permissions
- UI 頁面 / 視圖 → features、stories
- 中介軟體 / 背景任務 / 排程工作 → stories
- 測試 → 幫助確認有哪些 features 存在

### 步驟 2：提取中繼資料

```json
{
  "name": "<kebab-case 產品名稱>",
  "version": "<語意化版本號，新專案從 0.1.0 開始>",
  "description": "<一行產品描述>",
  "author": "<誰將審核批准此 lock>"
}
```

- `name`：從專案資料夾名稱或 package.json 衍生，一律使用 kebab-case
- `description`：用一句話描述產品的用途，從使用者角度出發
- `author`：詢問人類，或使用 git committer 名稱

### 步驟 3：識別 actors

Actors 是**使用產品的人**，而非程式碼實體。

尋找：
- 認證/RBAC 程式碼中的角色定義
- 不同的權限層級
- UI 中不同類型的使用者

```json
"actors": ["Admin", "Member", "Guest"]
```

規則：
- PascalCase
- 按字母順序排列
- 不是程式碼實體（不包含 `UserService`、`AuthMiddleware`）
- 如果只有一種使用者類型，使用 `["User"]`

### 步驟 4：提取 entities

Entities 是**產品儲存的內容** — 資料模型。

尋找：
- 資料庫表格 / Prisma 模型 / TypeORM 實體
- Mongoose schema / SQLAlchemy 模型
- 含有 `id` 欄位的 TypeScript 介面
- GraphQL 型別

**決策：寬鬆模式或嚴格模式？**

在以下情況使用**寬鬆模式**：
- 快速概覽即可
- 欄位是標準且可預測的
- 人類不需要鎖定特定欄位

```json
"entities": ["Conversation", "Message", "User"]
```

在以下情況使用**嚴格模式**：
- 欄位對產品至關重要（例如金融資料）
- 人類需要驗證確切的資料模型
- 產品有複雜或不尋常的欄位需求

```json
"entities": {
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "User": ["avatar", "email", "id", "name"]
}
```

規則：
- Entity 名稱使用 PascalCase
- 欄位名稱使用 camelCase
- 按字母順序排列（鍵和欄位陣列皆是）
- `[]` = entity 存在，但欄位未鎖定
- 欄位只有名稱 — **不包含型別**（型別是實作細節）

**關鍵判斷：是 entity 還是不是 entity？**

| 列為 entity | 不列為 entity |
|------------|--------------|
| User、Product、Order — 領域資料 | CachedQuote、TempSession — 實作細節 |
| Message、Conversation — 面向使用者 | MigrationLog、AuditTrail — 維運用途 |
| StockMetadata — 產品確實儲存此資料 | RedisKey、QueueJob — 基礎設施 |

如果某項看起來像 entity 但實際上是實作細節（快取表、暫存、任務佇列），**不要將其列為 entity**。取而代之，將其所服務的產品需求捕捉為一個 **story**（見步驟 6）。

### 步驟 5：提取 features

Features 是**原子級的產品能力** — 產品能做的事。

尋找：
- API 端點（每個主要端點 = 一個 feature）
- UI 操作（按鈕、表單、工作流程）
- 背景任務能力
- 系統整合

```json
"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

規則：
- camelCase
- 動詞 + 名詞格式：`sendMessage`、`createGroup`、`viewDashboard`
- 按字母順序排列
- 扁平列表 — 無子功能、無巢狀結構
- 每個 feature 必須能在程式碼庫中獨立識別

**粒度指南：**

| 太粗 | 適中 | 太細 |
|------|------|------|
| `manageMessages` | `sendMessage`、`deleteMessage` | `validateMessageLength` |
| `handleAuth` | `login`、`register`、`resetPassword` | `hashPassword` |
| `stockAnalysis` | `analyzeStock`、`viewStockChart` | `calculateMovingAverage` |

Features 應該對應**產品經理**會列出的事項，而非**開發者**會列出的事項。

### 步驟 6：撰寫 stories

Stories 描述 **actors、entities 和 features 如何互動**。它們是產品的敘事。

三種類型：

**功能流程** — 使用者使用產品時會發生什麼：
```
"Member sends Message to Conversation"
"Admin deletes any Message from Conversation"
"When Member reads Message, system creates ReadReceipt"
```

**體驗期望** — 使用者感知到的內容（非功能性需求）：
```
"User views StockQuote with data refreshing in real-time"
"User views Dashboard with market data loading instantly"
```

**系統行為** — 系統自主執行的動作：
```
"System fetches NewsArticle from multiple sources on schedule"
"System rate limits API requests per IP address"
"System calibrates model with Bayesian posteriors and detects concept drift"
```

規則：
- 每個 story 一句話
- 以 actor 名稱開頭（PascalCase，必須在 `actors` 中）或 `"System"`
- 以精確的 PascalCase 名稱引用 entities
- 現在式，主動語態
- 描述**做了什麼**，而非**如何實作**
- 按**敘事流程**排序（不按字母排序 — 這是唯一的例外）

**Entity 轉 story 規則：**

如果你發現服務於產品目的的實作細節，將它們轉換為 stories：

| 發現的實作項目 | 應撰寫的 story |
|--------------|---------------|
| `CachedQuote` 表格 | `"User views StockQuote with data refreshing in real-time"` |
| `CachedHistory` 表格 | `"User views StockChart with historical data loading instantly"` |
| 速率限制中介軟體 | `"System rate limits API requests per IP address"` |
| 嵌入向量儲存 | `"System searches knowledge base by semantic similarity"` |
| 背景排程任務 | `"System fetches NewsArticle from multiple sources on schedule"` |

### 步驟 7：定義 permissions

Permissions 連結 **actors 與 features** — 誰能做什麼。

**選擇正確的模式：**

如果產品沒有認證或只有基本認證：
- 完全省略此欄位（AI 自由決定）

如果產品使用已知的認證模型：
```json
"permissions": "rbac"
```

如果需要指定精確的角色與功能對應：
```json
"permissions": {
  "Admin": ["createGroup", "deleteMessage", "sendMessage"],
  "Guest": ["listConversations"],
  "Member": ["createGroup", "sendMessage"]
}
```

規則：
- 鍵使用 PascalCase，必須與 `actors` 一致
- 值是 camelCase 陣列，應為 `features` 的子集
- 鍵和值陣列均按字母順序排列
- 系統專屬的 features（如 `fetchNews`、`enrichNewsWithAi`）不放入 permissions — 它們不是使用者可呼叫的

### 步驟 8：決定 denied 項目

**這是最重要的步驟。** Denied 項目是防止 AI 超出產品範圍的護欄。

在傳統開發中，產品受限於人類的時間。但有了 AI，就沒有天然的限制 — 除非明確告知，否則 AI 會不斷新增 features、entities 和行為。`denied` 就是你劃定那條界線的方式。

這不是「產品目前還沒有的所有東西」。而是針對**刻意的產品決策**，其中缺席本身就具有重要意義。

**決策框架：**

問：「如果 AI 意外加入了這個功能，是否構成產品違規？」
- 是 → 列入 denied
- 不是，只是尚未開發 → 不要列入

**寬鬆模式** — 扁平列表：
```json
"denied": ["executeTrade", "Reaction", "voiceCall"]
```

**附帶理由**（建議使用） — 每個項目附帶說明：
```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "Reaction": "Intentionally excluded to keep messaging simple",
  "voiceCall": "Text-based communication only"
}
```

理由有助於 Reviewer 理解為何排除某項目，也幫助人類做出更好的批准/拒絕決策。

**三類 denied 項目：**

| 類別 | 範例 | 理由 |
|------|------|------|
| 產品範圍 | `"voiceCall"` | "Text-based communication only" |
| 責任邊界 | `"executeTrade"` | "No brokerage liability" |
| 範圍護欄 | `"Reaction"` | "Keep messaging simple" |

### 步驟 9：組裝與驗證

按照以下鍵的順序組裝完整的 lock：

```
$schema → name → version → description → author → license → keywords → private
→ actors → entities → features → stories → permissions → denied
```

執行驗證清單（見[第 5 節](#5-驗證清單)）。

### 步驟 10：生成 Markdown 檢視

在 JSON lock 之後，生成一份 `product.lock.md` 供人類審閱：

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

## 3. 欄位參考

### 中繼資料欄位

| 欄位 | 必填 | 格式 | 說明 |
|------|------|------|------|
| `$schema` | 否 | string | 用於驗證的 Schema URL |
| `name` | **是** | kebab-case string | 產品識別碼 |
| `version` | **是** | semver string | 產品版本 |
| `description` | **是** | string | 一行產品描述 |
| `author` | **是** | string | 誰審核批准此 lock |
| `license` | 否 | string | 授權條款識別碼 |
| `keywords` | 否 | lowercase string[] | 可搜尋標籤 |
| `private` | 否 | boolean | 不公開分享 |

### 產品邊界欄位

全部為選填。省略 = AI 自由決定。

| 欄位 | 類型 | 說明 |
|------|------|------|
| `actors` | `string[]` | 使用者角色 |
| `entities` | `string[]` 或 `Record<string, string[]>` | 資料模型 |
| `features` | `string[]` | 產品能力 |
| `stories` | `string[]` | 互動流程與行為 |
| `permissions` | `string` 或 `Record<string, string[]>` | 存取控制 |
| `denied` | `string[]` 或 `Record<string, string>` | 明確排除項目 |

---

## 4. 命名慣例

| 欄位 | 慣例 | 範例 |
|------|------|------|
| `name` | kebab-case | `"chat-system"` |
| `actors` | PascalCase | `"Admin"`、`"Member"` |
| `entities` | PascalCase | `"User"`、`"ReadReceipt"` |
| entity 欄位 | camelCase | `"senderId"`、`"createdAt"` |
| `features` | camelCase | `"sendMessage"`、`"createGroup"` |
| `permissions` 鍵 | PascalCase | `"Admin"`、`"Guest"` |
| `permissions` 值 | camelCase | `"deleteMessage"` |
| `denied` 鍵 | PascalCase 或 camelCase | `"Reaction"`、`"editMessage"` |
| `denied` 值 | 自由格式字串 | `"Text-based only"` |
| `keywords` | lowercase | `"chat"`、`"realtime"` |

---

## 5. 驗證清單

在提交 lock 之前，請驗證以下所有項目：

| # | 檢查項目 | 通過？ |
|---|---------|--------|
| 1 | `name`、`version`、`description`、`author` 存在 | |
| 2 | 至少有一個產品邊界欄位存在 | |
| 3 | 所有 entity 名稱為 PascalCase | |
| 4 | 所有 feature 名稱為 camelCase（動詞 + 名詞） | |
| 5 | 所有 actor 名稱為 PascalCase | |
| 6 | 所有陣列按字母順序排列（stories 除外） | |
| 7 | 所有物件的鍵按字母順序排列 | |
| 8 | Stories 以 actor 名稱或 "System" 開頭 | |
| 9 | Stories 以精確的 PascalCase 名稱引用 entities | |
| 10 | `permissions` 的鍵與 `actors` 一致 | |
| 11 | `denied` 項目未出現在 `entities` 或 `features` 中 | |
| 12 | 沒有實作用途的 entities（快取、暫存、佇列 → stories） | |
| 13 | Stories 中沒有實作細節（沒有 Redis、WebSocket、框架名稱） | |
| 14 | Entity 欄位只有名稱，沒有型別 | |

---

## 6. 常見錯誤

### 1. 包含實作用途的 entities
```
錯誤：  "entities": ["CachedQuote", "Message", "User"]
正確：  "entities": ["Message", "User"]
        "stories": ["User views StockQuote with data refreshing in real-time"]
```

### 2. Stories 中包含實作細節
```
錯誤：  "System uses Redis to cache stock quotes"
正確：  "User views StockQuote with data refreshing in real-time"
```

### 3. 使用規格語言而非 stories
```
錯誤：  "Users should be able to send messages"
正確：  "Member sends Message to Conversation"
```

### 4. 過度限制（Over-denying）
```
錯誤：  "denied": ["exportData", "multiLanguage", "mobileApp", "paymentBilling"]
        （這些只是尚未開發，並非刻意排除）

正確：  "denied": {
          "executeTrade": "Market intelligence only — no brokerage liability",
          "voiceCall": "Text-based communication only"
        }
        （附帶理由的刻意產品決策）
```

### 5. 限制不足（Under-denying）
```
錯誤：  "denied": []   或完全省略 denied
        （AI 沒有護欄，可以任意新增功能）

正確：  認真思考產品絕對不能變成什麼。
        denied 是 AI 時代開發中最重要的欄位。
```

### 6. Features 粒度太細
```
錯誤：  "features": ["validateEmail", "hashPassword", "generateToken", "refreshToken"]
正確：  "features": ["login", "register", "resetPassword"]
```

### 7. 遺漏系統 stories
如果有背景任務、排程工作或自動化流程，它們也需要 stories：
```
"System fetches NewsArticle from multiple sources on schedule"
"System detects SupplyChainDisruption from NewsArticle"
```

### 8. 陣列未排序
```
錯誤：  "actors": ["Member", "Admin", "Guest"]
正確：  "actors": ["Admin", "Guest", "Member"]
```

---

## 7. 完整範例

假設有一個聊天應用程式的程式碼庫，其中包含：
- Prisma schema：User、Conversation、Message、ReadReceipt 模型
- API 路由：/messages、/conversations、/groups
- 認證：透過 RBAC 設定三個角色（admin、member、guest）
- 沒有語音/視訊功能
- 沒有訊息編輯功能

生成的 lock：

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

*本指南是 [Product Lock 規格書](https://spec.productlock.org) 的一部分。*
