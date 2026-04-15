# JDN LLM Wiki — LLM 維護型知識庫架構規格

> **用途**：本文件為 AI 代理的系統規格說明。  
> 任何代理（Claude Code、Codex、Antigravity 或等效 LLM 代理）完整閱讀本文件後，  
> 無需任何額外的人工說明，即可從零複製整個知識庫檔案管理策略。
>
> **不含個人資料。** 所有範例僅為結構性與方法論示範。  
> **規格日期**：2026-04-15 ｜ **規格版本**：1.0 ｜ **語言版本**：繁體中文

---

## 1. 系統哲學

### 1.1 核心原則

「JDN LLM Wiki」是一套**以角色為基礎、由 LLM 維護的知識庫**，其中：

- **人類**提供原始素材（PDF、筆記、文章、資料匯出）
- **LLM 代理**閱讀素材並合成結構化 Wiki 頁面
- **Git** 是規範性的真相來源與版本歷史
- **Obsidian** 提供人類可讀的本地編輯環境（透過 obsidian-git 同步）
- **Notion** 作為任務/產出的匯集點與摘要儀表板（透過 MCP）
- **知識圖譜**（Graphify）揭示跨頁面的隱藏結構

### 1.2 設計公理

1. **原始素材唯讀** — 代理永遠不修改來源檔案
2. **Wiki 頁面由代理維護** — 代理負責撰寫、更新、建立交叉引用
3. **每次操作皆留紀錄** — `wiki/log.md` 僅可追加（append-only）
4. **索引保持最新** — `wiki/index.md` 在每次 ingest 後反映最新頁面清單
5. **結構優先於內容** — 每頁強制執行 frontmatter schema
6. **角色劃分空間** — 每頁面恰好屬於一個角色命名空間

### 1.3 何時使用此架構

在以下情況適合使用本系統：
- 擁有需要合成而非單純儲存的多元素材
- 有多個「身份/角色」或工作流，彼此共享部分知識但需要分隔
- 需要人類探索（Obsidian）與代理檢索（Wiki 頁面）並存
- 長期累積知識，未加結構管理時可發現性會下降

---

## 2. 目錄架構

```
{PROJECT_ROOT}/           ← Git repo 根目錄（同時也是 Obsidian Vault 根目錄）
├── WIKI.md               ← Schema 文件（代理必須先讀此檔）
├── raw/                  ← 原始素材（唯讀，永不修改）
│   ├── {角色-A}/         ← 依角色分類的原始檔案
│   ├── {角色-B}/
│   └── {角色-N}/
├── wiki/                 ← LLM 維護的知識頁面
│   ├── index.md          ← 主索引（每次 ingest 後更新）
│   ├── log.md            ← 僅追加的操作日誌
│   ├── overview.md       ← 跨角色高層次綜述（選填）
│   ├── {角色-A}/         ← 每個角色使用中文或描述性目錄名
│   │   ├── {主題}.md
│   │   └── {子主題}/
│   │       └── {項目}.md
│   ├── {角色-B}/
│   └── cross-role/       ← 跨越多個角色的頁面
│       ├── reuse-opportunities.md
│       └── {合成主題}.md
├── docs/                 ← 人類撰寫的設計文件（非代理維護）
├── artifacts/            ← 代理生成的產出物（PDF、簡報、docx）
├── archive/              ← 已棄用/已替換的素材
└── graphify-out/         ← 知識圖譜輸出（自動生成）
    ├── GRAPH_REPORT.md   ← 人類可讀的圖譜分析
    ├── graph.html        ← 互動式視覺化
    ├── graph.json        ← 原始圖譜資料
    ├── manifest.json     ← 檔案追蹤清單
    └── cache/            ← 每個檔案的語意提取快取
```

### 2.1 命名慣例

