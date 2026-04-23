# JDN_llm_wiki — LLM-Maintained Knowledge Base Architecture

> **Purpose**: This document is a system specification for AI agents.
> An agent (Claude Code, Codex, Antigravity, or equivalent) that reads this file in full
> can replicate the entire knowledge-base file management strategy from scratch,
> without any additional human explanation.
>
> **No personal data included.** All examples are structural/methodological.
> **Date of specification**: 2026-04-15 | **Spec version**: 1.0

---

## 1. System Philosophy

### 1.1 Core Principle

A "JDN LLM Wiki" is a **role-based, LLM-maintained knowledge base** where:

- **Humans** provide raw source materials (PDFs, notes, articles, data exports)
- **LLM agents** read those materials and synthesize structured wiki pages
- **Git** is the canonical source of truth and version history
- **Obsidian** provides human-readable local editing (via obsidian-git sync)
- **Notion** serves as task/output sink and summary dashboard (via MCP)
- **Knowledge graphs** (Graphify) reveal hidden structure across pages

### 1.2 Design Axioms

1. **Raw materials are read-only** — agents never modify source files
2. **Wiki pages are agent-maintained** — agents write, update, cross-link
3. **Every action is logged** — `wiki/log.md` is append-only
4. **Index stays current** — `wiki/index.md` reflects every page after ingest
5. **Structure before content** — frontmatter schema enforced on every page
6. **Roles partition the space** — each page belongs to exactly one role namespace

### 1.3 When to Use This Architecture

Use this system when you have:
- Diverse source materials that need synthesis, not just storage
- Multiple "hats" or workflows that share some knowledge but need separation
- A need for both human exploration (Obsidian) and agent retrieval (wiki pages)
- Long-term accumulation where discoverability degrades without structure

---

## 2. Directory Architecture

```
{PROJECT_ROOT}/           ← Git repo root (also Obsidian Vault root)
├── WIKI.md               ← Schema document (agents must read this first)
├── raw/                  ← Source materials (READ-ONLY, never modified)
│   ├── {role-A}/         ← Raw files organized by role
│   ├── {role-B}/
│   └── {role-N}/
├── wiki/                 ← LLM-maintained knowledge pages
│   ├── index.md          ← Master index (updated after every ingest)
│   ├── log.md            ← Append-only operation log
│   ├── overview.md       ← Cross-role high-level synthesis (optional)
│   ├── {Role-A}/         ← CJK or descriptive dir name per role
│   │   ├── {topic}.md
│   │   └── {subtopic}/
│   │       └── {item}.md
│   ├── {Role-B}/
│   └── cross-role/       ← Pages that span multiple roles
│       ├── reuse-opportunities.md
│       └── {synthesis-topic}.md
├── docs/                 ← Human-authored design documents (not agent-maintained)
├── artifacts/            ← Agent-generated outputs (PDFs, slides, docx)
└── archive/              ← Deprecated/superseded materials
```

### 2.1 Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Wiki page files | English kebab-case `.md` | `academic-profile.md` |
| Role directories | CJK or full descriptive names | `研究者/`, `Developer/` |
| Sub-topic dirs | Same as role dirs | `academic-base/`, `古文30篇/` |
| Raw source files | Preserve original filename | `paper-2024.pdf` |
| Cross-role pages | English kebab-case in `cross-role/` | `reuse-opportunities.md` |

### 2.2 What Belongs Where

| Content Type | Location | Writable by Agent? |
|-------------|----------|-------------------|
| Source PDFs, notes, exports | `raw/` | ❌ Never |
| Synthesized wiki pages | `wiki/` | ✅ Yes |
| Design decisions, specs | `docs/` | ❌ Human only |
| Generated outputs (docx, PDF) | `artifacts/` | ✅ Yes |

---

## 3. Role-Based Organization Model

### 3.1 What a Role Is

A **role** is a coherent persona with its own:
- Trigger vocabulary (keywords that activate the role)
- Primary outputs (what the role produces)
- Raw material types (what sources it consumes)
- Wiki namespace (`wiki/{RoleName}/`)
- Skill set (optional: linked to Claude Code Skills)

### 3.2 Role Definition Template

