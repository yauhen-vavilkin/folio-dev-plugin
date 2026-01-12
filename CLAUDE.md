# FOLIO Development Marketplace for Claude Code

> A marketplace of plugins, skills, and tools for documenting features and maintaining FOLIO microservices.

## Why This Exists

FOLIO is a large-scale microservices architecture with dozens of Java/Spring/Quarkus modules. Maintaining accurate documentation across these modules is challenging:

- **Documentation drift**: Code changes faster than documentation
- **Onboarding overhead**: New developers struggle to understand what features do and why they exist
- **AI context limitations**: AI agents need behavioral documentation to effectively modify and extend features
- **Inconsistent documentation**: Different teams use different formats and approaches

This marketplace provides **document-driven development** tools that keep documentation synchronized with code.

## What We're Building

A **marketplace** (collection of plugins) containing skills, agents, and tools for FOLIO microservices development:

- **`folio-dev` plugin**: Contains the `/document-feature` skill
- **Future**: Test generation, code review, and more skills

### Philosophy: Behavioral Documentation

Following the research in `knowledge-base/DDD research.md`, we focus on **behavioral documentation**—capturing WHAT features do and WHY they exist, not HOW they're implemented.

## Project Structure (Marketplace)

```
folio-dev-plugin/                    # Marketplace root
├── .claude-plugin/
│   └── marketplace.json             # Marketplace configuration
├── plugins/
│   └── folio-dev/                   # Individual plugin
│       ├── .claude-plugin/
│       │   └── plugin.json          # Plugin metadata
│       ├── skills/
│       │   └── document-feature/
│       │       └── SKILL.md         # Skill definition
│       └── README.md                # Plugin documentation
├── knowledge-base/                  # Research and best practices
│   └── DDD research.md
├── CLAUDE.md                        # This file
└── README.md                        # Marketplace overview
```

**Key convention**: `marketplace.json` must be inside `.claude-plugin/`, not at root.

## Installation

```bash
/plugin install https://github.com/yauhen-vavilkin/folio-dev-plugin
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

### Skill Version History
- **0.1.0**: Initial version
- **0.2.0**: Added endpoint discovery via flow tracing
- **0.3.0**: Made skill generic, removed module-specific details, added non-endpoint handling

## Document-Feature Skill Workflow

1. **Analyze changes**: `git diff master...HEAD --stat`
2. **Infer feature name**: Propose options, ask user to confirm
3. **Deep code inspection**:
   - Read changed files (services, caches, events, config)
   - **Trace upward**: Find controllers using modified services (`grep -r "ServiceName" src/main/java --include="*Controller.java"`)
   - **Conditional endpoints**: Include section only if endpoints found
4. **Interview user**: Ask about unclear business rationale, config choices, edge cases
5. **Generate documentation**: Write directly to `docs/features/[name].md` and update index

## Usage Example

```bash
# Install marketplace
/plugin install https://github.com/yauhen-vavilkin/folio-dev-plugin

# Implement feature on branch
git checkout -b feature/my-feature
# ... make changes ...

# Document the feature
/document-feature
```

## Open Questions

| Topic | Status | Notes |
|-------|--------|-------|
| Configuration defaults in docs | Discuss | Should we include defaults or just reference README? |
| Feature vs enhancement naming | Open | How to name when enhancing existing vs creating new? |

## Development Roadmap

### Phase 1: Feature Documentation (Current)
- [x] `/document-feature` skill (v0.3.0)
- [x] Marketplace structure
- [ ] Test on more FOLIO modules
- [ ] Refine based on feedback

### Phase 2: Additional Skills
- [ ] `/write-tests` - Generate tests from feature documentation
- [ ] `/review-code` - Review changes against documented behavior

### Phase 3: Integration
- [ ] Pre-commit hooks to validate documentation
- [ ] CI checks for missing documentation

## References

- **Marketplace Repository**: https://github.com/yauhen-vavilkin/folio-dev-plugin
- **Official Plugins**: https://github.com/anthropics/claude-plugins-official
- **Claude Code Docs**: https://code.claude.com/docs/en/plugins
- **FOLIO Developer Docs**: https://dev.folio.org/
