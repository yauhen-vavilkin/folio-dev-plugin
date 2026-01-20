# FOLIO Dev Plugin

Skills and tools for FOLIO microservices development with behavioral documentation.

## Skills

### document-feature

Analyzes code changes and generates behavioral feature documentation.

**Invocation:** `/folio-dev:document-feature`

**Workflow:**
1. Analyzes `git diff master...HEAD`
2. Proposes a feature name for confirmation
3. Inspects code for endpoints, caching, events, and configuration
4. Interviews you about unclear business rationale
5. Generates `docs/features/[feature-name].md`
6. Updates `docs/features.md` index

## Documentation Format

Feature documentation uses **behavioral description**—focusing on WHAT features do and WHY they exist, not HOW they're implemented.

## Installation

This plugin is distributed through the `folio-dev` marketplace. Install the marketplace to get this plugin:

```bash
/plugin install https://github.com/yauhen-vavilkin/folio-dev-plugin
```

Then install the plugin:

```bash
/plugin install folio-dev@folio-dev
```

## License

Apache-2.0
