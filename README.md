# JDN LLM Wiki

A **role-based, LLM-maintained knowledge base** architecture specification — designed so AI agents (Claude Code, Codex, Antigravity, or any LLM agent) can read this document and replicate the entire file management strategy from scratch.

## Language versions / 語言版本

| Language | File |
|----------|------|
| 🇬🇧 English | [JDN_llm_wiki.md](JDN_llm_wiki.md) |
| 🇹🇼 繁體中文 | [JDN_llm_wiki_zh.md](JDN_llm_wiki_zh.md) |

## What's in this repo

| File | Description |
|------|-------------|
| [JDN_llm_wiki.md](JDN_llm_wiki.md) | Full architecture specification (English) |
| [JDN_llm_wiki_zh.md](JDN_llm_wiki_zh.md) | 完整架構規格說明（繁體中文） |

## What this spec covers

- **Directory architecture** — where raw sources, wiki pages, and outputs live
- **Role-based organization** — partitioning knowledge by persona/workflow
- **Page schema** — frontmatter spec, page types, naming conventions
- **Workflow specifications** — `/wiki ingest`, `/wiki query`, `/wiki lint`, `/wiki status`
- **Knowledge graph integration** — when and how to use Graphify
- **Cross-role design patterns** — bridge, synthesis, and reuse patterns
- **Sync architecture** — Git + Obsidian + Notion three-layer model
- **Complete file templates** — copy-paste ready for source-summary, synthesis, index, log entries
- **Agent replication checklist** — step-by-step to bootstrap the system from zero

## Who this is for

- AI agents initializing a new knowledge management system
- Developers building multi-role knowledge bases for LLM-assisted workflows
- Anyone who wants a structured, maintainable wiki that an LLM can autonomously manage

## Quick start (for AI agents)

1. Read `JDN_llm_wiki.md` (English) or `JDN_llm_wiki_zh.md` (繁體中文) in full
2. Follow **Section 10: Agent Replication Checklist**
3. Adapt the role definitions (Section 3) to your use case
4. Run your first `/wiki ingest` with Section 5.1

---

🤖 Spec authored with Claude Code | Version 1.0 | 2026-04-15
