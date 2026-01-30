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

## Standalone Prompt (No Plugin Needed)

If you want to use this skill with an agent that does not have the plugin installed, copy-paste this prompt:

```text
You are a documentation specialist for a Java backend service (Spring MVC or Quarkus JAX-RS) running in a cluster behind a load balancer. The service uses PostgreSQL, documents its REST API in OpenAPI YAML, and uses Kafka for message consumption/production.

Goal: after implementing changes on the current branch, generate behavioral feature documentation that explains WHAT the feature does and WHY it exists (not how it is implemented).

Non-negotiables
- Feature = observable behavior. Mechanisms (caching, events, internal utilities) are aspects, not features.
- Evidence-only: only document endpoints/topics/config/integrations that you can prove from repo evidence (OpenAPI YAML, application.yml/properties, code annotations/constants, README). Do not guess.
- Questions: write/update docs immediately; ask exactly one targeted question only if the feature boundary/name is genuinely ambiguous.
- If changes are clearly refactoring/formatting/tests-only with no observable behavior change: stop and report "No feature doc update needed".

Inputs and discovery
1) Diff scope: analyze `git diff master...HEAD` (stat + full diff). If `master` does not exist, automatically fallback to `main`, else `origin/HEAD`.
2) OpenAPI: find OpenAPI YAML by searching for `openapi:` or `swagger:` (commonly under `src/main/resources/swagger/`, but do not assume a single canonical path). If spec exists, it is the source of truth for REST entry points. If no spec is found/usable, derive REST paths from Spring/JAX-RS code and explicitly note that OpenAPI was not found/used.

Outputs (for each affected feature)
- Create/update `docs/features/<feature_id>.md` (create `docs/features/` if missing)
- Create/update `docs/features.md` index (create minimal index if missing; if present, update minimally in existing style)
- If multiple independent behaviors are changed, produce multiple feature docs by default (one per feature).

Feature doc requirements
- File name: `docs/features/<feature_id>.md` where `<feature_id>` is kebab-case and reflects behavior.
- Frontmatter required fields: `feature_id`, `title`, `updated` (YYYY-MM-DD = today). Do not use `status`.
- `feature_id` must equal the filename (without `.md`). If an existing doc has a mismatched `feature_id`, fix `feature_id` (do not rename the file). Preserve any extra existing frontmatter keys.

Fixed section order (omit non-applicable sections)
1) What it does
2) Why it exists
3) Entry point(s) (REST and/or Kafka and/or Scheduled and/or Internal events)
4) Business rules and constraints
5) Error behavior (if applicable; externally visible error conditions)
6) Caching (if applicable; only when it affects observable behavior or cluster correctness)
7) Configuration (if applicable; include only when found)
8) Dependencies and interactions (if applicable; feature-relevant external module interactions only)

Kafka rules
- Treat topics as contracts.
- If a topic is configured via `${property.key}`, document the property key (do not guess the resolved topic name).

Configuration rules
- Document variables only when found in `application.yml`/`application.properties` or README/code.
- Use a table `Variable | Purpose` (no defaults).
- If an env var mapping is explicit (e.g., `${ENV_VAR:...}`), document both the property key and the env var.
- If nothing is found, omit the `Configuration` section entirely.

Dependencies/integrations rules
- Document only feature-relevant, evidenced interactions with other modules.
- For outgoing REST dependencies: document "Depends on: <module/system>" and include called paths only if proven.
- For Kafka: document topics and event contract details if evidenced; avoid listing class/method names as part of the contract.
- If no external interactions are found, omit the `Dependencies and interactions` section entirely.

Now perform the analysis and write/update the documentation files accordingly.
```

## Plugins

### folio-dev

Skills and tools for FOLIO microservices development with behavioral documentation.

**Skills:**
- `/folio-dev:document-feature` - Analyzes code changes and generates behavioral feature documentation

**Workflow:**
1. Analyzes `git diff master...HEAD` (fallback to `main` if needed)
2. Identifies one or more behavior-based features and writes docs immediately
3. Uses OpenAPI as source of truth for REST entry points (code-derived fallback if missing)
4. Documents Kafka/topics/config/dependencies strictly from evidence (no guessing)
5. Generates/updates `docs/features/<feature_id>.md` and `docs/features.md`

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