| 項目 | 慣例 | 範例 |
|------|------|------|
| Wiki 頁面檔案 | 英文 kebab-case `.md` | `academic-profile.md` |
| 角色目錄 | 中文或完整描述性名稱 | `研究者/`、`Developer/` |
| 子主題目錄 | 與角色目錄相同 | `academic-base/`、`古文30篇/` |
| 原始素材檔案 | 保留原始檔名 | `paper-2024.pdf` |
| 跨角色頁面 | 英文 kebab-case，置於 `cross-role/` | `reuse-opportunities.md` |

### 2.2 內容歸屬位置

| 內容類型 | 位置 | 代理可寫入？ |
|---------|------|------------|
| 原始 PDF、筆記、匯出資料 | `raw/` | ❌ 永不 |
| 合成後的 Wiki 頁面 | `wiki/` | ✅ 是 |
| 設計決策、規格文件 | `docs/` | ❌ 人類專屬 |
| 生成的產出物（docx、PDF） | `artifacts/` | ✅ 是 |
| 圖譜輸出 | `graphify-out/` | ✅ 自動生成 |

---

## 3. 角色型組織模型

### 3.1 什麼是角色

**角色**是一個具有以下特徵的一致性人設：
- 觸發詞彙（啟動該角色的關鍵詞）
- 主要產出物（該角色生產什麼）
- 原始素材類型（該角色消費哪些來源）
- Wiki 命名空間（`wiki/{角色名稱}/`）
- 技能組合（選填：連結至 Claude Code Skills）

### 3.2 角色定義範本

```yaml
# 角色定義（概念上存放於 WIKI.md 與 CLAUDE.md）
role:
  id: researcher
  display_name: 研究者（Researcher）
  trigger_keywords: [碩論, 語料庫, 文獻, APA, 期刊投稿]
  primary_outputs: [論文草稿 .docx, 文獻清單 APA 7th, 研究架構圖]
  raw_source_types: [PDF 論文, 語料庫資料, 研討會簡報]
  wiki_namespace: wiki/研究者/
  skills: [tw-research-proposal-diamond, tw-research-citation-checker]
  notion_workspace: 研究者工作區 Research
```

### 3.3 建議的初始角色集（6 個角色）

適用於管理多領域知識工作者：

| 角色 | 用途 | 典型 Wiki 頁面 |
|------|------|---------------|
| 主編（Editor） | 內容創作、社群媒體、出版 | 貼文草稿、風格指南、選題庫 |
| 研究者（Researcher） | 學術寫作、文獻回顧 | 論文摘要、合成頁面、引用筆記 |
| 教師（Teacher） | 課程設計、教案規劃 | 選文分析、教學框架、考試資源 |
| 開發者（Developer） | 程式專案、工具開發 | 專案筆記、API 文件、部署記錄 |
| 行政（Administrator） | 營運、財務、行政事務 | 流程、模板、聯絡清單 |
| 學習者（Learner） | 自主學習 | 概念地圖、學習計畫、進度記錄 |

**調整角色集**：從 2-3 個角色開始。只有在發現 10 頁以上不適合現有角色時才新增角色。若某命名空間在 3 個月後少於 5 頁，則考慮合併角色。

### 3.4 跨角色頁面

在以下情況建立跨角色頁面：
- 內容確實同等服務於 2 個以上角色
- 合成揭示了角色特定知識之間的連結
- 某概念是跨角色的共同詞彙

跨角色頁面存放於 `wiki/cross-role/`，並連結至角色特定頁面，而非取代它們。

---

## 4. 頁面 Schema

### 4.1 Frontmatter 規格

每個 Wiki 頁面必須以此 YAML frontmatter 開頭：

```yaml
---
title: "顯示語言的頁面標題"
role: researcher          # 必須是：editor|researcher|teacher|developer|admin|learner|cross-role 之一
type: source-summary      # 見 4.2 有效值
sources:                  # 此頁面合成的 raw/ 路徑清單
  - raw/researcher/paper-2024.pdf
  - raw/researcher/conference-notes.md
created: 2026-01-15       # ISO 日期，首次建立
updated: 2026-04-10       # ISO 日期，最後一次有意義的更新
links:                    # 相關 Wiki 頁面路徑清單（相對路徑）
  - wiki/researcher/related-concept.md
  - wiki/cross-role/synthesis-topic.md
tags: []                  # 選填：Obsidian 搜尋用的自由標籤
---
```

