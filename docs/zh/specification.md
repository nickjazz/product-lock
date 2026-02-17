---
layout: default
title: 規格書
nav_order: 2
permalink: /zh/specification
lang: zh
lang_label: 繁體中文
page_id: specification
---

# Product Lock 規格書

### 版本 0.1.0 — 2026 年 2 月

> 給人和 AI 共用的產品邊界規格。
>
> product.lock.json 描述軟體產品「是什麼」和「不是什麼」。
> 不描述產品怎麼做。

本文件中的關鍵詞「必須（MUST）」、「不得（MUST NOT）」、「應當（SHOULD）」、「不應（SHOULD NOT）」和「可以（MAY）」按照 [RFC 2119](https://tools.ietf.org/html/rfc2119) 的定義來解讀。

---

## 目錄

1. [簡介](#1-簡介)
2. [快速開始](#2-快速開始)
3. [設計原則](#3-設計原則)
4. [檔案格式](#4-檔案格式)
5. [後設資料欄位](#5-後設資料欄位)
6. [產品邊界欄位](#6-產品邊界欄位)
7. [漸進式嚴格](#7-漸進式嚴格)
8. [寫法規範](#8-寫法規範)
9. [驗證規則](#9-驗證規則)
10. [角色](#10-角色)
11. [生命週期](#11-生命週期)
12. [常見問題](#12-常見問題)
13. [參考文獻](#13-參考文獻)
14. [未來考慮](#14-未來考慮)

---

## 1. 簡介

### 1.1 問題

AI 現在能寫軟體產品的大部分程式碼。但沒有邊界的程式碼生成帶來了新問題：**範圍漂移**。AI 會自行新增沒人要的功能、建立不該存在的 entity、構建跨越產品線的能力。

在傳統開發中，產品天然受限於人力和時間。有了 AI，這個限制消失了。產品邊界必須被明確定義。

### 1.2 解法

`product.lock.json` 是一個機器可讀、人可審閱的檔案，定義軟體產品包含什麼以及不能包含什麼。它是三方之間的契約：

- **人** — 批准邊界
- **AI Worker** — 在邊界內構建
- **AI Reviewer** — 驗證程式碼是否符合邊界

### 1.3 範圍

Product Lock 只定義**產品層**：

| 在範圍內 | 不在範圍內 |
|----------|-----------|
| 誰在用產品（actors） | 路由、端點 |
| 存什麼資料（entities） | 依賴套件 |
| 能做什麼（features） | 框架選擇 |
| 怎麼互動（stories） | 檔案結構 |
| 誰能做什麼（permissions） | 部署設定 |
| 什麼不能有（denied） | 程式碼風格、設計模式 |

### 1.4 格式策略

**JSON 是唯一真相來源。Markdown 是渲染給人看的。**

`product.lock.json` 是正式格式 — 確定性、可驗證 schema、只有一種寫法。

當呈現給人類批准時，lock 渲染為 Markdown 以提高可讀性。人永遠不需要讀原始 JSON。

```
AI Worker 寫 JSON  →  AI Reviewer 讀 JSON  →  人讀 Markdown
```

---

## 2. 快速開始

### 2.1 最簡範例

最簡單的合法 lock：

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

四個後設資料欄位加上至少一個產品邊界欄位。就這樣。

### 2.2 典型範例

包含所有產品邊界欄位的完整 lock：

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

### 2.3 Lock 即規格

Lock 是可攜帶的。有人分享了他的 `chat-system` lock，你丟給 AI：

> 「用 Python + FastAPI + SQLAlchemy 做一個這樣的產品。」

同一份 lock，不同技術棧。產品邊界完全一樣。

---

## 3. 設計原則

### 3.1 產品，不是程式碼

Lock 描述產品邊界，不描述技術實作。沒有路由、沒有依賴、沒有框架選擇。用不同技術棧構建的兩個產品可以共用同一份 lock。

### 3.2 漸進式嚴格

越細就越嚴，沒指定就寬鬆。省略一個欄位，AI 自由決定。寫 `entities: ["User"]`，Reviewer 只檢查 User 存在。寫 `entities: { "User": ["id", "name"] }`，Reviewer 檢查精確欄位。Lock 只強制你指定的部分，不多。

### 3.3 可掃描

人應當能快速掃描 lock 結構並做出批准/否決的決定，不需要讀程式碼。簡單產品幾秒鐘；複雜產品幾分鐘。無論如何，掃描 lock 比 review 程式碼快一個數量級。

### 3.4 語言無關

同一份 lock 適用於 TypeScript、Python、Go、Java 或任何其他語言。Lock 使用產品級命名規範（PascalCase entities、camelCase features），AI Worker 在生成程式碼時轉換為目標語言的命名慣例。

### 3.5 Lock 即規格

Lock 是可分享的產品規格。收到別人的 lock 等於收到他們的產品需求。可以版本控制、可以 diff、可以分享，就像其他規格文件一樣。

### 3.6 邊界靠排除來守

在 AI 時代，定義產品「不做什麼」比定義「做什麼」更重要。AI 可以無限增加功能、entity 和行為。唯一守住產品邊界的方法就是明確宣告排除項。`denied` 欄位就是那道護欄。

---

## 4. 檔案格式

- 檔案必須命名為 `product.lock.json`
- 檔案必須是合法 JSON（[RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259)）
- 檔案必須使用 UTF-8 編碼
- 檔案應當放在專案根目錄
- 可以額外生成 `product.lock.md` 作為人可讀的渲染視圖

---

## 5. 後設資料欄位

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `$schema` | string | 否 | Schema URL，啟用編輯器驗證與自動補全 |
| `name` | string | **是** | 產品識別名稱，kebab-case |
| `version` | string | **是** | 產品版本，應當遵循 [semver](https://semver.org/) |
| `description` | string | **是** | 一行產品描述 |
| `author` | string | **是** | 批准這份 lock 的人 |
| `license` | string | 否 | 授權標識（例如 `"MIT"`、`"UNLICENSED"`） |
| `keywords` | string[] | 否 | 標籤，用於分享時的搜尋與發現 |
| `private` | boolean | 否 | 設為 `true` 表示不打算公開分享 |

這些欄位刻意沿用 [package.json](https://docs.npmjs.com/cli/v10/configuring-npm/package-json) 慣例。

**範例：**
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

## 6. 產品邊界欄位

所有產品邊界欄位都是**可選的**。省略的欄位代表 AI 自由決定，Reviewer 不檢查。

產品邊界欄位出現時必須按以下順序排列：

```
actors → entities → features → stories → permissions → denied
```

這遵循概念流程：誰在用 → 存什麼資料 → 能做什麼 → 怎麼互動 → 誰能做什麼 → 什麼被排除。

---

### 6.1 `actors`

**用途：** 定義使用產品的人。這是使用者角色，不是程式碼 entity。

**格式：** `string[]`

**範例：**
```json
"actors": ["Admin", "Guest", "Member"]
```

**規則：**
- 項目必須是 PascalCase
- 陣列必須按字母排序
- 省略時 AI 自行決定所有使用者角色

**反面範例：**
```json
"actors": ["admin", "UserService", "AuthMiddleware"]
```
`admin` 不是 PascalCase。`UserService` 和 `AuthMiddleware` 是程式碼 entity，不是使用者角色。

---

### 6.2 `entities`

**用途：** 定義資料模型 — 產品儲存什麼。

**寬鬆模式** — 名稱陣列。Reviewer 只檢查 entity 是否存在。

```json
"entities": ["Conversation", "Message", "ReadReceipt", "User"]
```

**嚴格模式** — 物件加欄位列表。Reviewer 同時檢查 entity 和欄位。

```json
"entities": {
  "Conversation": [],
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "ReadReceipt": [],
  "User": ["avatar", "email", "id", "name"]
}
```

**規則：**
- Entity 名稱必須是 PascalCase
- 欄位名稱必須是 camelCase
- 空陣列 `[]` 代表 entity 必須存在，但欄位不鎖定
- 非空陣列代表這些欄位必須存在 — 不能多，不能少
- 欄位只有名稱，沒有型別（型別是實作細節）
- 陣列和 key 必須按字母排序

**反面範例：**
```json
"entities": {
  "CachedQuote": ["symbol", "price", "cachedAt"],
  "User": ["id", "name"]
}
```
`CachedQuote` 是實作細節（快取表），不是產品 entity。它服務的產品需求應當改寫成 story：
```json
"stories": ["User views StockQuote with data refreshing in real-time"]
```

**關鍵規則：** Entity 回答「產品存什麼？」如果某個東西是基礎設施（快取、佇列、暫存表、遷移紀錄），它不屬於 entities。把它服務的產品需求寫成 story。

---

### 6.3 `features`

**用途：** 定義產品能力 — 產品能做什麼。

**格式：** `string[]`

**範例：**
```json
"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

**規則：**
- Features 必須是 camelCase
- Features 應當遵循動詞 + 名詞格式（`sendMessage`、`createGroup`、`viewDashboard`）
- 陣列必須按字母排序
- 沒有子功能 — 保持扁平
- 每個 feature 必須在程式碼庫中可獨立識別

**粒度指南：**

| 太粗 | 剛好 | 太細 |
|------|------|------|
| `manageMessages` | `sendMessage`、`deleteMessage` | `validateMessageLength` |
| `handleAuth` | `login`、`register` | `hashPassword` |

Features 是**產品經理**會列的東西，不是**開發者**會列的東西。

**反面範例：**
```json
"features": ["hashPassword", "validateEmail", "generateJwt"]
```
這些是實作細節。產品功能是 `login`、`register`、`resetPassword`。

---

### 6.4 `stories`

**用途：** 定義互動流程、體驗預期和系統行為。Stories 是產品的敘事 — 把 actors、entities 和 features 串聯成有意義的流程。

**格式：** `string[]`

**範例：**
```json
"stories": [
  "Member sends Message to Conversation",
  "Member creates Conversation as Group and invites other Members",
  "When Member reads Message, system creates ReadReceipt",
  "Admin deletes any Message from Conversation",
  "Guest views Conversation but cannot send Message"
]
```

**三種 story：**

| 類型 | 用途 | 範例 |
|------|------|------|
| 功能流程 | 使用者操作產品時發生什麼 | `"Member sends Message to Conversation"` |
| 體驗預期 | 使用者感知到什麼（非功能性需求） | `"User views Dashboard with data loading instantly"` |
| 系統行為 | 系統自主做什麼 | `"System fetches NewsArticle from multiple sources on schedule"` |

**規則：**
- 每條 story 必須是一句話
- Stories 必須以 actor 名稱（PascalCase，對應 `actors`）或 `"System"` 開頭
- Stories 必須以精確的 PascalCase 名稱引用 entities
- Stories 必須使用現在式主動語態
- Stories 必須描述「發生什麼」，不描述「怎麼實作」
- Stories 按**敘事順序**排列，不按字母排序（這是唯一例外）

**反面範例：**
```
"Messages are sent via WebSocket"          ← 實作細節（WebSocket）
"Use Redis to cache stock quotes"          ← 實作細節（Redis）
"The user should be able to send messages" ← spec 語言，不是 story
"member sends message to conversation"     ← 大小寫不對
```

**實作細節轉 story：** 如果某個東西感覺像 entity 但其實是基礎設施，把它服務的產品需求寫成 story：

| 實作細節 | Story |
|----------|-------|
| CachedQuote 快取表 | `"User views StockQuote with data refreshing in real-time"` |
| Rate limiter 中介層 | `"System rate limits API requests per IP address"` |
| Embedding 向量儲存 | `"System searches knowledge base by semantic similarity"` |

---

### 6.5 `permissions`

**用途：** 定義存取控制 — 誰能做什麼。

**寬鬆模式** — 只寫模型名稱。AI 自行決定實作細節。

```json
"permissions": "rbac"
```

合法模型名稱：`"rbac"`、`"abac"`、`"acl"`。

**嚴格模式** — 角色-權限矩陣。Reviewer 檢查每個角色剛好有這些權限。

```json
"permissions": {
  "Admin": ["createGroup", "deleteMessage", "listConversations", "removeMember", "sendMessage"],
  "Guest": ["listConversations"],
  "Member": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
}
```

**規則：**
- 物件模式時：key 必須對應 `actors` 項目
- 物件模式時：value 必須是 `features` 的子集（或類似功能的動作）
- Key 和 value 陣列必須按字母排序
- 系統專屬功能（例如 `fetchNews`、`enrichData`）不出現在 permissions — 它們不是使用者可觸發的
- 省略時 AI 自行決定所有存取控制

**反面範例：**
```json
"permissions": {
  "admin": ["send-message"]
}
```
`admin` 不是 PascalCase（必須對應 actors）。`send-message` 不是 camelCase（必須對應 features）。

---

### 6.6 `denied`

**用途：** 定義明確排除 — 產品不能有什麼。

**這是 AI 時代最重要的欄位。**

在傳統開發中，產品天然受限於人力和時間。有了 AI，這個限制不存在 — AI 可以無限增加功能、entity 和行為。唯一守住產品邊界的方法就是明確宣告什麼不能存在。其他欄位定義產品「是什麼」。`denied` 定義產品「不是什麼」。沒有 `denied`，AI 就沒有護欄。

**寬鬆模式** — 扁平列表。

```json
"denied": ["Reaction", "editMessage", "voiceCall"]
```

**附理由模式**（推薦）— 項目加說明。理由幫助 Reviewer 理解每個排除的重要性，幫助人做出更明智的決定。

```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "Reaction": "Intentionally excluded to keep messaging simple",
  "voiceCall": "Text-based communication only"
}
```

**規則：**
- Reviewer 必須檢查 denied 項目在程式碼庫中不存在
- 如果發現 → 違規
- Denied key 必須是 PascalCase（entity 類）或 camelCase（feature 類）
- Key 必須按字母排序
- 理由是資訊性的 — 不影響驗證行為

**什麼該放進 denied：**

| 類別 | 範例 | 理由 |
|------|------|------|
| 產品範圍 | `"voiceCall"` | "Text-based communication only" |
| 責任邊界 | `"executeTrade"` | "No brokerage liability" |
| 範圍護欄 | `"Reaction"` | "Keep messaging simple" |

**什麼不該放進 denied：**
- 只是還沒做的功能（那是 backlog，不是產品決策）
- 實作方式（`"不要用 Redis"` — 那是 HOW，不是 WHAT）
- 暫時的限制，下個版本就會加

**反面範例：**
```json
"denied": ["exportData", "multiLanguage", "mobileApp", "paymentBilling"]
```
這些只是還沒做的功能，不是刻意的產品排除。判斷方法：「如果 AI 不小心加了這個，算不算產品違規？」如果不算，它不屬於 denied。

---

## 7. 漸進式嚴格

Lock 只強制你指定的部分，不多。

| 你怎麼寫 | Reviewer 檢查什麼 |
|----------|------------------|
| 欄位完全省略 | 什麼都不檢查 — AI 自由決定 |
| `"entities": ["User"]` | User entity 存在 |
| `"entities": { "User": [] }` | User entity 存在（同上） |
| `"entities": { "User": ["id", "name"] }` | User 剛好有 `id` 和 `name`，不能多不能少 |
| `"features": ["sendMessage"]` | sendMessage 行為存在於程式碼庫 |
| `"stories": [...]` | 每條描述的流程存在於程式碼庫 |
| `"permissions": "rbac"` | 某種形式的 RBAC 存在 |
| `"permissions": { "Admin": ["delete"] }` | Admin 剛好有這些權限 |
| `"denied": ["Reaction"]` | Reaction 在任何地方都不存在 |
| `"denied": { "Reaction": "理由" }` | 同樣檢查，理由記錄產品決策 |

**規則：沒鎖的就是自由的。** AI 可以在未鎖定的 entity 上加欄位、加不在列表中的 feature、建立未提及的 entity。Reviewer 只檢查 lock 裡有的東西。

**例外：`denied`。** 被拒絕的項目無論其他欄位怎麼寫，都會被檢查是否不存在。這是防止 AI 超出產品範圍的護欄。

---

## 8. 寫法規範

這些規範確保每份 lock 不管是誰（或哪個 AI）生成的，看起來都一樣。

### 8.1 命名

| 欄位 | 規則 | 範例 |
|------|------|------|
| `name` | kebab-case | `"chat-system"` |
| `actors` | PascalCase | `"Admin"`、`"Member"` |
| `entities` | PascalCase | `"User"`、`"ReadReceipt"` |
| entity 欄位 | camelCase | `"senderId"`、`"createdAt"` |
| `features` | camelCase | `"sendMessage"`、`"createGroup"` |
| `permissions` key | PascalCase | `"Admin"`、`"Guest"` |
| `permissions` value | camelCase | `"deleteMessage"` |
| `denied` key | PascalCase 或 camelCase | `"Reaction"`、`"editMessage"` |
| `denied` value | 自由格式字串 | `"Text-based only"` |
| `keywords` | 全小寫 | `"chat"`、`"realtime"` |

Lock 定義的是**產品名稱**，不是程式碼名稱。生成程式碼時，AI Worker 會轉換為目標語言的命名慣例（例如 lock 裡的 `sendMessage` 在 Python 中變成 `send_message`）。

### 8.2 排序

所有陣列和物件的 key 必須按字母排序。

```json
"actors": ["Admin", "Guest", "Member"]

"entities": {
  "Conversation": [],
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "User": ["avatar", "email", "id", "name"]
}

"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

**例外：** `stories` 按敘事順序排列，不按字母排序。順序本身在說故事。

### 8.3 Key 順序

頂層 key 必須按以下順序排列：

```
$schema → name → version → description → author → license → keywords → private
→ actors → entities → features → stories → permissions → denied
```

後設資料在前，然後產品邊界欄位按概念順序。

### 8.4 Stories 格式

- 每條 story 一句話
- 以 actor 名稱或 `"System"` 開頭
- 以精確的 PascalCase 名稱引用 entities
- 現在式主動語態
- 三種：**功能流程**、**體驗預期**、**系統行為**

---

## 9. 驗證規則

一份合法的 `product.lock.json` 必須滿足以下所有規則：

| # | 規則 | 嚴重度 |
|---|------|--------|
| 1 | `name`、`version`、`description`、`author` 必須存在 | 錯誤 |
| 2 | 至少一個產品邊界欄位必須存在 | 錯誤 |
| 3 | `entities` 必須是 `string[]` 或 `Record<string, string[]>` | 錯誤 |
| 4 | `features` 必須是 `string[]` | 錯誤 |
| 5 | `actors` 必須是 `string[]` | 錯誤 |
| 6 | `stories` 必須是 `string[]` | 錯誤 |
| 7 | `permissions` 必須是 `string` 或 `Record<string, string[]>` | 錯誤 |
| 8 | `denied` 必須是 `string[]` 或 `Record<string, string>` | 錯誤 |
| 9 | `permissions` 的 key 必須對應 `actors` 項目（兩者都有定義時） | 錯誤 |
| 10 | `denied` 項目不得同時出現在 `entities` 或 `features` 中 | 錯誤 |
| 11 | 所有陣列和物件 key 必須按字母排序（`stories` 除外） | 錯誤 |
| 12 | `name` 必須 kebab-case；`entities` 必須 PascalCase；`features` 必須 camelCase；`actors` 必須 PascalCase | 錯誤 |
| 13 | `stories` 中引用的 entity/feature/actor 名稱應當存在於對應欄位 | 警告 |
| 14 | `version` 應當遵循 semver 格式 | 警告 |

---

## 10. 角色

### 10.1 人（老闆）

- 掃描 lock（渲染為 Markdown）
- 批准或否決
- 修改 `denied` 清單
- 不讀程式碼，不做技術審查

### 10.2 AI Worker（員工）

1. 寫程式碼
2. 從程式碼生成 `product.lock.json`
3. 準備證據（內部用，不給人看）
4. 將 lock + 證據提交給 Reviewer

### 10.3 AI Reviewer（主管）

1. 收到 Worker 的 lock
2. 獨立檢查程式碼庫（不信任 Worker 的證據）
3. 檢查：
   - 每個鎖定的 entity 存在（嚴格模式下欄位也要對）
   - 每個鎖定的 feature 以可識別的行為存在
   - 每條 story 的流程在程式碼中存在
   - 每個鎖定的 permission 正確配置
   - 每個 denied 項目不存在
   - 沒有未申報的 entity/feature 存在於程式碼中但不在 lock 裡
4. 回報：**通過**或**違規清單**

---

## 11. 生命週期

```
1. 人描述意圖（自然語言）
        ↓
2. AI Worker 構建產品
        ↓
3. AI Worker 從程式碼生成 product.lock.json
        ↓
4. 人掃描 lock（Markdown 視圖）→ 批准 / 修改 / 否決
        ↓
5. Lock 凍結
        ↓
6. AI Worker 在 lock 邊界內繼續開發
        ↓
7. AI Reviewer 定期檢查程式碼是否符合 lock
        ↓
8. 違規回報 → 人決定處理方式
        ↓
9. 下一個迭代 → 版本升級 → 回到步驟 4
```

---

## 12. 常見問題

### 為什麼用 JSON 而不是 YAML？

JSON 是確定性的 — 同樣的資料只有一種表示方式。YAML 有多種等效寫法（縮排風格、流式 vs 區塊、引號規則）。對於 AI 生成 + AI 驗證的規格，確定性比人的書寫便利更重要。人讀的是 Markdown 視圖。

### 什麼時候該用寬鬆模式？什麼時候用嚴格模式？

當大致結構重要但細節不重要時用**寬鬆模式**。當特定欄位對產品至關重要時用**嚴格模式**（例如金融資料模型、合規敏感的 entity）。從寬鬆開始，需要時再收緊。

### `denied` 和「還沒做」有什麼差別？

判斷方法：「如果 AI 不小心加了這個，算不算產品違規？」如果算，放進 `denied`。如果不算，那只是 backlog。

### 同一份 lock 能產出不同的程式碼庫嗎？

能。Lock 定義 WHAT，不定義 HOW。同一份 lock 搭配不同技術棧會產出結構不同的程式碼，但實作相同的產品邊界。搭配 `product.plan.md` 可以同時指定 HOW。

### 微服務怎麼辦？一個服務一份 lock？

一個**產品**一份 lock，不是一個服務一份。如果你的微服務共同構成一個產品，它們共用一份 lock。如果它們是獨立產品，各自有自己的 lock。

### 可以在 lock 中加自訂欄位嗎？

v0.1 不支援。未來版本可能支援 `x-` 前綴的擴充欄位。目前只使用已定義的欄位。

---

## 13. 參考文獻

- [RFC 8259 — The JavaScript Object Notation (JSON) Data Interchange Format](https://datatracker.ietf.org/doc/html/rfc8259)
- [RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [package.json — npm documentation](https://docs.npmjs.com/cli/v10/configuring-npm/package-json)

---

## 14. 未來考慮

以下功能不在 v0.1 範圍內，但可能在未來版本加入：

- `extends` — 繼承另一份 lock，覆蓋特定欄位
- `milestones` — 階段標記（MVP / v2 / future）
- `contributors` — 多位作者
- `repository` — 原始碼位置
- `changelog` — lock 版本差異歷史
- `x-` 前綴的擴充欄位

---

*Product Lock Specification 採用 [MIT License](https://opensource.org/licenses/MIT) 授權。*
*規格來源：[productlock.org](https://spec.productlock.org)*
