# karpathy-llm-wiki

**A reusable Agent Skill for maintaining a shared GitHub-backed LLM wiki through small, reviewable pull requests.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/Astro-Han/karpathy-llm-wiki?style=social)](https://github.com/Astro-Han/karpathy-llm-wiki)
[![GitHub forks](https://img.shields.io/github/forks/Astro-Han/karpathy-llm-wiki?style=social)](https://github.com/Astro-Han/karpathy-llm-wiki)
[![Agent Skills](https://img.shields.io/badge/Agent_Skills-compatible-blue)](https://agentskills.io)
[![Install](https://img.shields.io/badge/Install-npx_add--skill-green)](https://github.com/Astro-Han/karpathy-llm-wiki#install)

<p align="center">
  <img src="assets/karpathy-tweet.png" alt="Karpathy's tweet about LLM Wiki" width="560">
</p>

`collaborative-llm-wiki` packages [Karpathy's LLM Wiki idea](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) into a collaboration-first skill. It helps an agent ingest sources, compile them into wiki articles, answer questions from the wiki, lint wiki health, and prepare PR-safe changes for a maintainer to review and merge.

Unlike a personal Karpathy-style LLM wiki, this project is designed for a collaborative GitHub repository. Its main purpose is to make LLM-maintained knowledge changes safe for teams: configurable wiki roots, wiki-root-only edits, PR-lint, append-only operation logs, commit traceability, and maintainer-reviewed merges.

## What Is an LLM Wiki?

An **LLM wiki** is a knowledge system where source material is compiled into wiki articles instead of being re-summarized from raw documents on every query. New sources improve the wiki over time through article updates, citations, conflict notes, and follow-up items.

This skill is designed for shared GitHub repositories:

| Operation | What it does | PR behavior |
|-----------|--------------|-------------|
| **Ingest** | Stores one source in raw form and updates the primary article | Small wiki-root-only PR |
| **Query** | Answers from existing wiki articles with citations | Read-only unless archived |
| **Archive / Memory Bank** | Saves a final answer, correction, decision, or reusable note as a point-in-time wiki article | New memory article plus log entry |
| **Lint** | Checks links, index entries, duplicates, and PR readiness | Small auto-fixes or report-only findings |
| **Compact** | Summarizes old log entries and links them to commits | User-requested PR-safe log maintenance |

See [SKILL.md](SKILL.md) for the full workflow specification.

## Wiki Root

The wiki lives under a configured wiki root. The default is `wiki/`; teams can use `llm/wiki/` or another path when they want the wiki isolated from application code.

```text
your-repo/
└── wiki/                    # or llm/wiki/
    ├── llm_wiki_config.md   # root marker and schema
    ├── raw/                 # immutable source material
    │   └── topic/
    │       └── 2026-05-14-source-title.md
    ├── articles/            # wiki articles compiled from raw sources
    │   └── topic/
    │       └── concept-name.md
    ├── index.md             # global index
    ├── log.md               # append-only operation log
    └── references/          # templates copied into the wiki project
```

All write operations stay under the configured wiki root. The skill does not modify app code, source directories, package manifests, or CI configuration as part of wiki maintenance.

The root is set during initialization. If you want `llm/wiki/`, name it explicitly in the first setup request:

> "Set up a shared LLM wiki under `llm/wiki/`."

That creates `llm/wiki/llm_wiki_config.md`:

```md
# LLM Wiki Config

> Schema: collaborative-llm-wiki/v1
> Wiki-Root: llm/wiki/
> Created: 2026-05-14
```

If the first setup request does not name a root, the agent uses the default `wiki/`. It cannot discover `llm/wiki/` before that directory and `llm_wiki_config.md` exist.

On later tasks, the agent discovers the root by searching for a valid `llm_wiki_config.md` plus the required sibling files: `raw/`, `articles/`, `index.md`, and `log.md`. If more than one valid wiki root exists, the agent stops and asks which one to use instead of guessing.

## PR Workflows

### CI PR Workflow

Use this when the team can safely provide the required API key to CI.

1. CI receives a wiki task such as ingest, archive, lint, or compact.
2. CI runs the agent against the configured wiki root.
3. CI creates a PR containing only wiki-root changes.
4. PR-lint verifies path boundaries, append-only log behavior, `Change-ID`, and diff/log consistency.
5. A maintainer reviews and merges.

### Local PR Workflow

Use this when API keys cannot be placed in CI.

1. A contributor runs the agent locally.
2. The agent prepares wiki-root-only changes.
3. The contributor opens a PR.
4. PR-lint checks the same boundaries and log requirements.
5. A maintainer reviews and merges.

Both workflows produce the same files and log format. The skill prepares changes; it does not merge them.

## Log And Commit Traceability

`<wiki-root>/log.md` is the single canonical operation log. Each PR appends one structured entry with a stable `Change-ID`.

Commit messages should include trailers:

```text
Wiki-Log: <wiki-root>/log.md
Wiki-Change-ID: chg-YYYYMMDD-NNN
```

The log entry itself uses `Commit: linked by commit trailer` because a commit cannot contain its own final hash. During log compact, the skill can look up commit history and add related commit IDs to the compact summary.

## Quick Start

### 1. Install the skill

```bash
npx add-skill Astro-Han/karpathy-llm-wiki
```

Works with tools that support the [Agent Skills](https://agentskills.io) standard.

### 2. Initialize a shared wiki

Ask the agent to set up the wiki root:

> "Set up a shared LLM wiki under `llm/wiki/`."

The agent creates the wiki structure, index, log, and template copies under that root.

### 3. Ingest a source

> "Ingest this article into the shared wiki and keep the PR small: https://example.com/attention-is-all-you-need"

The agent checks for duplicate raw sources, stores the source, updates the primary article, touches the index entry, and appends a structured log entry.

### 4. Run PR-lint

> "Run PR-lint and verify all changed files are under `llm/wiki/`."

PR-lint reports blocking issues instead of auto-fixing merge-readiness problems.

## Tool Compatibility

This skill follows the [agentskills.io](https://agentskills.io) open standard:

| Tool | Install method |
|------|----------------|
| Claude Code | `npx add-skill Astro-Han/karpathy-llm-wiki` |
| Cursor | `npx add-skill Astro-Han/karpathy-llm-wiki` |
| Codex CLI | Copy to `.agents/skills/karpathy-llm-wiki/` |
| OpenCode | `npx add-skill Astro-Han/karpathy-llm-wiki` |
| Other tools | Copy `SKILL.md` and `references/` into the tool's skill directory |

## FAQ

### Why support both CI and local PR workflows?

Some teams can store API keys in CI and let automation open wiki PRs. Other teams must keep API keys local. This skill supports both while preserving the same rule: wiki work changes only the configured wiki root.

### What sources can be ingested?

Web pages, papers, blog posts, PDFs, markdown files, text files, and pasted text. The skill converts source material into markdown under `<wiki-root>/raw/` and compiles it into wiki articles under `<wiki-root>/articles/`.

### What happens when one source affects many topics?

The current PR updates the primary article and records broader work in `Follow-up Required`. This keeps PRs reviewable and avoids broad cross-topic edits.

### Is Archive a memory bank?

Yes. Archive saves conversation-derived knowledge: final answers, user-corrected conclusions, decisions, operating rules, reusable prompts, and notes for future answers. Archive pages are marked with `[Memory]` in the index and do not create raw source files.

### How are old log entries handled?

Log compact runs only when requested. It summarizes selected detailed entries, connects them to commit IDs, and marks retention status. It does not delete detailed entries unless the user explicitly approves specific `Change-ID` values.

## Inspired By

Unofficial community implementation of the workflow from [Karpathy's LLM Wiki idea](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). The value here is the reusable workflow, prompt structure, and collaboration-safe knowledge compilation rules.

See also: [lucasastorian/llmwiki](https://github.com/lucasastorian/llmwiki), [atomicmemory/llm-wiki-compiler](https://github.com/atomicmemory/llm-wiki-compiler).

## License

[MIT](LICENSE)