**欄位規則：**
- `role` — 從定義的角色集中恰好選一個值
- `type` — 從頁面類型分類法中恰好選一個值（見 4.2）
- `sources` — 若無原始素材（如合成頁面）則為空列表 `[]`
- `links` — 雙向原則：若 A 連結 B，B 也應連結 A
- `created`/`updated` — 永遠使用 ISO 8601 日期格式（YYYY-MM-DD）

### 4.2 頁面類型分類法

| 類型 | 使用時機 | 範例 |
|------|---------|------|
| `source-summary` | 摘要單一原始素材 | 論文摘要、文章摘要 |
| `entity` | 描述特定人物、文本、專案或工具 | 作者簡介、選文分析、專案筆記 |
| `concept` | 說明抽象概念或方法 | 研究方法論、教學框架 |
| `synthesis` | 將多個來源連結成新的洞察 | 文獻回顧段落、主題分析 |
| `profile` | 描述角色、人設或配置 | 教師簡介、系統總覽 |
| `index` | 為命名空間下的子項目建立目錄 | 教科書索引、專案清單 |

### 4.3 頁面內容結構

frontmatter 之後，使用以下結構（視需要調整）：

```markdown
## [主要概念 / 摘要]

一段執行摘要，說明本頁涵蓋的內容。

## [核心內容段落]

實質內容。適當使用標題、表格、清單。
優先使用結構化格式而非純散文，以便代理閱讀。

## 引用來源 / Sources

- [來源標題](../raw/path/to/file.pdf) — 簡短描述

## 相關連結 / Related

- [相關頁面](../wiki/role/page.md) — 說明關聯原因
```

---

## 5. 工作流程規格

### 5.1 `/wiki ingest` — 新增知識

**觸發時機**：使用者輸入 `/wiki ingest [路徑或角色名稱]`

**代理步驟**：

```
1. 讀取 wiki/index.md
   → 了解現有頁面清單以避免重複

2. 確認素材來源
   a. 若給定路徑：讀取該特定檔案
   b. 若給定角色：列出 raw/{角色}/ 中尚未收錄於索引的檔案
   c. 若未指定：列出所有 raw/ 子目錄中最新的未處理檔案

3. 讀取素材全文
   → 永遠不跳過此步驟；淺讀只會產出品質差的 Wiki 頁面

4. 分析與提取
   → 識別：主要主張、關鍵概念、重要實體、方法論、
     對其他角色的影響

5. 與用戶確認（選填，若有 `--auto` 旗標則跳過）
   → 「我發現 3 個關鍵主題：X、Y、Z。是否也要強調 [主題]？」

6. 撰寫 Wiki 頁面
   a. 建立 wiki/{角色}/{檔名}.md，包含完整 frontmatter
   b. 素材密集時：拆分成多頁（每個概念一頁）
   c. 素材單薄時：整合至現有相關頁面

7. 更新 wiki/index.md
   → 在正確的角色分區下新增頁面條目

8. 檢查跨角色連結
   → 此頁面是否與其他角色的頁面有關？若有，加入雙向連結

9. 追加 wiki/log.md
   → 格式：## [YYYY-MM-DD] ingest | {素材檔名} | {N 頁建立/更新}
```

**決策邏輯：建立新頁面 vs. 更新現有頁面**

| 條件 | 動作 |
|------|------|
| 新素材，無相關頁面存在 | 建立新頁面 |
| 新素材，存在密切相關頁面 | 更新現有頁面，記下新素材 |
| 重複素材（已 ingest） | 記錄日期，若內容未變則跳過 |
| 素材跨越多個不相關主題 | 拆分成 2 頁以上 |

### 5.2 `/wiki query` — 檢索知識

**觸發時機**：使用者輸入 `/wiki query [自然語言問題]`

**代理步驟**：