```yaml
# Role definition (stored conceptually in WIKI.md and CLAUDE.md)
role:
  id: researcher
  display_name: 研究者（Researcher）
  trigger_keywords: [碩論, 語料庫, 文獻, APA, 期刊投稿]
  primary_outputs: [論文草稿 .docx, 文獻清單 APA 7th, 研究架構圖]
  raw_source_types: [PDF papers, corpus data, conference slides]
  wiki_namespace: wiki/研究者/
  skills: [tw-research-proposal-diamond, tw-research-citation-checker]
  notion_workspace: 研究者工作區 Research
```

### 3.3 Recommended Starter Role Set (6 roles)

For a knowledge worker managing multiple domains:

| Role | Purpose | Typical Wiki Pages |
|------|---------|-------------------|
| Editor | Content creation, social media, publishing | Post drafts, style guides, topic databases |
| Researcher | Academic writing, literature review | Paper summaries, synthesis pages, citation notes |
| Teacher | Curriculum design, lesson planning | Text analysis, teaching frameworks, exam resources |
| Developer | Code projects, tool building | Project notes, API docs, deployment records |
| Administrator | Operations, finance, admin tasks | Procedures, templates, contact lists |
| Learner | Self-directed study | Concept maps, study plans, progress logs |

**Adapting the role set**: Start with 2-3 roles. Add roles only when you find yourself with >10 pages that don't fit existing roles. Merge roles if one namespace has <5 pages after 3 months.

### 3.4 Cross-Role Pages

Create a cross-role page when:
- Content genuinely serves 2+ roles equally
- A synthesis reveals connections between role-specific knowledge
- A concept is shared vocabulary across roles

Cross-role pages live in `wiki/cross-role/` and link TO role-specific pages, never replacing them.

---

## 4. Page Schema

### 4.1 Frontmatter Specification

Every wiki page MUST begin with this YAML frontmatter:

```yaml
---
title: "Page Title in Display Language"
role: researcher          # One of: editor|researcher|teacher|developer|admin|learner|cross-role
type: source-summary      # See 4.2 for valid values
sources:                  # List of raw/ paths this page synthesizes
  - raw/researcher/paper-2024.pdf
  - raw/researcher/conference-notes.md
created: 2026-01-15       # ISO date of first creation
updated: 2026-04-10       # ISO date of last meaningful update
links:                    # List of related wiki page paths (relative)
  - wiki/researcher/related-concept.md
  - wiki/cross-role/synthesis-topic.md
tags: []                  # Optional: freeform tags for Obsidian search
---
```

**Field rules:**
- `role` — exactly one value from the defined role set
- `type` — exactly one value from the type taxonomy (4.2)
- `sources` — empty list `[]` if no raw sources (e.g., synthesis pages)
- `links` — bidirectional: if A links B, B should link A
- `created`/`updated` — always ISO 8601 date (YYYY-MM-DD)

### 4.2 Page Type Taxonomy

| Type | When to Use | Example |
|------|------------|---------|
| `source-summary` | Summarizing a single raw source | Paper summary, article digest |
| `entity` | Describing a specific person, text, project, or tool | Author profile, text analysis, project note |
| `concept` | Explaining an abstract concept or method | Research methodology, teaching framework |
| `synthesis` | Connecting multiple sources into new insight | Literature review section, thematic analysis |
| `profile` | Describing a role, persona, or configuration | Teacher profile, system overview |
| `index` | Cataloguing sub-items in a namespace | Textbook index, project list |

### 4.3 Page Content Structure

After the frontmatter, use this structure (adapt as needed):

```markdown
## [Main Concept / Summary]

One-paragraph executive summary of what this page covers.

## [Core Content Section]

Substantive content. Use headers, tables, lists as appropriate.
Prefer structured over prose for agent readability.

## 引用來源 / Sources

- [Source Title](../raw/path/to/file.pdf) — brief description

## 相關連結 / Related

- [Related Page](../wiki/role/page.md) — why it's related
```

---

## 5. Workflow Specifications

### 5.1 `/wiki ingest` — Adding New Knowledge

**Trigger**: User says `/wiki ingest [path or role-name]`

**Agent steps**:

