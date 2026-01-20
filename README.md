# FOLIO Development Marketplace for Claude Code

> A marketplace of plugins, skills, and tools for documenting features and maintaining FOLIO microservices.

## Installation

1. **Install the marketplace:**
```bash
/plugin install https://github.com/yauhen-vavilkin/folio-dev-plugin
```

2. **Install plugins from the marketplace:**
```bash
/plugin install folio-dev@folio-dev
```

3. **Use the skills:**
```bash
/folio-dev:document-feature
```

## Plugins

### folio-dev

Skills and tools for FOLIO microservices development with behavioral documentation.

**Skills:**
- `/folio-dev:document-feature` - Analyzes code changes and generates behavioral feature documentation

**Workflow:**
1. Analyzes `git diff master...HEAD`
2. Proposes a feature name for confirmation
3. Inspects code for endpoints, caching, events, and configuration
4. Interviews you about unclear business rationale
5. Generates `docs/features/[feature-name].md`
6. Updates `docs/features.md` index

See [plugins/folio-dev/README.md](plugins/folio-dev/README.md) for details.

## Documentation Format

Feature documentation uses **behavioral description**—focusing on WHAT features do and WHY they exist, not HOW they're implemented.

## Structure

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
├── knowledge-base/                  # Shared research and best practices
│   └── DDD research.md
├── CLAUDE.md                        # Project overview and philosophy
└── README.md                        # This file
```

This follows the **multi-plugin marketplace pattern** (like `anthropics/claude-plugins-official`), allowing future plugins to be added easily.

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