```
1. 讀取 wiki/index.md
   → 識別最可能回答查詢的 1-5 頁

2. 讀取相關頁面（最多 5 頁，優先選 synthesis 類型）
   → 優先順序：synthesis > source-summary > entity > concept

3. 合成答案
   → 以 [頁面標題](相對路徑.md) 格式引用來源
   → 標注缺口：「這在 Wiki 中尚未涵蓋——考慮 ingest X」

4. 判斷保存價值
   → 若答案以新方式結合了 3 頁以上的洞察：提議保存
   → 若答案只是簡單的資料檢索：不建立噪音

5. 若保存：建立跨角色或角色特定的合成頁面
   → 更新 index.md 和 log.md
```

**輸出格式選項**：
- 預設：含引用的 Markdown 回答
- `--marp`：Marp 投影片格式（適合報告用）
- `--apa`：APA 7th 參考文獻清單
- `--table`：摘要表格格式

### 5.3 `/wiki lint` — 健康檢查

**觸發時機**：使用者輸入 `/wiki lint`

**代理步驟**：

```
1. 讀取 wiki/index.md → 取得完整頁面清單

2. 對索引中的每個頁面：
   a. 驗證檔案存在於列出的路徑（捕捉移動/刪除的檔案）
   b. 讀取 frontmatter → 確認所有必填欄位存在
   c. 確認 links[] → 驗證每個目標路徑存在
   d. 確認 sources[] → 驗證每個 raw/ 路徑存在
   e. 標記空頁面（只有 frontmatter，無內容）

3. 掃描孤立頁面：
   → wiki/ 目錄中存在但未列於 index.md 的頁面

4. 標記過時內容：
   → updated 日期超過 6 個月且有來源有新版本的頁面

5. 回報：
   - ✅ 健康頁面：N 頁
   - ⚠️ 發現問題：[清單]
   - 🔗 死連結：[清單]
   - 📄 孤立頁面：[清單]
   - 💡 建議新頁面：[基於尚未 ingest 的來源檔案]

6. 追加 wiki/log.md：
   ## [YYYY-MM-DD] lint | {N 個問題} | {N 頁健康}
```

### 5.4 `/wiki status` — 統計儀表板

**觸發時機**：使用者輸入 `/wiki status`

**輸出格式**：

```
Wiki 統計（截至 YYYY-MM-DD）
─────────────────────────────
角色         Raw Sources    Wiki 頁面    最後 ingest
{角色-A}     {N} 個         {N} 頁       YYYY-MM-DD
{角色-B}     {N} 個         {N} 頁       YYYY-MM-DD
跨角色        —              {N} 頁       YYYY-MM-DD
─────────────────────────────
合計          {N} sources    {N} 頁
最近操作（最後 3 筆）：
  - [log 條目 1]
  - [log 條目 2]
  - [log 條目 3]
```

---

## 6. 知識圖譜整合（Graphify）

### 6.1 Graphify 的功能

Graphify 掃描專案中的所有檔案，透過 LLM 提取語意節點與邊，將其聚合成社群，並輸出：
- `graphify-out/GRAPH_REPORT.md` — 人類可讀的分析
- `graphify-out/graph.html` — 互動式視覺化
- `graphify-out/graph.json` — 機器可讀的圖譜

### 6.2 何時執行 `/graphify`

| 觸發條件 | 動作 |
|---------|------|
| 新 ingest 10 頁以上 | 執行完整 graphify 更新 |
| 建立新角色命名空間後 | 執行以觀察聚合情況 |
| 開始大型合成專案前 | 先讀取 GRAPH_REPORT.md |
| 每月例行維護 | 執行以偵測漂移與孤立頁面 |

### 6.3 閱讀 GRAPH_REPORT.md

**重點檢查段落**：

1. **God Nodes（高連接度節點）** — 連接數最高的節點 = 你的核心抽象
   - 若 Wiki 頁面是 god node：可能開發不足（太多東西指向它但它未展開）
   - 若原始素材是 god node：可能需要拆分 ingest 成多頁 Wiki

