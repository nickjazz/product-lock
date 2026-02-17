---
layout: default
title: 計畫條目規格
nav_order: 7
permalink: /zh/plan-entry-spec
lang: zh
lang_label: 繁體中文
page_id: plan-entry-spec
---

# 產品計畫條目規格

### 版本 0.1.0 — 2026 年 2 月

> 計畫條目記錄開發前和開發中的實作計畫。
>
> 所有計畫存放在 `plans/` 目錄中，取代單一的 `product.plan.md` 檔案。
>
> Plan = HOW & WHY（AI 打算怎麼做）。Worklog = WHAT & WHEN（AI 實際做了什麼）。

完整的 lock 規格請參閱 [Product Lock 規格](https://spec.productlock.org)。

---

## 目錄

1. [與其他檔案的關係](#1-與其他檔案的關係)
2. [設計原則](#2-設計原則)
3. [目錄與命名](#3-目錄與命名)
4. [格式](#4-格式)
5. [必要章節](#5-必要章節)
6. [選用章節 — 一般](#6-選用章節--一般)
7. [選用章節 — 完整藍圖](#7-選用章節--完整藍圖)
8. [Plan 與 Worklog 的比較](#8-plan-與-worklog-的比較)

---

## 1. 與其他檔案的關係

```
plans/*.md        →  計畫（HOW & WHY — AI 打算怎麼做）
worklogs/*.md     →  紀錄（WHAT & WHEN — AI 實際做了什麼）
product.lock.json →  邊界（WHAT — 產品是什麼）
```

Plan 和 Worklog 獨立但互補：
- **Plan = 事前** — 記錄意圖、方法、步驟、風險
- **Worklog = 事後** — 記錄實際變更、決策、發現
- 透過日期/標題鬆散關聯，不強制 1:1 對應

`plans/` 目錄取代單一的 `product.plan.md` 檔案。所有計畫 — 從完整的產品藍圖到輕量的工作階段計畫 — 都放在同一個目錄中，透過內容和命名自然區分。

---

## 2. 設計原則

1. **可執行** — 計畫是 AI 可以遵循實作的具體指南
2. **有範圍** — 每個計畫都有明確的目標和邊界
3. **純文字** — Markdown 格式，人類可讀
4. **粒度彈性** — 從完整產品藍圖到單一工作階段任務計畫
5. **非阻塞** — 計畫不需要核准才能存在；它們記錄意圖

---

## 3. 目錄與命名

計畫必須（MUST）放置在專案根目錄的 `plans/` 目錄中。

**命名慣例：** `{YYYY-MM-DD}-{kebab-title}.md`

範例：
- `plans/2026-02-17-add-payment-entity.md`
- `plans/2026-02-18-full-product-blueprint.md`
- `plans/2026-02-19-refactor-auth-flow.md`

```
project-root/
├── product.lock.json
├── plans/
│   ├── 2026-02-17-add-payment-entity.md
│   ├── 2026-02-18-full-product-blueprint.md
│   └── 2026-02-19-refactor-auth-flow.md
└── worklogs/
    └── ...
```

---

## 4. 格式

```markdown
# Plan: {title}

### Context
- Date: {YYYY-MM-DD}
- Branch: `{branch}`
- Agent: {誰建立了這個計畫}
- Lock version: {版本}
- PLS: {分數}
- Scope: {範圍描述}

### Goal
{這個計畫要達成什麼 — 1-3 句。}

### Approach
{高層策略 — 如何完成目標。}

### Steps
1. {第一步}
2. {第二步}
3. ...
```

---

## 5. 必要章節

每個計畫條目都必須（MUST）包含以下章節。

### 5.1 Context

關於計畫建立時間和位置的元資料。

| 欄位 | 必要 | 說明 |
|------|------|------|
| Date | 是 | `YYYY-MM-DD` |
| Branch | 是 | Git 分支名稱 |
| Agent | 否 | 誰建立了計畫（預設：「AI」） |
| Lock version | 否 | 目前的 lock 版本 |
| PLS | 否 | 目前的 PLS 分數 |
| Scope | 否 | 簡短的範圍描述 |

### 5.2 Goal

一到三句話描述這個計畫要達成什麼。這是成功標準 — 當目標達成時，計畫就完成了。

規則：
- 必須（MUST）具體且可量測
- 應該（SHOULD）在適用時參考 lock 概念（entities、features）
- 最多一到三句

### 5.3 Approach

高層策略，解釋「如何」和「為什麼」。在相關時提及考慮過的替代方案。

### 5.4 Steps

有序的、編號的實作步驟列表。

規則：
- 編號、有序
- 每個步驟應該（SHOULD）在可能時可獨立驗證
- 目標 3–15 個步驟
- 使用祈使句：「新增 X」、「更新 Y」、「設定 Z」

---

## 6. 選用章節 — 一般

在與計畫相關時加入這些章節。

### 6.1 Background

額外脈絡：先前經驗、相關 issue、使用者需求。

### 6.2 Constraints

影響方法選擇的已知限制或要求。

```markdown
### Constraints
- 不得破壞現有 API
- 回應時間低於 200ms
- 必須相容 Node 18+
```

### 6.3 Risks

潛在問題及其緩解措施。

```markdown
### Risks
- Stripe webhook 傳遞可能延遲 — 實作冪等鍵
- 大型資料表的資料庫遷移 — 在離峰時段執行
```

### 6.4 Expected Outcome

計畫執行後程式碼庫應該呈現的樣貌。對審查者驗證很有用。

### 6.5 References

Issue、PR、規格或外部資源的連結。

---

## 7. 選用章節 — 完整藍圖

對於全面性的產品計畫（取代舊的 `product.plan.md`），可以加入以下額外章節：

| 章節 | 何時加入 |
|------|---------|
| Stack | 每一層的技術選型 |
| Schemas | 包含型別、約束、關聯的完整實體定義 |
| Endpoints | 每個功能的 API 契約 |
| Architecture | 非簡單的系統架構模式（微服務、CQRS 等） |
| Auth & Permissions | 驗證方法、會話管理、角色對應、RLS |
| File Structure | 非標準的專案佈局 |

這些章節遵循[計畫規格](/zh/plan-spec)中描述的相同格式。

---

## 8. Plan 與 Worklog 的比較

| 面向 | Plan | Worklog |
|------|------|---------|
| 時機 | 實作前/中 | 實作後 |
| 目的 | 意圖和方法（HOW & WHY） | 實際變更（WHAT & WHEN） |
| 內容 | 目標、方法、步驟、風險 | 變更、檔案、決策 |
| 粒度 | 彈性（藍圖到任務） | 每個工作階段 |
| 驗證 | 審查者檢查程式碼是否符合計畫 | 審查者檢查日誌是否符合變更 |
| 目錄 | `plans/` | `worklogs/` |
| 命名 | `{YYYY-MM-DD}-{kebab-title}.md` | `{YYYY-MM-DD}-{kebab-title}.md` |

### 審查者整合

審查產品時，AI 審查者會檢查計畫與程式碼庫的對應，並報告：

| 狀態 | 意義 |
|------|------|
| MATCH | 實作與計畫一致 |
| DEVIATION | 實作與計畫有顯著差異 |
| INCOMPLETE | 計畫僅部分實作 |
| UNPLANNED | 存在沒有對應計畫的重要程式碼 |

---

*本規格為 [Product Lock 規格](https://spec.productlock.org) 的一部分。*
