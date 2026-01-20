# FOLIO Development Plugin for Claude Code

> Skills and tools for documenting features and maintaining FOLIO microservices.

## Installation

Install the plugin directly from GitHub:

```bash
/plugin install https://github.com/yauhen-vavilkin/folio-dev-plugin
```

## Skills

### document-feature

Analyzes code changes and generates behavioral feature documentation.

**Invocation:** `/document-feature`

**Workflow:**
1. Analyzes `git diff master...HEAD`
2. Proposes a feature name for confirmation
3. Inspects code for endpoints, caching, events, and configuration
4. Interviews you about unclear business rationale
5. Generates `docs/features/[feature-name].md`
6. Updates `docs/features.md` index

**Quick Start:**

After installing the plugin and implementing a feature on a branch:

```bash
/document-feature
```

## Documentation Format

Feature documentation uses **behavioral description**—focusing on WHAT features do and WHY they exist, not HOW they're implemented.

## Plugin Structure

```
folio-dev-plugin/                    # Plugin root
├── .claude-plugin/
│   └── plugin.json                  # Plugin metadata
├── skills/
│   └── document-feature/
│       └── SKILL.md                 # Skill definition
├── knowledge-base/                  # Research and best practices
│   └── DDD research.md
├── CLAUDE.md                        # Project overview and philosophy
└── README.md                        # This file
```

## Philosophy

This plugin follows **document-driven development** principles. See [CLAUDE.md](CLAUDE.md) for:
- Why this plugin exists
- What we're building
- Development roadmap
- Research foundation

## Contributing

We welcome contributions and improvements:

1. Follow the [Claude Code Plugin](https://code.claude.com/docs/en/plugins) conventions
2. Test on real FOLIO modules before submitting
3. Document in `CLAUDE.md` when adding major features
4. Add research and examples to `knowledge-base/`

## License

Apache-2.0

## Links

- **Plugin Repository**: https://github.com/yauhen-vavilkin/folio-dev-plugin
- **FOLIO Developer Documentation**: https://dev.folio.org/
- **Claude Code Documentation**: https://code.claude.com/docs/en/plugins