2. **社群（Communities）** — 偵測到的相關內容叢集
   - 健康：每個社群大致對應一個角色或子主題
   - 警告：若兩個角色在同一社群，檢查是否缺少邊界頁面
   - 警告：若社群只有 1 個成員，代表孤立——加入連結

3. **驚喜連結（Surprising Connections）** — 圖譜推斷出但你未明確建立的邊
   - 審查這些：部分是值得記錄於跨角色頁面的有效洞察
   - 其他是因相似術語造成的誤報——加入消歧義

4. **超邊（Hyperedges）** — 群組關係
   - 代表主題分組；驗證是否符合你的思維模型

### 6.4 增量更新

Graphify 使用檔案雜湊快取（`graphify-out/cache/`）。後續執行時：
- 未變更的檔案重用快取的提取結果（無 LLM 費用）
- 只有新增/修改的檔案才重新提取
- 每次均從完整合併資料集重建圖譜

**快取管理**：
- 除非想要完整重新提取，否則不要手動刪除快取檔案
- 若專案中某檔案被刪除，其快取條目會成為孤立 — 執行 `/graphify clean` 清除

---

## 7. 跨角色設計模式

### 7.1 橋接模式（Bridge Pattern）

當兩個角色共享某概念時，建立一個橋接它們的跨角色頁面：

```
wiki/researcher/semantic-similarity.md  →→ 連結至 →→
wiki/cross-role/semantic-similarity-bridge.md
   ←← 連結自 ←←  wiki/developer/embedding-search.md
```

橋接頁面解釋連結；角色特定頁面包含適合該角色深度的內容。

### 7.2 合成模式（Synthesis Pattern）

在 ingest 5 頁以上相關頁面後，建立合成頁面：

```yaml
type: synthesis
sources: []   # synthesis 從 wiki 頁面提取，而非原始素材
links:
  - wiki/researcher/paper-A.md
  - wiki/researcher/paper-B.md
  - wiki/cross-role/shared-concept.md
```

合成頁面是最高價值的產出：它們捕捉任何單一來源都不存在的洞察。

### 7.3 再利用機會模式（Reuse Opportunities Pattern）

維護 `wiki/cross-role/reuse-opportunities.md` 作為活文件，列出：
- 出現在 2 個以上角色但尚未橋接的概念
- 為某角色開發但可服務其他角色的工具
- 具有教學/開發意涵的研究發現

此文件既是跨角色合成，也是代理的待辦清單。

---

## 8. 同步架構

### 8.1 三層同步模型

```
第一層：Git（規範性真相）
   ↑↓ git push / git pull
第二層：Obsidian（本地人類編輯）
   ← obsidian-git 外掛定時自動 commit 並 push
第三層：Notion（任務/儀表板匯集點）
   ← Claude Code 搭配 Notion MCP 依需求建立摘要頁面
```

### 8.2 Git 設定

**建議的 `.gitignore`**：

```gitignore
# Obsidian 工作區檔案（機器特定）
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.trash/

# 可重新生成的產出物
artifacts/*.pdf
artifacts/*.docx
graphify-out/graph.html
graphify-out/.graphify_*.json

# 不同步的私人筆記
docs/private/
```

**分支策略**：單一 `main` 分支。所有代理的 commit 直接推送至 main。僅在實驗性重構時使用功能分支。

