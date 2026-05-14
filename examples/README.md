# Examples

This directory contains sample files showing the content style used by `collaborative-llm-wiki`.

## Files

| File | What it shows |
|------|---------------|
| `claude-code-statusline-landscape.md` | Wiki article compiled from raw source material, with structured sections, tables, citations, and cross-references |
| `2026-03-19-claude-code-statusline-landscape.md` | Raw source material before compilation |
| `ai-coding-tools-index.md` | Topic-style index content with one-line summaries |
| `log-sample.md` | Collaboration-first `log.md` entries with Change-ID and PR review notes |

## Raw Source vs Wiki Article

**Raw source** (`2026-03-19-claude-code-statusline-landscape.md`):
- Original research notes
- Unstructured content
- Metadata header with Source, Collected, and Published dates

**Wiki article compiled from source** (`claude-code-statusline-landscape.md`):
- Structured sections
- Synthesized tables
- Cross-references to related wiki articles
- Source-attributed claims

## Operation Log

The canonical wiki log is `<wiki-root>/log.md`. Each PR appends a structured entry with:

- `Change-ID`
- operation type
- raw sources
- touched articles
- index updates
- decisions
- follow-up items
- review notes
