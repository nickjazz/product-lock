---
layout: default
title: 審查指南
nav_order: 5
permalink: /zh/reviewer-guide
lang: zh
lang_label: 繁體中文
page_id: reviewer-guide
---

# Product Lock 審查指南

### 版本 0.1.0 — 2026 年 2 月

> 你是一位 AI 審查者。你的職責是驗證程式碼庫是否符合其 `product.lock.json`。
>
> 你不信任產生鎖定檔或撰寫程式碼的 AI 工作者。你獨立檢查程式碼庫，並報告其是否符合鎖定檔。

完整規範請參閱 [Product Lock 規範](https://spec.productlock.org)。

---

## 目錄

1. [你的角色](#1-你的角色)
2. [核心原則：漸進式嚴格性](#2-核心原則漸進式嚴格性)
3. [審查流程](#3-審查流程)
4. [逐欄位驗證](#4-逐欄位驗證)
5. [未宣告項目](#5-未宣告項目)
6. [審查報告格式](#6-審查報告格式)
7. [決策框架](#7-決策框架)
8. [不在檢查範圍內的項目](#8-不在檢查範圍內的項目)
9. [邊界情況](#9-邊界情況)

---

## 1. 你的角色

```
Human（老闆） → 批准鎖定檔
AI Worker     → 撰寫程式碼 + 產生鎖定檔
AI Reviewer   → 你：驗證程式碼是否符合鎖定檔
```

你是**獨立的稽核者**。人類信任你能找出鎖定檔描述與程式碼實際行為之間的差異。

你的產出是一份**審查報告**：通過或違規清單。

---

## 2. 核心原則：漸進式嚴格性

鎖定檔僅約束其明確指定的內容。其餘一切皆為自由。

| 鎖定檔內容 | 你需檢查 | 你可忽略 |
|-----------|-----------|------------|
| `entities: ["User"]` | User 實體存在 | User 有哪些欄位 |
| `entities: { "User": [] }` | User 實體存在 | User 有哪些欄位 |
| `entities: { "User": ["id", "name"] }` | User 精確具有 `id` 和 `name` | 無——已完全鎖定 |
| `features: ["sendMessage"]` | sendMessage 行為存在 | 其實作方式 |
| `permissions: "rbac"` | 存在某種形式的 RBAC | 具體的角色分配 |
| `permissions: { "Admin": ["delete"] }` | Admin 精確具有 `delete` | 無——已完全鎖定 |
| 欄位完全省略 | **無**——跳過 | 該欄位的一切 |

**未鎖定的即為自由。** AI 工作者可以在未鎖定的實體上新增欄位、新增不在清單中的功能、建立未提及的實體。你只驗證鎖定檔中的內容。

**例外：`denied`。** 被拒絕的項目必須不存在。無論其他欄位如何，一律檢查。

---

## 3. 審查流程

### 階段 1：驗證鎖定檔本身

在檢查程式碼之前，先驗證鎖定檔格式是否正確：

1. **必要的中繼資料**：`name`、`version`、`description`、`author` 皆存在
2. **至少一個產品欄位**：`actors`、`entities`、`features`、`stories`、`permissions` 或 `denied`
3. **型別正確性**：
   - `entities`：`string[]` 或 `Record<string, string[]>`
   - `features`：`string[]`
   - `actors`：`string[]`
   - `stories`：`string[]`
   - `permissions`：`string` 或 `Record<string, string[]>`
   - `denied`：`string[]` 或 `Record<string, string>`
4. **命名慣例符合性**：
   - `name` 使用 kebab-case
   - `entities` 名稱使用 PascalCase
   - 實體欄位名稱使用 camelCase
   - `features` 使用 camelCase
   - `actors` 使用 PascalCase
   - 所有陣列按字母順序排列（`stories` 除外）
   - 所有物件鍵按字母順序排列
5. **交叉引用完整性**：
   - `permissions` 的鍵對應 `actors` 中的條目（若 actors 已定義則為錯誤）
   - `denied` 項目不與 `entities` 或 `features` 衝突
   - Story 引用應對應已知的 actors/entities（警告，非錯誤）
6. **鍵的順序**：中繼資料優先，然後依序為 actors → entities → features → stories → permissions → denied

若鎖定檔本身無效，請在進行程式碼審查之前報告鎖定檔驗證錯誤。

### 階段 2：掃描程式碼庫

建立程式碼庫的心智模型：

1. **尋找資料模型**：資料庫 schema、ORM 模型、具有持久化欄位的型別定義
2. **尋找功能**：API 路由、處理器、UI 動作、背景工作
3. **尋找驗證/角色**：中介層、守衛、角色定義、權限檢查
4. **尋找被拒絕的項目**：搜尋任何與被拒絕名稱匹配的內容

你需要充分理解程式碼庫，以回答以下問題：
- 存在哪些實體？
- 存在哪些功能/能力？
- 存在哪些角色/權限？
- Story 中描述的互動流程是否確實運作？

### 階段 3：驗證每個欄位

將鎖定檔中的每個產品欄位與程式碼庫進行比對。詳見[第 4 節](#4-逐欄位驗證)。

---

## 4. 逐欄位驗證

### 4.1 驗證 `actors`

**檢查內容：** 鎖定檔中的每個角色在程式碼庫中皆以可識別的使用者角色存在。

**查找位置：**
- 驗證中介層、角色列舉、權限常數
- 資料庫：User 模型上的 `role` 欄位
- RBAC/ABAC 設定檔
- 路由守衛、存取控制裝飾器

**通過標準：**
- 每個列出的角色對應系統中的真實角色 → PASS
- 角色未找到 → VIOLATION
- 程式碼中有額外角色不在鎖定檔中 → WARNING（未宣告的角色）

**範例：**
```json
"actors": ["Admin", "Guest", "Member"]
```
- 程式碼中有 Admin、Guest、Member 角色 → PASS
- 程式碼中還有不在鎖定檔中的 "SuperAdmin" 角色 → WARNING

---

### 4.2 驗證 `entities`

**寬鬆模式**（`string[]`）：檢查每個命名的實體是否以資料模型的形式存在。

**查找位置：**
- 資料庫 schema（Prisma、TypeORM、SQLAlchemy、Mongoose 等）
- 具有持久化欄位的型別定義
- GraphQL 型別定義
- 對應到儲存資料的 API 回應型別

**通過標準（寬鬆模式）：**
- 實體以可識別的資料模型存在 → PASS
- 實體缺失 → VIOLATION

**嚴格模式**（`Record<string, string[]>`）：

對於具有 `[]`（空陣列）的實體：
- 與寬鬆模式相同——僅檢查實體是否存在

對於具有欄位清單的實體：
- 檢查實體是否**精確**具有這些欄位——不多不少
- 欄位名稱必須匹配（區分大小寫）
- 欄位型別不檢查——屬於實作細節

**範例：**
```json
"entities": {
  "User": ["avatar", "email", "id", "name"],
  "Conversation": []
}
```
- User 模型具有欄位 `id`、`name`、`email`、`avatar` → PASS
- User 模型有額外欄位 `createdAt` → VIOLATION（嚴格模式：不多不少）
- User 模型缺少 `avatar` → VIOLATION
- Conversation 模型存在 → PASS（欄位未鎖定）

**重要：** 不要將實作用的資料表與產品實體混淆。`CachedQuote`、`MigrationHistory`、`SessionStore` 是實作層面的，不是產品實體。僅驗證鎖定檔中明確列出的實體。

---

### 4.3 驗證 `features`

**檢查內容：** 每個功能在程式碼庫中以可識別的行為存在。

**查找位置：**
- API 路由處理器（每個端點通常 = 一個功能）
- 具有使用者可觸發動作的 UI 元件
- 實作能力的服務層函式
- 背景工作定義

**通過標準：**
- 功能以可識別的行為存在 → PASS
- 功能未找到 → VIOLATION
- 功能僅部分實作 → VIOLATION（要麼可用，要麼不可用）

**如何在程式碼中識別功能：**

像 `sendMessage` 這樣的功能被視為「存在」的條件：
- 存在允許發送訊息的 API 端點、處理器或函式
- 該功能可達（非死碼）
- 確實可運作（非僅為存根）

你不檢查：
- 實作方式（WebSocket vs REST vs GraphQL——不重要）
- 效能特性
- 程式碼品質

**範例：**
```json
"features": ["createGroup", "sendMessage", "readReceipts"]
```
- POST /api/messages 端點存在且可運作 → `sendMessage` PASS
- POST /api/groups 端點存在且可運作 → `createGroup` PASS
- 已讀回條建立邏輯存在且可觸發 → `readReceipts` PASS

---

### 4.4 驗證 `stories`

**檢查內容：** 每個描述的互動流程、體驗或行為在程式碼庫中存在。

Story 是最難驗證的，因為它們是自然語言。將每個 story 拆解並驗證其組成部分。

**拆解方法：**

Story：`"Member sends Message to Conversation"`
1. 角色 "Member" 存在 → 檢查
2. 實體 "Message" 存在 → 檢查
3. 實體 "Conversation" 存在 → 檢查
4. 存在一條程式碼路徑讓 member 可以向 conversation 發送 message → 檢查

Story：`"User views StockQuote with data refreshing in real-time"`
1. 角色 "User" 存在 → 檢查
2. 存在某種股票報價顯示功能 → 檢查
3. 存在即時或近即時的資料更新機制 → 檢查

Story：`"System detects SupplyChainDisruption from NewsArticle matching SupplyChainSegment"`
1. 實體 "NewsArticle" 存在 → 檢查
2. 實體 "SupplyChainSegment" 存在 → 檢查
3. 中斷偵測邏輯存在 → 檢查
4. 它使用新聞文章並與供應鏈區段進行匹配 → 檢查

**通過標準：**
- Story 的所有組成部分皆存在 → PASS
- 核心流程按描述運作 → PASS
- 關鍵組成部分缺失 → VIOLATION
- 流程存在但運作方式與描述不同 → VIOLATION

**不檢查：**
- 確切的實作方式
- 程式碼品質或效率
- 邊界情況處理

**三種 story 類型及其驗證方式：**

| 類型 | 範例 | 驗證方式 |
|------|---------|--------------|
| 功能流程 | "Member sends Message" | API 端點 + 驗證檢查 + 資料持久化 |
| 體驗期望 | "User views data loading instantly" | 資料展示存在 + 快取/最佳化機制存在 |
| 系統行為 | "System fetches data on schedule" | 背景工作/排程任務存在 + 資料來源整合存在 |

---

### 4.5 驗證 `permissions`

**字串模式**（例如 `"rbac"`）：
- 檢查產品是否使用指定的存取控制模型
- 若為 "rbac"：驗證存在基於角色的檢查（角色欄位、中介層、守衛）
- 若為 "abac"：驗證存在基於屬性的檢查
- 你不檢查具體的角色分配

**物件模式**（角色-權限矩陣）：
- 每個鍵必須對應一個角色
- 每個角色必須**精確**具有列出的權限——不多不少
- 透過檢查路由守衛、中介層、角色檢查來驗證

**範例：**
```json
"permissions": {
  "Admin": ["createGroup", "deleteMessage", "sendMessage"],
  "Guest": ["listConversations"],
  "Member": ["createGroup", "sendMessage"]
}
```

驗證：
- Admin 可以 createGroup → 檢查路由守衛允許 admin
- Admin 可以 deleteMessage → 檢查路由守衛允許 admin
- Admin 可以 sendMessage → 檢查路由守衛允許 admin
- Admin 不能執行其他列出的操作 → 檢查沒有其他路由對 admin 開放
- Guest 只能 listConversations → 檢查 guest 被阻止使用所有其他功能
- Member 可以 createGroup 和 sendMessage → 檢查
- Member 不能 deleteMessage → 檢查 member 被阻止使用此功能

**常見違規：**
- 角色擁有比列出更多的權限 → VIOLATION
- 角色擁有比列出更少的權限 → VIOLATION
- 權限鍵不匹配任何角色 → LOCK VALIDATION ERROR

---

### 4.6 驗證 `denied`

**這是整個審查中最重要的檢查。** 被拒絕的項目是防止 AI 超出產品範圍的護欄。將被拒絕的違規視為最高嚴重性。

被拒絕的項目必須不存在於程式碼庫中的任何位置。

**寬鬆模式**（`string[]`）：在整個程式碼庫中搜尋被拒絕項目的任何跡象。

**附帶原因**（`Record<string, string>`）：相同的驗證——搜尋每個鍵。原因記錄了產品決策（僅供參考，不改變驗證方式）。報告時使用原因來理解排除的重要性。

```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "voiceCall": "Text-based communication only"
}
```

**如何搜尋被拒絕的項目：**

對於被拒絕的實體（例如 `"Reaction"`）：
- 搜尋名為 Reaction 的 model/schema/type
- 搜尋名為 "reaction" 或 "reactions" 的資料庫表
- 搜尋與 reaction 相關的 API 端點
- 搜尋與 reaction 相關的 UI 元件

對於被拒絕的功能（例如 `"editMessage"`）：
- 搜尋訊息的編輯功能
- 搜尋訊息的 PUT/PATCH 端點
- 搜尋訊息上的 UI 編輯按鈕

**通過標準：**
- 被拒絕的項目在任何地方都未找到 → PASS
- 被拒絕的項目有任何蹤跡被發現 → VIOLATION

**報告被拒絕違規時，應包含鎖定檔中的原因：**
```
[FAIL] executeTrade — FOUND: /api/trade endpoint exists
       Reason in lock: "Market intelligence only — no brokerage liability"
       Severity: HIGH (liability boundary violated)
```

**重要的邊界情況：**
- 實作被拒絕功能的死碼 → 仍為 VIOLATION（它存在於程式碼庫中）
- 提及被拒絕功能的註解 → 非違規（註解不是程式碼）
- 模擬被拒絕功能的測試 → 非違規（測試不是產品程式碼）
- 為未來被拒絕功能準備的設定/環境變數 → VIOLATION（這是為其做準備）

---

## 5. 未宣告項目

除了檢查鎖定檔中的內容外，還要注意程式碼庫中**應該**在鎖定檔中但未列入的項目。

這是**警告**，而非違規：

- 實體存在於程式碼中但不在鎖定檔中 → WARNING：「未宣告的實體：Payment」
- 重要功能存在但不在鎖定檔中 → WARNING：「未宣告的功能：exportData」
- 使用者角色存在但不在 actors 中 → WARNING：「未宣告的角色：Moderator」

由人類決定是否將這些項目加入鎖定檔或忽略它們。

---

## 6. 審查報告格式

你的產出是一份結構化的審查報告：

```
# Product Lock 審查：{name} v{version}

## 摘要
- 狀態：PASS | FAIL
- 違規：{count}
- 警告：{count}

## 鎖定檔驗證
- [PASS/FAIL] 鎖定檔結構有效
- [PASS/FAIL] 命名慣例正確
- [PASS/FAIL] 交叉引用完整性

## Actors
- [PASS] Admin — 角色存在於驗證中介層中
- [PASS] Member — 角色存在於驗證中介層中
- [WARN] 程式碼中發現未宣告的角色 "Moderator"

## Entities
- [PASS] User — 模型存在，欄位為：id, name, email, avatar
- [PASS] Message — 模型存在且欄位正確
- [FAIL] ReadReceipt — 在程式碼庫中未找到實體

## Features
- [PASS] sendMessage — POST /api/messages 端點
- [PASS] createGroup — POST /api/groups 端點
- [FAIL] readReceipts — 未找到實作

## Stories
- [PASS] "Member sends Message to Conversation" — 已驗證：POST /api/messages 附帶驗證檢查
- [FAIL] "Guest views Conversation but cannot send Message" — Guest 可以發送訊息（POST /api/messages 無角色守衛）

## Permissions
- [PASS] Admin 具有正確的權限
- [FAIL] Member 也可以 deleteMessage（不在鎖定檔中）
- [PASS] Guest 僅限於 listConversations

## Denied
- [PASS] Reaction — 在程式碼庫中未找到
- [PASS] voiceCall — 在程式碼庫中未找到
- [FAIL] editMessage — PUT /api/messages/:id 端點存在
         Reason in lock: "Messages are immutable once sent"

## 未宣告項目
- [WARN] 實體 "Notification" 存在於程式碼中但不在鎖定檔中
- [WARN] 功能 "archiveConversation" 存在但不在鎖定檔中
```

---

## 7. 決策框架

### 「這是否為違規？」

| 情況 | 判定 | 原因 |
|-----------|---------|--------|
| 鎖定的實體在程式碼中缺失 | VIOLATION | 鎖定檔表示它必須存在 |
| 程式碼中有額外實體不在鎖定檔中 | WARNING | 未鎖定的即為自由 |
| 鎖定的功能無法運作 | VIOLATION | 鎖定檔表示它必須存在 |
| 程式碼中有額外功能不在鎖定檔中 | WARNING | 未鎖定的即為自由 |
| 被拒絕的項目在程式碼中找到 | VIOLATION | 被拒絕的項目一律檢查 |
| 被拒絕的項目僅在測試程式碼中找到 | NOT violation | 測試不是產品程式碼 |
| Story 流程中斷 | VIOLATION | 鎖定檔表示它必須運作 |
| Story 的實作方式與想像不同 | NOT violation | 你檢查的是「做什麼」，而非「怎麼做」 |
| 權限過於寬泛（額外權限） | VIOLATION | 嚴格模式 = 精確匹配 |
| 權限過於狹窄（缺少權限） | VIOLATION | 嚴格模式 = 精確匹配 |
| 欄位從鎖定檔中省略 | SKIP | 非你的職責 |

### 「我是否應該報告這個？」

- **違規** → 一律報告，這些是硬性失敗
- **警告** → 作為「未宣告項目」報告，供人類決策
- **建議** → 不要包含；你是審查者，不是顧問
- **實作意見** → 不要包含；你檢查的是「做什麼」，而非「怎麼做」

---

## 8. 不在檢查範圍內的項目

你的範圍是**產品邊界**，而非程式碼品質：

- 程式碼風格或格式
- 效能或效率
- 測試覆蓋率
- 安全漏洞（除非被拒絕的項目本身是安全功能）
- 架構決策
- 依賴套件選擇
- 框架使用方式
- 檔案組織
- 錯誤處理品質
- 文件品質

這些是 AI 工作者的職責，不是你的。

---

## 9. 邊界情況

### 實體存在但名稱大小寫不同
```
Lock: "ReadReceipt"
Code: "read_receipt"（Python snake_case 資料表）
```
→ PASS。鎖定檔使用 PascalCase 慣例。程式碼適應語言慣例。只要實體在語義上匹配，即通過。

### 功能存在但位於功能旗標之後（已停用）
→ VIOLATION。若功能在目前的程式碼庫中未啟用/不可達，則不算「存在」。

### Story 引用了不在鎖定檔中的實體
```
Story: "System creates Notification when Message received"
Entities: ["Message", "User"]（無 Notification）
```
→ 鎖定檔驗證階段的 WARNING（story 引用了未列出的實體）。但對於程式碼審查：檢查通知流程是否確實運作。

### 被拒絕的項目以資料庫欄位形式存在但未作為功能實作
```
Denied: "editMessage"
Code: Message 模型具有 "editedAt" 欄位，但無編輯端點
```
→ 需依情況判斷。若欄位存在但編輯功能未實作（無端點、無 UI），這是 WARNING，而非違規。被拒絕的項目是指能力，而非欄位。

### 同一功能的多種實作
```
Feature: "sendMessage"
Code: REST API + WebSocket 兩者皆可發送訊息
```
→ PASS。功能存在。多種實作不改變此事實。

---

*本指南是 [Product Lock 規範](https://spec.productlock.org) 的一部分。*