**Commit 訊息格式**：
```
{類型}: {簡短描述}

- {細節 1}
- {細節 2}

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

類型：`feat`（新頁面）、`update`（頁面更新）、`fix`（修正）、`chore`（維護）、`docs`（系統文件）

### 8.3 Obsidian 設定

**必要外掛**：
- `obsidian-git` — 啟動時自動 pull，每 N 分鐘自動 commit + push
- `Dataview`（選填）— 依 frontmatter 欄位查詢 Wiki 頁面

**Vault 設定**：
- 預設新檔案位置：`wiki/{目前角色}/`
- 模板資料夾：`docs/templates/`
- 附件資料夾：`raw/attachments/`

**圖形視圖設定**（用於發現）：
- 依 `role` frontmatter 欄位為節點上色
- 過濾排除 `raw/` 目錄
- 將標籤顯示為額外節點

### 8.4 Notion 同步（透過 MCP）

Notion 不是鏡像 — 它是每個角色工作區的**摘要儀表板**。

**同步模型**：
- 每次重大 ingest 事件對應一個 Notion 頁面（摘要 + GitHub 連結）
- Notion 資料庫儲存任務導向資料（非 Wiki 內容）
- 代理透過 `notion-create-pages` MCP 工具依需求建立 Notion 頁面
- 永遠不嘗試自動保持 Notion 同步 — 依需求建立摘要

---

## 9. 完整檔案範本

### 9.1 來源摘要頁面範本（source-summary）

```markdown
---
title: "{來源標題} — 摘要"
role: researcher
type: source-summary
sources:
  - raw/researcher/{檔名}.pdf
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
links: []
---

## 核心主張

{一句話：這個來源主張/展示了什麼？}

## 方法論

{如何進行研究 / 如何產生內容？}
- 方法一：{描述}
- 方法二：{描述}

## 關鍵發現

| 發現 | 細節 | 重要性 |
|------|------|--------|
| {發現 1} | {細節} | 高/中/低 |
| {發現 2} | {細節} | 高/中/低 |

## 與知識庫的連結

- 與 [{相關頁面}]({路徑}) 的關係：{關聯原因}

## 引用資訊

> {作者姓, 名. (年份). 標題. 期刊/出版社. DOI/URL}

## 代理筆記

{記錄哪些內容令人意外、哪些模糊、哪些需要後續跟進}
```

### 9.2 合成頁面範本（synthesis）

```markdown
---
title: "{合成主題} — 綜合分析"
role: cross-role
type: synthesis
sources: []
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
links:
  - wiki/{角色-A}/{頁面-A}.md
  - wiki/{角色-B}/{頁面-B}.md
---

## 綜合問題

> {以問題形式陳述本合成頁面回答的問題}

## 跨來源洞察

{綜合任何單一來源都不存在的洞察的段落。
這是合成頁面的獨特價值。}

## 支持證據

| 洞察 | 來源頁面 | 信心度 |
|------|---------|--------|
| {洞察 1} | [{頁面}]({路徑}) | 高/中 |
| {洞察 2} | [{頁面}]({路徑}) | 高/中 |

## 矛盾與張力

{來源之間在哪裡有分歧？存在哪些張力？}

## 行動建議

- 對 {角色-A}：{意涵}
- 對 {角色-B}：{意涵}

## 未解問題

- {待解問題 1}
- {待解問題 2}
```

### 9.3 索引頁面範本（適用於角色命名空間）

```markdown
---
title: "{角色} Wiki 索引"
role: {role}
type: index
sources: []
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
links: []
---

## 概覽

{2-3 句話描述此角色及其 Wiki 頁面涵蓋的內容}

**統計**：{N} 頁面 ｜ {N} 原始素材 ｜ 最後更新：{日期}

---

## 頁面清單

### {子分類 1}

| 頁面 | 類型 | 摘要 | 更新日 |
|------|------|------|--------|
| [{頁面標題}]({路徑}) | {類型} | {一行描述} | {日期} |

### {子分類 2}

| 頁面 | 類型 | 摘要 | 更新日 |
|------|------|------|--------|
| [{頁面標題}]({路徑}) | {類型} | {一行描述} | {日期} |

---

## 常見查詢

- **Q**：{關於此角色的常見問題}
  → 見：[{回答頁面}]({路徑})
```

### 9.4 日誌條目格式

追加至 `wiki/log.md` — 永遠不覆寫：

```markdown
## [YYYY-MM-DD] {操作} | {簡短描述}

- **操作**：{ingest | query | lint | graphify | update}
- **範圍**：{N 個檔案 / N 頁 / 角色名稱}
- **結果**：{N 頁建立，N 頁更新，N 個錯誤}
- **備註**：{任何值得注意的觀察}
```

---

## 10. 代理複製檢查清單

初次閱讀本文件的代理應依照以下步驟從零初始化系統：

### 第一階段：建立鷹架（執行一次）

```
□ 建立目錄結構：
  mkdir -p raw wiki/cross-role docs artifacts archive graphify-out

