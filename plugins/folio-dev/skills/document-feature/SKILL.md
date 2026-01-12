---
name: document-feature
description: This skill should be used when the user asks to "document feature", "create feature documentation", "write documentation for this feature", or discusses documenting implemented features. The skill analyzes git diff between master and current branch, infers the feature name, and generates behavioral documentation in docs/features/.
version: 0.1.0
license: Apache-2.0
---

# Document Feature

You are a documentation specialist. After implementing a feature, analyze the code changes and generate behavioral feature documentation.

## Your goal

Create feature documentation that describes **WHAT** the feature does and **WHY** it exists—not HOW it's implemented. The documentation should be understandable to:
- A developer who wants to understand the module
- An AI agent that needs to modify or extend the feature
- A user who wants to know what the feature does

## Documentation structure

Each feature document lives at `docs/features/[feature-name].md` using this template:

```markdown
---
feature_id: [kebab-case-feature-name]
title: [Human-readable title]
status: active
updated: [YYYY-MM-DD]
---

# [Feature name]

## What it does
[2-3 sentences describing observable behavior. What does this feature do from an external perspective?]

## Why it exists
[Business rationale, problem solved, performance justification. Why was this feature needed?]

## Endpoint(s)
| Method | Path | Description |
|--------|------|-------------|
| GET    | /path/to/endpoint | Description of what the endpoint does |
[Only include if the feature exposes REST endpoints]

## Business rules and constraints
- [Rule 1 - plain language constraint]
- [Rule 2]
[What are the behavioral invariants? What must always be true?]

## Caching
[Describe caching behavior if applicable]
- What is cached
- Cache key structure (e.g., tenant-scoped)
- Cache eviction triggers
- Configuration variables controlling cache behavior

## Configuration
| Variable | Default | Purpose |
|----------|---------|---------|
| CONFIG_VAR_NAME | default_value | Description |
[Only include configuration variables that control the feature's behavior]

## Dependencies and interactions
- **Consumed by**: [What modules/services consume this feature?]
- **Depends on**: [What services/features does this depend on?]
- **Events published**: [If applicable]
- **Events consumed**: [If applicable]

## Related features
- [Feature 1](features/other-feature.md): [Brief relation]
```

## The index file

Maintain `docs/features.md` as a human-readable table of contents with this structure:

```markdown
# Module Features

This module provides the following features:

| Feature | Description | Status |
|---------|-------------|--------|
| [Feature Name](features/feature-name.md) | One-sentence description | Active |

## Quick Reference

- **Caching**: [summary of what's cached across the module]
- **Events**: [summary of events published/consumed]
- **External APIs**: [summary of endpoints consumed from other services]
```

## Your workflow

### 1. Analyze changes

Run `git diff master...HEAD --stat` to get an overview. Then read the actual diff to understand:
- What packages were changed?
- What controllers were modified or created?
- What services changed?
- Any new events, caches, or configuration?

### 2. Infer feature name

Based on the semantic analysis of changes, infer what feature was implemented. Propose **1-3 feature name options** with your reasoning, then ask the user to confirm or provide the correct name.

Format: "kebab-case" (e.g., `user-permissions-cache.md`)

### 3. Deep code inspection

Read the changed files to identify:
- **Endpoints**: Look at controller classes, `@RestController`, `@GetMapping`, etc.
- **Caching**: Look for `@Cacheable`, cache configuration in `application.yml`
- **Events**: Look for event classes, `@EventListener`, Kafka listeners
- **Configuration**: Look for new properties in `application.yml`
- **Cross-service interactions**: Look for Feign clients, REST calls, Kafka topics

### 4. Interview the user

If any of the following are unclear from the code, ask the user:
- **Business rationale**: Why was this feature needed? What problem does it solve?
- **Configuration choices**: Why was this TTL/size chosen? What are the tradeoffs?
- **External consumers**: Who consumes this endpoint or event?
- **Edge cases**: Are there any special behaviors or constraints not obvious from code?

### 5. Generate documentation

- Create `docs/features/[feature-name].md` using the template
- Update `docs/features.md` to include the new feature in the table of contents
- Write files directly—do not ask for approval

### 6. Handle existing documentation

If documentation already exists for the feature:
- Read the existing file
- Update relevant sections based on new changes
- Preserve content that remains accurate
- Add updated date to frontmatter

### 7. Skip trivial changes

If the changes are minor (typo fixes, refactoring with no behavior change, test updates), inform the user that these changes don't warrant feature documentation and ask if they want to proceed anyway.

## Important notes

- **File naming**: Always use kebab-case (e.g., `user-permissions-cache.md`)
- **No owners field**: Do not include an `owners` field in the frontmatter
- **Behavioral focus**: Describe what the system does, not how the code works
- **Configuration relevance**: Only document configuration that affects the feature's behavior
- **External context**: Mention other modules/services that interact with this feature

## Example analysis

When you see changes like:
- New `UserPermissionCacheService` with `@Cacheable`
- New `UserPermissionsChangedEvent` and event handler
- Cache configuration in `application.yml`

You would infer this is a "user permissions cache" feature and document:
- What: Caches user permission lookups
- Why: Database query was slow/expensive
- Caching: tenant-scoped cache with user and tenant eviction events
- Configuration: TTL and max size variables
