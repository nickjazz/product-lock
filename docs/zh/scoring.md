---
layout: default
title: 評分系統
nav_order: 8
permalink: /zh/scoring
lang: zh
lang_label: 繁體中文
page_id: scoring
---

# Product Lock 評分系統

### 版本 0.1.0 — 2026 年 2 月

> 一套從 product.lock.json 衍生的複雜度評分系統。
>
> 相同的 lock，相同的分數。此分數衡量的是產品複雜度，而非程式碼複雜度。

完整規格請參閱 [Product Lock Specification](https://spec.productlock.org)。

---

## 目錄

1. [衡量什麼](#1-衡量什麼)
2. [四個維度](#2-四個維度)
3. [公式](#3-公式)
4. [分數等級](#4-分數等級)
5. [計算規則](#5-計算規則)
6. [基準測試](#6-基準測試)
7. [使用案例](#7-使用案例)

---

## 1. 衡量什麼

Product Lock Score (PLS) 衡量軟體產品的**產品邊界複雜度**。它回答的問題是：「從產品角度來看，這個產品有多複雜？」

PLS 衡量的內容：
- 產品管理多少資料
- 產品有多少功能
- 各部分之間的關聯程度
- 存取控制的複雜程度

PLS 不衡量的內容：
- 程式碼品質或程式碼規模
- 實作的技術難度
- 效能需求
- 基礎設施複雜度

**分數完全衍生自 product.lock.json。** 相同的 lock，相同的分數。程式語言、框架和架構不會影響分數。

---

## 2. 四個維度

產品複雜度拆解為四個獨立的維度：

### D — 資料複雜度

產品儲存了多少資料。更多的實體搭配更多的欄位 = 更高的資料複雜度。

```
D = entity_count × avg_fields_per_entity × 0.3
```

一個擁有 45 個實體、平均 14 個欄位的產品（Sentry）在資料複雜度上遠高於一個擁有 7 個實體、平均 6 個欄位的產品（Hacker News）。

### F — 功能複雜度

產品能做什麼。每個 feature 都是一個可獨立識別的功能。

```
F = feature_count × 1.0
```

權重設為 1.0，因為 feature 是產品能力最直接的衡量指標。擁有 180 個 feature 的產品（GitLab）比擁有 12 個 feature 的產品（Hacker News）做的事情多得多。

### I — 互動複雜度

各部分如何連接。Story 描述跨實體的流程——story 越多，產品的關聯性就越強。

```
I = story_count × 0.5
```

權重設為 0.5，因為 story 是敘事性的（一個 story 可能涵蓋多個互動），其數量不如實體或 feature 那樣精確。

### A — 存取複雜度

誰能做什麼。更多的角色搭配更多不同的權限組合 = 更高的存取控制複雜度。

```
A = permission_matrix_size × 0.1
```

Permission matrix size = 有意義的角色-功能組合數量。權重設為 0.1，因為存取控制增加的是組織複雜度，但不會改變產品本身的功能。

---

## 3. 公式

```
PLS = D + F + I + A

Where:
  D = entity_count × avg_fields_per_entity × 0.3
  F = feature_count × 1.0
  I = story_count × 0.5
  A = permission_matrix_size × 0.1
```

### 如何從 product.lock.json 讀取各欄位

| 變數 | 來源 | 計算方式 |
|------|------|---------|
| `entity_count` | `entities` | entity 鍵的數量（loose 或 strict 模式） |
| `avg_fields_per_entity` | `entities` | 欄位陣列的平均長度（僅限 strict 模式） |
| `feature_count` | `features` | features 陣列的長度 |
| `story_count` | `stories` | stories 陣列的長度 |
| `permission_matrix_size` | `permissions` | 所有角色的權限陣列長度總和 |

### 缺失欄位的處理方式

| 情境 | 規則 |
|------|------|
| `entities` 為 loose 模式（`string[]`） | 使用預設值：每個 entity 8 個欄位 |
| `entities` 為 strict 模式但值為 `[]` | 該 entity 計為 0 個欄位 |
| `features` 省略 | F = 0 |
| `stories` 省略 | I = 0 |
| `permissions` 省略 | A = 0 |
| `permissions` 為字串（`"rbac"`） | 使用預設值：actors × features × 0.6 |
| `actors` 省略 | 使用 1（單一使用者類型） |
| `denied` | 不計入分數（屬於邊界成熟度，非複雜度） |

### 為什麼排除 `denied`

`denied` 衡量的是邊界成熟度，而非產品複雜度。一個有 50 個 denied 項目的產品並不比有 0 個的更複雜——它是更受約束的。約束是複雜度的反面。

---

## 4. 分數等級

| PLS 範圍 | 等級 | 描述 | 範例 |
|----------|------|------|------|
| < 50 | **簡單** | 單一用途工具，最小資料模型，少量功能 | Hacker News (32), Excalidraw (44) |
| 50–150 | **中等** | 有明確領域的聚焦型產品，中等資料模型 | Plausible (58), Ghost (145) |
| 150–300 | **複雜** | 多功能平台，豐富資料模型，跨實體互動 | Cal.com (162), Discourse (170), Supabase (250) |
| 300–500 | **非常複雜** | 企業級平台，大量資料模型，深層權限系統 | Elasticsearch (287), Sentry (325) |
| 500+ | **巨型** | 多產品平台，數百個實體和功能 | GitLab (1037) |

### 各等級的實務意涵

| 等級 | 人工審查時間 | AI 生成範圍 | 典型 lock 大小 |
|------|------------|------------|---------------|
| 簡單 | 數秒 | 單次提示 | < 30 行 |
| 中等 | 1–3 分鐘 | 單次會話 | 30–80 行 |
| 複雜 | 3–10 分鐘 | 多次會話 | 80–200 行 |
| 非常複雜 | 10–30 分鐘 | 分階段生成 | 200–500 行 |
| 巨型 | 30 分鐘以上 | AI 代理團隊 | 500+ 行 |

---

## 5. 計算規則

### 規則 1：只計算產品實體

只計算出現在 `entities` 欄位中的實體。實作用的資料表（快取、佇列、遷移紀錄）不應出現在 lock 中，因此不影響分數。

### 規則 2：Strict 模式會產生更高的分數

Strict 模式鎖定更多欄位，產生更高（且更準確）的 D 分數。這是刻意的設計——一個指定了確切欄位的產品，擁有更明確（因此更可衡量）的資料模型。

```json
// Loose: entity_count=3, avg_fields=8 (default)
// D = 3 × 8 × 0.3 = 7.2
"entities": ["User", "Post", "Comment"]

// Strict: entity_count=3, avg_fields=10
// D = 3 × 10 × 0.3 = 9.0
"entities": {
  "User": ["avatar", "email", "id", "name", "role"],
  "Post": ["authorId", "content", "createdAt", "id", "slug", "status", "tags", "title"],
  "Comment": ["authorId", "content", "createdAt", "id", "postId", "replyToId", "status"]
}
```

### 規則 3：Permission matrix 計算實際條目數

當 permissions 為物件時，計算權限條目的總數：

```json
"permissions": {
  "Admin": ["createPost", "deletePost", "manageUsers", "publishPost"],
  "Editor": ["createPost", "publishPost"],
  "Viewer": ["viewPost"]
}
// permission_matrix_size = 4 + 2 + 1 = 7
```

當 permissions 為字串（如 `"rbac"`）時，使用估算值：`actors × features × 0.6`。

### 規則 4：分數具有確定性

相同的 lock 永遠產生相同的分數。沒有隨機性、沒有外部資料、沒有主觀判斷。公式僅使用 lock 檔案中的資料。

### 規則 5：四捨五入至整數

最終的 PLS 四捨五入至最接近的整數。

---

## 6. 基準測試

已針對 10 個知名開源產品進行驗證。所有分數均符合公認的複雜度排名。

| # | 產品 | Entities | AvgFields | Features | Stories | PermMatrix | D | F | I | A | **PLS** | 等級 |
|---|------|----------|-----------|----------|---------|------------|---|---|---|---|---------|------|
| 1 | Hacker News | 7 | 6 | 12 | 10 | 26 | 12.6 | 12 | 5 | 2.6 | **32** | 簡單 |
| 2 | Excalidraw | 8 | 10 | 14 | 10 | 10 | 24 | 14 | 5 | 1 | **44** | 簡單 |
| 3 | Plausible | 12 | 8 | 18 | 15 | 36 | 28.8 | 18 | 7.5 | 3.6 | **58** | 中等 |
| 4 | Ghost | 22 | 12 | 34 | 45 | 90 | 79.2 | 34 | 22.5 | 9 | **145** | 中等 |
| 5 | Cal.com | 26 | 11 | 38 | 55 | 105 | 85.8 | 38 | 27.5 | 10.5 | **162** | 複雜 |
| 6 | Discourse | 28 | 10 | 42 | 60 | 135 | 84 | 42 | 30 | 13.5 | **170** | 複雜 |
| 7 | Supabase | 40 | 11 | 78 | 35 | 220 | 132 | 78 | 17.5 | 22 | **250** | 複雜 |
| 8 | Elasticsearch | 35 | 12 | 90 | 45 | 480 | 126 | 90 | 22.5 | 48 | **287** | 複雜 |
| 9 | Sentry | 45 | 14 | 75 | 60 | 310 | 189 | 75 | 30 | 31 | **325** | 非常複雜 |
| 10 | GitLab | 120 | 18 | 180 | 250 | 840 | 648 | 180 | 125 | 84 | **1037** | 巨型 |

### 驗證

評分產生的排名與廣泛認可的複雜度認知一致：

```
Hacker News (32) < Excalidraw (44) < Plausible (58) < Ghost (145) < Cal.com (162)
< Discourse (170) < Supabase (250) < Elasticsearch (287) < Sentry (325) < GitLab (1037)
```

基準測試的主要觀察：

- **簡單產品**（< 50）：單一用途，< 10 個實體，< 15 個 feature
- **中等產品**（50–150）：聚焦領域，10–25 個實體，15–35 個 feature
- **複雜產品**（150–300）：多功能平台，25–40 個實體，35–80 個 feature
- **非常複雜的產品**（300–500）：企業級平台，35–50 個實體，75–90 個 feature，深層權限模型
- **巨型產品**（500+）：多產品平台，100+ 個實體，150+ 個 feature，200+ 個 story

---

## 7. 使用案例

### 比較產品

> 「我們的產品比 Ghost 更複雜嗎？」將兩個產品都 lock 起來，比較分數即可。

### 追蹤複雜度成長

> Version 1.0: PLS 85（中等）
> Version 2.0: PLS 160（複雜）
> Version 3.0: PLS 340（非常複雜）

如果分數出現意外跳躍，可能表示範圍蔓延（scope creep）。檢查 lock 版本之間的差異，找出新增了什麼。

### 估算工作量

分數與實作工作量相關。一個複雜產品（150–300）所需的工作量遠超過簡單產品（< 50）。使用基準測試產品作為參考點：

- 「我們的產品分數是 170，與 Discourse 相近。預期規模相當。」
- 「我們的產品分數是 45，與 Excalidraw 相近。這是一個聚焦型工具。」

### 範圍控制

為產品版本設定目標 PLS：

> 「MVP 必須維持在 PLS 100 以下。超過的功能移至 v2。」

如果新增一個 feature 導致分數超過目標，這是重新考慮範圍的訊號。

### 比較 lock 版本

```
v1.0.0 → v1.1.0: PLS 120 → 128 (+8)     // 小幅功能新增
v1.1.0 → v2.0.0: PLS 128 → 195 (+67)     // 大幅範圍擴展
v2.0.0 → v2.1.0: PLS 195 → 210 (+15)     // 中等幅度新增
v2.1.0 → v3.0.0: PLS 210 → 340 (+130)    // 警示：範圍可能正在蔓延
```

---

*此評分系統是 [Product Lock Specification](https://spec.productlock.org) 的一部分。*