□ 建立 WIKI.md（複製本文件的 schema 章節）

□ 建立 wiki/index.md（含角色分區的空索引）

□ 建立 wiki/log.md（含初始化條目）

□ 初始化 git：
  git init && git add . && git commit -m "init: JDN LLM Wiki scaffold"

□ 設定 .gitignore（見第 8.2 節）

□ 在 repo 根目錄設定 Obsidian Vault（安裝 obsidian-git 外掛）

□ （選填）建立 GitHub 私有 repo 並推送
```

### 第二階段：首次 Ingest（每個角色執行一次）

```
□ 將來源素材放入 raw/{角色}/

□ 對每個來源執行 /wiki ingest

□ 每次 ingest 後：驗證 index.md 已更新、log.md 已追加

□ 10 頁以上後：執行 /graphify 觀察初始結構

□ 讀取 GRAPH_REPORT.md：記錄 god nodes 與社群結構
```

### 第三階段：維護（週期性執行）

```
□ 每週：/wiki status（確認頁面數量、近期活動）
□ 每月：/wiki lint（捕捉死連結、孤立頁面）
□ 每次 ingest：/wiki ingest [新素材]
□ 每 10 頁：/graphify（更新知識圖譜）
□ 每季：審查跨角色合成機會
```

### 第四階段：品質訊號

系統運作良好時：
- ✅ Graphify 顯示 3-6 個與你的角色對應的清晰社群
- ✅ 跨角色頁面連結到 3 個以上角色命名空間的內容
- ✅ 合成頁面數量超過來源摘要頁面（深度 > 廣度）
- ✅ lint 報告中無孤立頁面
- ✅ God nodes 是元概念，而非個別原始素材

系統需要關注時：
- ⚠️ 某角色命名空間佔全部頁面的 60% 以上（不平衡）
- ⚠️ Graphify 顯示 1 個巨大社群（缺少邊界）
- ⚠️ log.md 超過 30 天無條目（停滯）
- ⚠️ index.md 列出不存在的頁面（漂移）

---

## 附錄 A：WIKI.md 最小範本

repo 根目錄的 `WIKI.md` 應始終包含：

```markdown
# Wiki 系統 Schema

## 目錄結構
{針對此特定專案鏡像本文件第 2 節}

## 已定義角色
{列出每個角色，含觸發詞與命名空間}

## 操作指令
- /wiki ingest [路徑|角色] — 新增知識
- /wiki query [問題] — 檢索與合成
- /wiki lint — 健康檢查
- /wiki status — 統計資訊

## Frontmatter 必填欄位
title, role, type, sources, created, updated, links

## 重要規則
- raw/ 唯讀
- log.md 僅追加
- 每次 ingest 均更新 index.md
```

---

## 附錄 B：Skills 整合

若使用 Claude Code Skills（`.claude/skills/` 目錄），將 Skills 對應至角色：

| 角色 | 建議 Skills |
|------|------------|
| 主編（Editor） | tw-social-infocard, tw-journal-editor |
| 研究者（Researcher） | tw-research-proposal-diamond, tw-research-citation-checker |
| 教師（Teacher） | tw-edu-lesson-plan-108, tw-edu-exam-generator |
| 開發者（Developer） | tw-weekly-report |
| 所有角色 | wiki（本 skill）、graphify |

Skills 擴增了 Wiki 系統 — 它們提供角色特定的工作流程，**產生**內容後再 ingest 進入 Wiki。

---

*本文件自我描述且自給自足。*  
*閱讀本文件的代理無需進一步說明即可建立完全相同的系統。*  
*版本：1.0 ｜ 規格日期：2026-04-15 ｜ 語言版本：繁體中文*  
*方法論：角色型 LLM Wiki + Git + Obsidian + Notion + Graphify*