```
1. READ wiki/index.md
   → Understand current page inventory to avoid duplication

2. IDENTIFY source material
   a. If path given: read that specific file
   b. If role given: list raw/{role}/ for files not yet in index
   c. If nothing: list all raw/ subdirs for newest unprocessed files

3. READ source material (full text)
   → Never skip this step; shallow reading produces poor wiki pages

4. ANALYZE and EXTRACT
   → Identify: main claim, key concepts, notable entities, methodology,
     implications for other roles

5. CONFIRM with user (optional, skip if `--auto` flag)
   → "I found 3 key themes: X, Y, Z. Should I also emphasize [topic]?"

6. WRITE wiki page(s)
   a. Create wiki/{role}/{filename}.md with full frontmatter
   b. For dense sources: split into multiple pages (1 page per concept)
   c. For thin sources: consolidate into existing related page

7. UPDATE wiki/index.md
   → Add new page entry under correct role section

8. CHECK cross-role links
   → Does this page relate to pages in OTHER roles? If yes, add bidirectional links

9. APPEND wiki/log.md
   → Format: ## [YYYY-MM-DD] ingest | {source filename} | {N pages created/updated}
```

**Decision logic: create new vs. update existing**

| Condition | Action |
|-----------|--------|
| New source, no related pages exist | Create new page |
| New source, closely related page exists | Update existing page, note the new source |
| Duplicate source (already ingested) | Note the date, skip unless content changed |
| Source spans multiple unrelated topics | Split into 2+ pages |

### 5.2 `/wiki query` — Retrieving Knowledge

**Trigger**: User says `/wiki query [natural language question]`

**Agent steps**:

```
1. READ wiki/index.md
   → Identify 1-5 pages most likely to answer the query

2. READ relevant pages (max 5, prefer synthesis type)
   → Prioritize: synthesis > source-summary > entity > concept

3. SYNTHESIZE answer
   → Cite sources with [Page Title](relative/path.md) format
   → Note gaps: "This isn't covered in the wiki yet — consider ingesting X"

4. JUDGE value of saving
   → If the answer combines insights from 3+ pages in a new way: offer to save
   → If the answer is a simple retrieval: don't create noise

5. IF saving: create cross-role or role-specific synthesis page
   → Update index.md and log.md
```

**Output format options**:
- Default: Markdown answer with citations
- `--marp`: Marp slide format (for presentations)
- `--apa`: APA 7th reference list
- `--table`: Summary table format

### 5.3 `/wiki lint` — Health Check

**Trigger**: User says `/wiki lint`

**Agent steps**:

```
1. READ wiki/index.md → get full page list

2. FOR EACH page in index:
   a. Verify file exists at listed path (catch moved/deleted files)
   b. Read frontmatter → check all required fields present
   c. Check links[] → verify each target path exists
   d. Check sources[] → verify each raw/ path exists
   e. Flag empty pages (frontmatter only, no content)

3. SCAN for orphans:
   → Pages in wiki/ directory not listed in index.md

4. FLAG stale content:
   → Pages where updated date > 6 months ago that have sources with newer versions

5. REPORT:
   - ✅ Healthy pages: N
   - ⚠️ Issues found: [list]
   - 🔗 Dead links: [list]
   - 📄 Orphaned pages: [list]
   - 💡 Suggested new pages: [based on source files not yet ingested]

6. APPEND wiki/log.md:
   ## [YYYY-MM-DD] lint | {N issues found} | {N pages healthy}
```

### 5.4 `/wiki status` — Statistics Dashboard

**Trigger**: User says `/wiki status`

**Output format**:

```
Wiki 統計（截至 YYYY-MM-DD）
─────────────────────────────
角色         Raw Sources    Wiki 頁面    最後 ingest
{Role-A}     {N} 個         {N} 頁       YYYY-MM-DD
{Role-B}     {N} 個         {N} 頁       YYYY-MM-DD
跨角色        —              {N} 頁       YYYY-MM-DD
─────────────────────────────
合計          {N} sources    {N} 頁
最近操作（最後 3 筆）：
  - [log entry 1]
  - [log entry 2]
  - [log entry 3]
```

---

## 6. Knowledge Graph

Visual graph navigation is handled by **Obsidian's built-in graph view**. Graphify was removed on 2026-04-24 due to high token cost (230+ LLM calls per update). Use `/wiki lint` to catch orphaned pages and missing links instead.

---

## 7. Cross-Role Design Patterns

### 7.1 The Bridge Pattern

When two roles share a concept, create a cross-role page that bridges them:

```
wiki/researcher/semantic-similarity.md  →→ links to →→
wiki/cross-role/semantic-similarity-bridge.md
   ←← links from ←←  wiki/developer/embedding-search.md
```

The bridge page explains the connection; the role-specific pages contain role-appropriate depth.

### 7.2 The Synthesis Pattern

After ingesting 5+ related pages, create a synthesis page:

