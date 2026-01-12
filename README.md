# FOLIO Development Marketplace for Claude Code

> A marketplace of plugins, skills, and tools for documenting features and maintaining FOLIO microservices.

## Installation

Install the marketplace directly from GitHub:

```bash
/plugin install https://github.com/yauhen-vavilkin/folio-dev-plugin
```

## Available Plugins

### folio-dev

Skills and tools for FOLIO microservices development with behavioral documentation.

**Skills:**
| Skill | Description | Invocation |
|-------|-------------|------------|
| `document-feature` | Analyzes code changes and generates behavioral feature documentation | `/document-feature` |

**Quick Start:**

After installing the plugin and implementing a feature on a branch:

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

## Marketplace Structure

```
folio-dev-plugin/                    # Marketplace root
├── marketplace.json                 # Marketplace configuration
├── plugins/
│   └── folio-dev/                   # Individual plugin
│       ├── .claude-plugin/
│       │   └── plugin.json          # Plugin metadata
│       ├── skills/
│       │   └── document-feature/
│       │       └── SKILL.md         # Skill definition
│       └── README.md                # Plugin documentation
├── knowledge-base/                  # Shared research and best practices
│   └── DDD research.md
├── CLAUDE.md                   # Project overview and philosophy
└── README.md                        # This file
```

## Philosophy

This marketplace follows **document-driven development** principles. See [CLAUDE.md](CLAUDE.md) for:
- Why this marketplace exists
- What we're building
- Development roadmap
- Research foundation

## Contributing

We welcome new plugins and skills for FOLIO development:

1. Follow the [Claude Code Plugin](https://code.claude.com/docs/en/plugins) conventions
2. Add your plugin under `plugins/your-plugin-name/`
3. Update `marketplace.json` to include your plugin
4. Document in `CLAUDE.md` when adding major features
5. Add research and examples to `knowledge-base/`
6. Test on real FOLIO modules before submitting

## License

Apache-2.0

## Links

- **Marketplace Repository**: https://github.com/yauhen-vavilkin/folio-dev-plugin
- **FOLIO Developer Documentation**: https://dev.folio.org/
- **Claude Code Documentation**: https://code.claude.com/docs/en/plugins
