# FOLIO Development Plugin for Claude Code

## Why This Exists

FOLIO is a large-scale microservices architecture with dozens of Java/Spring/Quarkus modules. Maintaining accurate documentation across these modules is challenging:

- **Documentation drift**: Code changes faster than documentation
- **Onboarding overhead**: New developers struggle to understand what features do and why they exist
- **AI context limitations**: AI agents need behavioral documentation to effectively modify and extend features
- **Inconsistent documentation**: Different teams use different formats and approaches

This plugin addresses these challenges by providing **document-driven development** tools that keep documentation synchronized with code.

## What We're Building

A Claude Code plugin containing skills, agents, and tools specifically designed for FOLIO microservices development:

- **`/document-feature`** skill: Analyzes code changes and generates behavioral feature documentation
- **Future skills**: Test generation, code review, ADR creation, and more

### Philosophy: Behavioral Documentation

Following the research in `knowledge-base/DDD research.md`, we focus on **behavioral documentation**—capturing WHAT features do and WHY they exist, not HOW they're implemented. This approach:

- Helps AI agents understand features without reading implementation code
- Provides onboarding material for new developers
- Documents business rules and invariants explicitly
- Keeps documentation stable even when implementation changes

## Project Structure

```
folio-dev-plugin/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── knowledge-base/              # Research and best practices
│   └── DDD research.md          # Document-driven development research
├── skills/                      # Skill definitions
│   └── document-feature/
│       └── SKILL.md             # Feature documentation skill
├── agents/                      # (Future) Agent definitions
├── commands/                    # (Future) Slash commands
├── claude-code.md               # This file
└── README.md                    # Plugin overview
```

## Documentation Format

Feature documentation follows this structure:

```
docs/
├── features.md                  # Feature index (table of contents)
└── features/
    └── [feature-name].md        # Individual feature documentation
```

Each feature document includes:

- **What it does**: Observable behavior from external perspective
- **Why it exists**: Business rationale and problem solved
- **Endpoint(s)**: REST endpoints exposed by the feature
- **Business rules**: Behavioral invariants and constraints
- **Caching**: Cache behavior, eviction triggers, configuration
- **Configuration**: Variables that control the feature
- **Dependencies**: Interactions with other services/modules

## Approach

### 1. Manual Invocation, Smart Automation

The `/document-feature` skill is invoked manually by developers after implementing a feature. This ensures:

- Documentation is only written for meaningful changes
- The developer can provide context during the interview phase
- The skill doesn't accidentally document trivial fixes

### 2. Interview-First Documentation

The skill recognizes when code alone is insufficient and asks the developer for:

- Business rationale not visible in code
- Configuration choices and tradeoffs
- External consumers and cross-service interactions
- Edge cases and special behaviors

### 3. Module-Scoped Skills

Each FOLIO module installs its own copy of the skill, allowing for:

- Module-specific customization
- Independent documentation maintenance
- Clear ownership boundaries

### 4. Behavioral Focus

Documentation describes contracts and behavior, not implementation:

- **DO**: "Cache entries are evicted when user permissions change"
- **DON'T**: "Cache eviction is handled by UserPermissionCacheEvictor using Caffeine"

## Usage

### Installing the Skill

Copy the skill to each module:

```bash
# In each FOLIO module
mkdir -p .claude/skills
cp /path/to/folio-dev-plugin/skills/document-feature/SKILL.md \
   .claude/skills/document-feature.md
```

### Running the Skill

After implementing a feature:

```bash
# From the feature branch
/document-feature
```

The skill will:

1. Analyze `git diff master...HEAD`
2. Propose a feature name for confirmation
3. Inspect code for endpoints, caching, events, configs
4. Interview you about unclear aspects
5. Generate `docs/features/[feature-name].md`
6. Update `docs/features.md` index

## Development Roadmap

### Phase 1: Feature Documentation (Current)
- [x] `/document-feature` skill
- [ ] Test on real FOLIO modules
- [ ] Refine based on feedback

### Phase 2: Additional Skills
- [ ] `/write-tests` - Generate tests from feature documentation
- [ ] `/review-code` - Review changes against documented behavior
- [ ] `/create-adr` - Create Architecture Decision Records

### Phase 3: Integration
- [ ] Pre-commit hooks to validate documentation
- [ ] CI checks for missing documentation
- [ ] Documentation sync detection

## Research Foundation

This plugin is built on research into:

- **README-driven development** (Tom Preston-Werner, 2010)
- **Architecture Decision Records** (Michael Nygard, 2011)
- **Living Documentation** (Cyrille Martraire)
- **Docs-as-code** methodology
- **AI-parseable formats** (Markdown + YAML frontmatter)
- **FOLIO's microservices architecture** (Java/Spring/Quarkus)

See `knowledge-base/DDD research.md` for full research details.

## Contributing

When adding new skills or agents:

1. Follow the plugin structure conventions
2. Document in this file why the skill exists
3. Add examples to the knowledge base
4. Test on real FOLIO modules before committing

## References

- [Official Claude Code Plugins](https://github.com/anthropics/claude-plugins-official)
- [Claude Code Documentation](https://code.claude.com/docs/en/plugins)
- [FOLIO Developer Documentation](https://dev.folio.org/)