```yaml
type: synthesis
sources: []   # synthesis pulls from wiki pages, not raw sources
links:
  - wiki/researcher/paper-A.md
  - wiki/researcher/paper-B.md
  - wiki/cross-role/shared-concept.md
```

Synthesis pages are the highest-value outputs: they capture insights that don't exist in any single source.

### 7.3 The Reuse Opportunities Pattern

Maintain `wiki/cross-role/reuse-opportunities.md` as a living document listing:
- Concepts that appear in 2+ roles but aren't yet bridged
- Tools built for one role that could serve another
- Research findings that have teaching/development implications

This file is both a cross-role synthesis and a todo list for the agent.

---

## 8. Sync Architecture

### 8.1 Three-Layer Sync Model

```
Layer 1: Git (canonical truth)
   ↑↓ git push / git pull
Layer 2: Obsidian (local human editing)
   ← obsidian-git plugin auto-commits and pushes on schedule
Layer 3: Notion (task/dashboard sink)
   ← Claude Code with Notion MCP creates summary pages on demand
```

### 8.2 Git Configuration

**Recommended `.gitignore`**:

```gitignore
# Obsidian workspace files (machine-specific)
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.trash/

# Artifacts that regenerate
artifacts/*.pdf
artifacts/*.docx

# Private notes not for sync
docs/private/
```

**Branch strategy**: Single `main` branch. All agent commits go directly to main. Feature branches only for experimental restructuring.

**Commit message format**:
```
{type}: {brief description}

- {detail 1}
- {detail 2}

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

Types: `feat` (new pages), `update` (page updates), `fix` (corrections), `chore` (maintenance), `docs` (system docs)

### 8.3 Obsidian Configuration

**Required plugins**:
- `obsidian-git` — auto-pull on startup, auto-commit+push every N minutes
- `Dataview` (optional) — query wiki pages by frontmatter fields

**Vault settings**:
- Default new file location: `wiki/{active-role}/`
- Template folder: `docs/templates/`
- Attachment folder: `raw/attachments/`

**Graph view settings** (for discovery):
- Color nodes by `role` frontmatter field
- Filter to exclude `raw/` directory
- Show tags as additional nodes

### 8.4 Notion Sync (via MCP)

Notion is NOT a mirror — it's a **summary dashboard** for each role workspace.

**Sync model**:
- One Notion page per major ingest event (summary + link to GitHub)
- Notion databases store task-oriented data (not wiki content)
- Agent creates Notion pages on demand via `notion-create-pages` MCP tool
- Never try to keep Notion in sync automatically — create summaries on request

---

## 9. Complete File Templates

### 9.1 Source-Summary Page Template

```markdown
---
title: "{Source Title} — 摘要"
role: researcher
type: source-summary
sources:
  - raw/researcher/{filename}.pdf
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
links: []
---

## 核心主張

{One sentence: what does this source argue/show?}

## 方法論 / 方法

{How was the research conducted / how was the content produced?}
- Method 1: {description}
- Method 2: {description}

## 關鍵發現

| 發現 | 細節 | 重要性 |
|------|------|--------|
| {finding 1} | {detail} | 高/中/低 |
| {finding 2} | {detail} | 高/中/低 |

## 與知識庫的連結

- 與 [{related page}]({path}) 的關係：{why related}

## 引用資訊

> {Author Last, F. (Year). Title. Journal/Publisher. DOI/URL}

## 代理人筆記 / Agent Notes

{Notes on what was surprising, what was ambiguous, what needs follow-up}
```

### 9.2 Synthesis Page Template

```markdown
---
title: "{Synthesis Topic} — 綜合分析"
role: cross-role
type: synthesis
sources: []
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
links:
  - wiki/{role-A}/{page-A}.md
  - wiki/{role-B}/{page-B}.md
---

## 綜合問題

> {The question this synthesis answers, stated as a question}

## 跨來源洞察

{Paragraph synthesizing insights that don't exist in any single source.
This is the unique value of the synthesis page.}

## 支持證據

| 洞察 | 來源頁面 | 信心度 |
|------|---------|--------|
| {insight 1} | [{page}]({path}) | 高/中 |
| {insight 2} | [{page}]({path}) | 高/中 |

## 矛盾與張力

{Where do the sources disagree? What tensions exist?}

## 行動建議

- 對 {Role-A}：{implication}
- 對 {Role-B}：{implication}

## 未解問題

