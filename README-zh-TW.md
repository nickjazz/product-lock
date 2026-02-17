<p align="center">
  <a href="https://productlock.org">
    <img src="logo.png" alt="Product Lock" width="60">
  </a>
</p>

<h1 align="center">product.lock.json</h1>

<p align="center">
  像管理團隊一樣管理 AI。
</p>

<p align="center">
  <a href="https://productlock.org"><img src="https://img.shields.io/badge/Website-productlock.org-blue?style=flat-square" alt="Website"></a>
  <a href="https://github.com/anthropics/product-lock/blob/main/LICENSE"><img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="License"></a>
  <a href="product-lock-spec.md"><img src="https://img.shields.io/badge/Spec-v0.1.0-orange?style=flat-square" alt="Spec Version"></a>
</p>

<p align="center">
  <a href="README.md">English</a> ·
  <strong>繁體中文</strong> ·
  <a href="README-ja.md">日本語</a> ·
  <a href="README-de.md">Deutsch</a>
</p>

---

**你的管理經驗 — 範圍控制、審查流程、決策追蹤 — 編碼成 AI 能遵循的協議。**

product.lock.json 定義一個軟體產品的邊界 — 它**是什麼**和**不是什麼**。不管用什麼語言、框架或架構，邊界都一樣。

```
AI Worker 寫程式  →  產生 lock  →  人類審批  →  AI Reviewer 驗證
```

---

## 為什麼需要？

AI 現在能寫大部分的程式碼。問題是：**它不知道什麼時候該停。**

你要一個聊天 app。它貼心地幫你加了表情回應、語音通話、檔案分享。你根本沒要這些。但 AI 不知道，因為你從沒說過「不要」。

傳統開發有天然的邊界 — 人的時間和精力。AI 沒有。所以邊界必須明確寫下來。

**product.lock.json 就是那個邊界。**

---

## 長什麼樣？

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

6 個欄位。6 個問題：

| 欄位 | 問題 |
|------|------|
| `actors` | 誰在用？ |
| `entities` | 存什麼資料？ |
| `features` | 能做什麼？ |
| `stories` | 東西之間怎麼互動？ |
| `permissions` | 誰能做什麼？ |
| `denied` | **什麼是絕對不能有的？** |

最後一個最重要。在 AI 時代，定義你**不做什麼**比定義你做什麼更重要。

---

## 核心理念

### 定義 WHAT，不定義 HOW

Lock 裡沒有路由、框架、依賴、檔案結構。

同一個 lock，不同技術棧。給 AI 說「用 Python + FastAPI 來做」或「用 TypeScript + Next.js 來做」。產品邊界完全不變。

### 寫越多，鎖越緊

```json
"entities": ["User"]                              // 只確認 User 存在
"entities": { "User": [] }                        // 同上 — 只確認存在
"entities": { "User": ["id", "name", "email"] }   // 精確欄位，不多不少
```

省略一個欄位 = AI 自由決定。你只鎖你在意的。

### 三個角色

| 角色 | 職責 |
|------|------|
| **人類** | 看 lock、批准或否決、管理 denied 清單 |
| **AI Worker** | 寫程式、產生 lock |
| **AI Reviewer** | 獨立檢查程式碼是否符合 lock。不信任 Worker |

人類永遠不用看程式碼。人類看的是 lock（渲染成 Markdown）。幾分鐘搞定，不用幾小時。

### `denied` 是護欄

```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "voiceCall": "Text-based communication only"
}
```

如果 AI 不小心加了 `denied` 裡的東西，Reviewer 會標記為違規。這是最重要的防線。

判斷標準：「如果 AI 自己加了這個，算不算產品違規？」是 → 放進 denied。不是 → 那只是待辦，不用放。

---

## 文件

| 文件 | 對象 | 用途 |
|------|------|------|
| [規格書](product-lock-spec.md) | 所有人 | 完整規格（英文） |
| [規格書（中文）](product-lock-spec-zh.md) | 所有人 | 完整規格（中文） |
| [Generator 指南](product-lock-generator-guide.md) | AI Worker | 如何從 codebase 產生 lock |
| [Reviewer 指南](product-lock-reviewer-guide.md) | AI Reviewer | 如何驗證程式碼是否符合 lock |
| [Plan 規格](product-plan-spec.md) | AI Worker | product.plan.md 格式（HOW） |
| [Scoring](product-lock-scoring.md) | 所有人 | 從 lock 量化產品複雜度 |
| [Worklog](product-lock-worklog.md) | AI + 人類 | 追蹤跨 session 的變更、決策和範疇 |

---

## 快速開始

### 最簡單的 lock

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

4 個 metadata 欄位 + 至少 1 個產品欄位。就這樣。

### 讓 AI 產生一個

把 [Generator 指南](product-lock-generator-guide.md) 給 AI，指向你的 codebase：

> 「讀這份指南，然後分析這個 codebase，產生一個 product.lock.json。」

### 讓 AI 來審查

把 [Reviewer 指南](product-lock-reviewer-guide.md) 給*另一個* AI，連同 lock 和 codebase：

> 「讀這份指南，然後對照 product.lock.json 審查這個 codebase。」

兩個 AI。互不信任。一個產生，一個驗證。人類做最終決定。

---

## 命名規範

| 欄位 | 格式 | 範例 |
|------|------|------|
| `name` | kebab-case | `"chat-system"` |
| `actors` | PascalCase | `"Admin"`, `"Member"` |
| `entities` | PascalCase | `"User"`, `"ReadReceipt"` |
| entity fields | camelCase | `"senderId"`, `"createdAt"` |
| `features` | camelCase | `"sendMessage"`, `"createGroup"` |
| `denied` | PascalCase 或 camelCase | `"Reaction"`, `"editMessage"` |

所有陣列按字母排序。唯一例外：`stories` 按敘事流程排。

---

## 常見問題

**為什麼用 JSON 不用 YAML？**
JSON 是確定性的 — 同一份資料，只有一種寫法。當 AI 產生、AI 驗證時，確定性比方便更重要。人類反正看的是渲染過的 Markdown。

**一個微服務一個 lock？**
一個產品一個 lock。微服務是 HOW，不是 WHAT。

**可以不寫 `denied` 嗎？**
可以。但那等於跟 AI 說「隨便加什麼都行」。你確定嗎？

**不同語言可以實作同一個 lock 嗎？**
可以。這就是重點。

---

## 授權

MIT

---

<p align="center">
  <a href="https://productlock.org"><strong>productlock.org</strong></a>
</p>
