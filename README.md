# FOLIO Development Plugin for Claude Code

> Skills, agents, and tools for documenting features and maintaining FOLIO microservices with behavioral documentation.

## Overview

FOLIO is a large-scale microservices architecture. Keeping documentation synchronized with code is challenging—this plugin provides tools for **document-driven development** that keep documentation accurate and useful for both humans and AI agents.

## What's Included

### Skills

| Skill | Description | Invocation |
|-------|-------------|------------|
| `document-feature` | Analyzes code changes and generates behavioral feature documentation | `/document-feature` |

## Quick Start

### Installation

1. Copy the skill to your FOLIO module:

```bash
mkdir -p .claude/skills
cp /path/to/folio-dev-plugin/skills/document-feature/SKILL.md \
   .claude/skills/document-feature.md
```

2. Ensure your `docs/` directory exists:

```bash
mkdir -p docs/features
```

### Usage

After implementing a feature on a branch:

```bash
/document-feature
```

The skill will:
1. Analyze `git diff master...HEAD`
2. Propose a feature name for your confirmation
3. Inspect code for endpoints, caching, events, and configuration
4. Interview you about unclear business rationale
5. Generate `docs/features/[feature-name].md`
6. Update `docs/features.md` index

## Documentation Format

Feature documentation uses **behavioral description**—focusing on WHAT features do and WHY they exist, not HOW they're implemented.

```markdown
---
feature_id: user-permissions-cache
title: User Permissions Cache
status: active
updated: 2025-01-12
---

# User Permissions Cache

## What it does
Caches user permission lookups to reduce database load during permission checks.

## Why it exists
Direct database queries for user permissions were creating performance bottlenecks...
```

## Project Structure

```
folio-dev-plugin/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata
├── knowledge-base/              # Research and best practices
│   └── DDD research.md
├── skills/                      # Skill definitions
│   └── document-feature/
│       └── SKILL.md
├── claude-code.md               # Project overview and philosophy
└── README.md                    # This file
```

## Philosophy

See [claude-code.md](claude-code.md) for:
- Why this plugin exists
- What we're building
- Development roadmap
- Research foundation

## Contributing

1. Follow the [Claude Code Plugin](https://github.com/anthropics/claude-plugins-official) conventions
2. Document in `claude-code.md` when adding new skills
3. Add research and examples to `knowledge-base/`
4. Test on real FOLIO modules before committing

## License

Apache-2.0

## References

- [Official Claude Code Plugins](https://github.com/anthropics/claude-plugins-official)
- [FOLIO Developer Documentation](https://dev.folio.org/)
