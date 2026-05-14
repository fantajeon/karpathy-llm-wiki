# LLM Wiki Config

> Schema: collaborative-llm-wiki/v1
> Wiki-Root: {repository-relative path, e.g. wiki/ or llm/wiki/}
> Created: {YYYY-MM-DD}

This file marks the configured wiki root. The skill should treat the directory containing this file as the wiki root after validating that `raw/`, `articles/`, `index.md`, and `log.md` are present.

Do not use this file to configure application behavior. It is only for wiki root discovery and PR-safe boundary checks.