- {open question 1}
- {open question 2}
```

### 9.3 Index Page Template (for role namespaces)

```markdown
---
title: "{Role} Wiki 索引"
role: {role}
type: index
sources: []
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
links: []
---

## 概覽

{2-3 sentences describing this role and what its wiki pages cover}

**統計**：{N} 頁面 ｜ {N} 原始素材 ｜ 最後更新：{date}

---

## 頁面清單

### {Sub-category 1}

| 頁面 | 類型 | 摘要 | 更新日 |
|------|------|------|--------|
| [{page-title}]({path}) | {type} | {one-line desc} | {date} |

### {Sub-category 2}

| 頁面 | 類型 | 摘要 | 更新日 |
|------|------|------|--------|
| [{page-title}]({path}) | {type} | {one-line desc} | {date} |

---

## 常見查詢

- **Q**: {common question about this role}
  → 見：[{answer page}]({path})
```

### 9.4 Log Entry Format

Append to `wiki/log.md` — NEVER overwrite:

```markdown
## [YYYY-MM-DD] {operation} | {brief description}

- **操作**：{ingest | query | lint | update}
- **範圍**：{N files / N pages / role name}
- **結果**：{N pages created, N updated, N errors}
- **備註**：{any notable observations}
```

---

## 10. Agent Replication Checklist

An agent reading this document for the first time should follow these steps to initialize the system from scratch:

### Phase 1: Scaffold (do once)

```
□ Create directory structure:
  mkdir -p raw wiki/cross-role docs artifacts archive

□ Create WIKI.md (copy this document's schema sections)

□ Create wiki/index.md (empty index with role sections)

□ Create wiki/log.md (with initialization entry)

□ Initialize git:
  git init && git add . && git commit -m "init: JDN LLM Wiki scaffold"

□ Configure .gitignore (see Section 8.2)

□ Set up Obsidian vault at repo root (install obsidian-git plugin)

□ (Optional) Create GitHub private repo and push
```

### Phase 2: First Ingest (for each role)

```
□ Place source materials in raw/{role}/

□ Run /wiki ingest for each source

□ After each ingest: verify index.md updated, log.md appended
```

### Phase 3: Maintenance (recurring)

```
□ Weekly: /wiki status (check page counts, recent activity)
□ Monthly: /wiki lint (catch dead links, orphaned pages)
□ Per-ingest: /wiki ingest [new materials]
□ Per-quarter: Review cross-role synthesis opportunities
```

### Phase 4: Quality Signals

The system is working well when:
- ✅ Cross-role pages link to content in 3+ role namespaces
- ✅ Synthesis pages outnumber source-summary pages (depth > breadth)
- ✅ No orphaned pages in lint report
- ✅ God nodes are meta-concepts, not individual raw sources

The system needs attention when:
- ⚠️ One role namespace has >60% of all pages (imbalance)
- ⚠️ Graphify shows 1 giant community (missing boundaries)
- ⚠️ Log.md shows no entries in 30+ days (stale)
- ⚠️ Index.md has pages listed that don't exist (drift)

---

## Appendix A: WIKI.md Minimal Template

The `WIKI.md` at repo root should always contain:

```markdown
# Wiki System Schema

## Directory Structure
{mirror Section 2 of this document for this specific project}

## Roles Defined
{list each role with trigger keywords and namespace}

## Operations
- /wiki ingest [path|role] — add new knowledge
- /wiki query [question] — retrieve and synthesize
- /wiki lint — health check
- /wiki status — statistics

## Frontmatter Required Fields
title, role, type, sources, created, updated, links

## Important Rules
- raw/ is READ-ONLY
- log.md is APPEND-ONLY
- Every ingest updates index.md
```

---

## Appendix B: Skill Integration

If using Claude Code Skills (`.claude/skills/` directory), map skills to roles:

| Role | Recommended Skills |
|------|-------------------|
| Editor | tw-social-infocard, tw-journal-editor |
| Researcher | tw-research-proposal-diamond, tw-research-citation-checker |
| Teacher | tw-edu-lesson-plan-108, tw-edu-exam-generator |
| Developer | tw-weekly-report |
| All roles | wiki (this skill) |

Skills augment the wiki system — they provide role-specific workflows that PRODUCE content that then gets ingested into the wiki.

---

*This document is self-describing and self-sufficient.*
*An agent that reads it can build an identical system without further instruction.*
*Version: 1.0 | Specification date: 2026-04-15*
*Method: Role-based LLM wiki with Git + Obsidian + Notion*
