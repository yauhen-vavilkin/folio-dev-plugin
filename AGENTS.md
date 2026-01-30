# Agent Working Agreement

This repository is a Claude Code marketplace + plugin definition (mostly Markdown and JSON). There is no application runtime here; the main “deliverable” is clear, consistent documentation and valid plugin metadata.

Primary project context lives in `CLAUDE.md` (philosophy, structure, and documentation conventions). If `AGENTS.md` and `CLAUDE.md` conflict, follow `CLAUDE.md`.

## Repository Map (Where Changes Usually Go)

- `CLAUDE.md`: project overview + decisions made (treat as source of truth).
- `.claude-plugin/marketplace.json`: marketplace metadata (points to plugins under `plugins/`).
- `plugins/folio-dev/.claude-plugin/plugin.json`: plugin metadata.
- `plugins/folio-dev/skills/*/SKILL.md`: skill definitions (behavior + workflow).
- `knowledge-base/*.md`: research and best practices (supporting material).
- `reports/*.md`: example reports and writeups (not executable, don’t treat as tests).

Notes:
- `docs/` is currently empty in this repo; templates inside skills refer to `docs/features/*` in *target* FOLIO module repos.
- `.claude/` is local session data and is gitignored; don’t depend on it.

## Build / Lint / Test Commands

There is no build system, dependency manager, or unit test suite in this repo today. Use validation checks as your “tests”.

### Validate JSON (Recommended)

Validate a single JSON file:

```bash
python -m json.tool .claude-plugin/marketplace.json > /dev/null
```

Validate all plugin JSON files:

```bash
python -m json.tool .claude-plugin/marketplace.json > /dev/null
python -m json.tool plugins/folio-dev/.claude-plugin/plugin.json > /dev/null
```

If you have `jq` installed:

```bash
jq . .claude-plugin/marketplace.json > /dev/null
jq . plugins/folio-dev/.claude-plugin/plugin.json > /dev/null
```

### Markdown Linting (Optional)

This repo does not vendor a Markdown linter. If you’re comfortable using ephemeral `npx` tooling:

```bash
npx --yes markdownlint-cli "**/*.md"
```

### Formatting (Optional)

No formatter is pinned in-repo. If you use ephemeral `npx`:

```bash
npx --yes prettier --check "**/*.{md,json}"
```

### “Run a Single Test” (Repo Equivalent)

When a workflow calls for “run one test”, do one targeted validation:

- JSON: `python -m json.tool path/to/file.json > /dev/null`
- Markdown (optional): `npx --yes markdownlint-cli path/to/file.md`

### Manual Smoke Test in Claude Code (When Releasing)

These are Claude Code slash commands (not shell commands):

```text
/plugin install https://github.com/yauhen-vavilkin/folio-dev-plugin
/plugin install folio-dev@folio-dev
/folio-dev:document-feature
```

## Code Style Guidelines (Markdown + JSON)

### General

- Keep changes minimal and intentional; this repo is a reference artifact for other agents.
- Prefer ASCII; avoid adding new tooling/config unless it’s clearly justified.
- Don’t add secrets, tokens, or environment-specific paths.

### JSON Style (`.claude-plugin/*.json`)

- Indentation: 2 spaces; UTF-8; newline at EOF.
- Strings: double quotes; no trailing commas.
- Keep key ordering stable to reduce noisy diffs (match existing order in file).
- Preserve schema fields (e.g., `$schema` in `.claude-plugin/marketplace.json`).
- Versioning: if you bump plugin versions, keep versions aligned across relevant metadata.

### Markdown Style (`*.md`)

- Use ATX headings (`#`, `##`, `###`) with a blank line after headings.
- Prefer fenced code blocks and include a language hint (`bash`, `json`, `markdown`, `text`).
- Keep lines reasonably wrapped (~100 chars) when it improves readability; don’t reflow large blocks just to wrap.
- Tables: use tables when they improve scanability; keep them small and stable (avoid reformat-only diffs).
- Use backticks for file paths, commands, and identifiers.

### YAML Frontmatter (Skill Docs)

`plugins/folio-dev/skills/*/SKILL.md` begins with YAML frontmatter:

- Keep frontmatter at the very top and delimited by `---`.
- Don’t introduce new frontmatter keys without a strong reason.
- Follow the existing decisions called out in `CLAUDE.md` (e.g., avoid adding an `owners` field in feature templates).

### Naming Conventions

- Directory and file names: kebab-case (`plugins/folio-dev/skills/document-feature/`).
- Human titles: Title Case in headings; keep consistent with existing docs.
- When describing feature docs in target FOLIO repos, keep the convention: `docs/features/<kebab-case>.md`.

### Writing “Agentic” Instructions (SKILL.md)

- Write behavior-first instructions: what to do and why, not implementation trivia.
- Make workflows deterministic: numbered steps, clear entry points, explicit output paths.
- Clarifying questions:
  - Ask only when blocked by ambiguity that changes output materially.
  - Ask exactly one targeted question, and propose a default.
- Error handling / fallbacks:
  - If a referenced file doesn’t exist in the current repo, say so and continue with the nearest applicable action.
  - Prefer read-only validation (JSON parse, lint) over speculative “fixes”.

### If You Add Code (Scripts / Tools)

This repo currently contains no executable source code. If you add a small script/tooling file, keep it boring and easy to maintain:

- Prefer a single, self-contained script; avoid adding a dependency stack unless it’s clearly worth it.
- Imports (where applicable): group standard library, third-party, then local; sort within groups; remove unused.
- Types: keep public surfaces typed (avoid `any`/implicit `object`); prefer explicit return types for key functions.
- Naming: favor clear, domain-ish names (match repo vocabulary: plugin, marketplace, skill, feature docs).
- Error handling: fail fast with a non-zero exit code; print actionable messages; don’t swallow exceptions.
- Documentation: add a short “How to run” snippet to `README.md` or the relevant `SKILL.md`.

## Cursor / Copilot Rules

- No `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md` are present in this repo currently.
- Treat `CLAUDE.md` + `AGENTS.md` as the operational ruleset for agents.

## Change Checklist

- Update the correct file (marketplace vs plugin vs skill vs knowledge base).
- Validate JSON after edits (`python -m json.tool ...`).
- Re-read edited Markdown sections to ensure they stay consistent with `CLAUDE.md` decisions.
- Keep diffs small: avoid whitespace-only churn unless it fixes clarity.
