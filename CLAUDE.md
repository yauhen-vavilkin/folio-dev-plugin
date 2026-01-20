# FOLIO Development Marketplace for Claude Code

> A single-plugin marketplace with skills and tools for documenting features and maintaining FOLIO microservices.

## Why This Exists

FOLIO is a large-scale microservices architecture with dozens of Java/Spring/Quarkus modules. Maintaining accurate documentation across these modules is challenging:

- **Documentation drift**: Code changes faster than documentation
- **Onboarding overhead**: New developers struggle to understand what features do and why they exist
- **AI context limitations**: AI agents need behavioral documentation to effectively modify and extend features
- **Inconsistent documentation**: Different teams use different formats and approaches

This plugin provides **document-driven development** tools that keep documentation synchronized with code.

## What We're Building

A **marketplace** containing plugins, skills, agents, and tools for FOLIO microservices development:

- **`folio-dev` plugin**: Contains the `/folio-dev:document-feature` skill for behavioral documentation
- **Future plugins**: Test generation, code review, and more can be added to this marketplace

### Philosophy: Behavioral Documentation

Following the research in `knowledge-base/DDD research.md`, we focus on **behavioral documentation**—capturing WHAT features do and WHY they exist, not HOW they're implemented.

## Project Structure

```
folio-dev-plugin/                    # Marketplace root
├── .claude-plugin/
│   └── marketplace.json             # Marketplace metadata
├── plugins/
│   └── folio-dev/                   # Individual plugin
│       ├── .claude-plugin/
│       │   └── plugin.json          # Plugin metadata
│       ├── skills/
│       │   └── document-feature/
│       │       └── SKILL.md         # Skill definition
│       └── README.md                # Plugin documentation
├── knowledge-base/                  # Research and best practices (shared across plugins)
│   └── DDD research.md
├── CLAUDE.md                        # This file
└── README.md                        # Marketplace overview
```

**Architecture**: This follows the **multi-plugin marketplace pattern** (like `anthropics/claude-plugins-official`), where:
- Plugins live in `plugins/plugin-name/` subdirectories
- Each plugin has its own `.claude-plugin/plugin.json`
- `marketplace.json` points to plugins via `source: "./plugins/plugin-name"`
- Allows adding more plugins in the future without restructuring

## Installation

1. Install the marketplace:
```bash
/plugin install https://github.com/yauhen-vavilkin/folio-dev-plugin
```

2. Install the plugin:
```bash
/plugin install folio-dev@folio-dev
```

3. Use the skill:
```bash
/folio-dev:document-feature
```

## Documentation Format

Feature documentation in FOLIO modules follows this structure:

```
docs/
├── features.md                      # Feature index (table of contents)
└── features/
    └── [feature-name].md            # Individual feature documentation (flat, no subfolders)
```

Each feature document includes:

- **What it does**: Observable behavior from external perspective
- **Why it exists**: Business rationale and problem solved
- **Endpoint(s)**: REST endpoints (if applicable - skip section if none)
- **Business rules**: Behavioral constraints
- **Caching**: Cache behavior, eviction triggers (if applicable)
- **Configuration**: Variables that control the feature (name only, defaults in README)
- **Dependencies**: Events published/consumed, service interactions

## Decisions Made (Session History)

### Frontmatter Decisions
- **No `owners` field**: Do not include in YAML frontmatter
- **No `created` field**: Only `updated` is needed
- **No `invariants` array**: Merge into business rules

### Content Decisions
- **No "Related decisions" / ADR sections**: We describe features only, not link to ADRs
- **No UI/consumer documentation**: Backend modules only, no UI references
- **No separate "Invariants" section**: Merged into "Business rules and constraints"
- **Configuration variables**: Describe what they control, but defaults live in README

### Skill Design Decisions
- **Generic patterns, not module-specific**: Use placeholders, not specific class names
- **Endpoint tracing**: When service changes, trace upward to find affected controllers
- **Handle non-endpoint features**: Event processors, scheduled jobs, internal utilities

### Architecture Decisions
- **Multi-plugin marketplace pattern**: Follows `anthropics/claude-plugins-official` structure
  - Plugins in `plugins/` subdirectories with individual `plugin.json` files
  - `marketplace.json` at root with `source: "./plugins/plugin-name"`
  - Allows adding future plugins (test generation, code review, etc.) without restructuring
  - Knowledge base shared across all plugins at root level

### Skill Version History
- **0.1.0**: Initial version
- **0.2.0**: Added endpoint discovery via flow tracing
- **0.3.0**: Made skill generic, removed module-specific details, added non-endpoint handling
- **0.4.0**: Feature-first identification, human intervention points for boundary clarification, flexible entry point templates

## Document-Feature Skill Workflow

1. **Analyze changes**: `git diff master...HEAD --stat`
2. **Identify entry points**: Detect REST, Kafka, Scheduled, Internal event patterns
3. **Clarify feature boundaries**: Ask user when changes touch multiple concerns
4. **Infer feature name**: Name based on behavior, not implementation
5. **Deep code inspection**:
   - Read changed files (services, caches, events, config)
   - **Verify entry points**: Find controllers/listeners using modified services
   - **Caching as aspect**: Document within feature, not as separate feature
6. **Interview user**: Ask about unclear business rationale, config choices, edge cases
7. **Generate documentation**: Write directly to `docs/features/[name].md` and update index

## Usage Example

```bash
# Install marketplace
/plugin install https://github.com/yauhen-vavilkin/folio-dev-plugin

# Install the folio-dev plugin from the marketplace
/plugin install folio-dev@folio-dev

# Implement feature on branch
git checkout -b feature/my-feature
# ... make changes ...

# Document the feature
/folio-dev:document-feature
```

## Open Questions

| Topic | Status | Notes |
|-------|--------|-------|
| Configuration defaults in docs | Resolved | Document in feature, reference README for module-wide defaults |
| Feature vs enhancement naming | Resolved | Ask user when unclear; enhancements update existing docs |
| Multiple features per commit | Open | Should skill split commits into multiple feature docs? |

## Development Roadmap

### Phase 1: Feature Documentation (Current)
- [x] `/document-feature` skill (v0.4.0)
- [x] Plugin structure
- [x] First tuning cycle (MODROLESKC-333 report)
- [ ] Test on more FOLIO modules
- [ ] Refine based on feedback

### Phase 2: Additional Skills
- [ ] `/write-tests` - Generate tests from feature documentation
- [ ] `/review-code` - Review changes against documented behavior

### Phase 3: Integration
- [ ] Pre-commit hooks to validate documentation
- [ ] CI checks for missing documentation

## References

- **Plugin Repository**: https://github.com/yauhen-vavilkin/folio-dev-plugin
- **Official Plugins**: https://github.com/anthropics/claude-plugins-official
- **Claude Code Docs**: https://code.claude.com/docs/en/plugins
- **FOLIO Developer Docs**: https://dev.folio.org/
