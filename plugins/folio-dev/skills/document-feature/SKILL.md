---
name: document-feature
description: This skill should be used when the user asks to "document feature", "create feature documentation", "write documentation for this feature", or discusses documenting implemented features. The skill analyzes git diff between master and current branch, infers the feature name, and generates behavioral documentation in docs/features/.
version: 0.4.0
license: Apache-2.0
---

# Document Feature

You are a documentation specialist. After implementing a feature, analyze the code changes and generate behavioral feature documentation.

## Your goal

Create feature documentation that describes **WHAT** the feature does and **WHY** it exists—not HOW it's implemented. The documentation should be understandable to:
- A developer who wants to understand the module
- An AI agent that needs to modify or extend the feature
- A user who wants to know what the feature does

## Critical principle: Feature = Observable Behavior

**A feature is what users or external systems can interact with.** Implementation details (caching, events, internal utilities) are **aspects** of features, not features themselves.

| This is a FEATURE | This is an ASPECT (not a feature) |
|-------------------|-----------------------------------|
| User permission lookup | Permission cache |
| Capability registration | Cache eviction |
| Role assignment | Event publishing |

**Name features after behavior, not implementation:**
- ✅ `user-permission-lookup.md` (what it does)
- ❌ `user-permissions-cache.md` (how it's optimized)

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

## Entry point(s)
[Include the appropriate section based on entry point type - see below]

## Business rules and constraints
- [Rule 1 - plain language constraint]
- [Rule 2]
[What are the behavioral invariants? What must always be true?]

## Caching (if applicable)
[Describe caching as an internal optimization of this feature]
- What is cached
- Cache key structure (e.g., tenant-scoped)
- Cache eviction triggers

## Configuration (if applicable)
| Variable | Default | Purpose |
|----------|---------|---------|
| CONFIG_VAR_NAME | default_value | Description |

## Dependencies and interactions
- **Consumed by**: [What modules/services consume this feature?]
- **Depends on**: [What services/features does this depend on?]
- **Events published**: [If applicable]
- **Events consumed**: [If applicable]
```

### Entry point sections by type

**For REST endpoint features:**
```markdown
## Entry point(s)
| Method | Path | Description |
|--------|------|-------------|
| GET | /permissions/users/{id} | Returns list of permissions for a user |
```

**For Kafka consumer features:**
```markdown
## Entry point(s)
| Type | Topic | Description |
|------|-------|-------------|
| Kafka Consumer | `mgr-tenant-entitlements.capability` | Processes capability change events |

### Event processing
- **When processed**: On each message from the topic
- **Event types handled**: CREATE, UPDATE, DELETE
- **Processing behavior**: [What happens when event is received]
```

**For scheduled job features:**
```markdown
## Entry point(s)
| Type | Schedule | Description |
|------|----------|-------------|
| Scheduled Job | Every 5 minutes | Refreshes token cache |
```

**For internal event features:**
```markdown
## Entry point(s)
| Type | Event | Description |
|------|-------|-------------|
| Internal Event | `UserPermissionsChangedEvent` | Triggered when user permissions are modified |
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
- What entry points were modified or created?
- What services changed?
- Any new events, caches, or configuration?

### 2. Identify entry points and observable behavior

**First, identify ALL entry points in the changes:**

| Pattern to look for | Entry point type |
|---------------------|------------------|
| `@RestController`, `@GetMapping`, `@PostMapping` | REST endpoint |
| `@KafkaListener` | Kafka consumer |
| `@Scheduled` | Scheduled job |
| `@EventListener`, `@TransactionalEventListener` | Internal event handler |

**Then ask: "What user-observable behavior changed?"**

The feature is what external consumers interact with. Caching, events, and internal utilities are aspects of features, not features themselves.

### 3. Clarify feature boundaries with user

**IMPORTANT: When changes touch multiple concerns, ASK the user:**

> "The changes touch [cache/events/multiple services]. Help me understand the feature boundary:
> 1. Is this a NEW feature, or adding behavior to an EXISTING feature?
> 2. What is the primary user-observable behavior?
> 3. [If events are involved] Are these new event handlers, or adding logic to existing ones?"

**Example clarification:**
- Changes add cache eviction to existing event handlers → Document as "Caching" section in existing feature
- Changes create new Kafka listener → Document as new event processing feature

### 4. Infer feature name (behavior-based)

Based on the observable behavior, propose **1-3 feature name options**. Names must reflect behavior, not implementation:

| Changes observed | Bad name | Good name |
|------------------|----------|-----------|
| Cache for permission lookups | `user-permissions-cache` | `user-permission-lookup` |
| Kafka listener for capabilities | `capability-kafka-handler` | `capability-event-processing` |
| Scheduled token refresh | `token-refresh-scheduler` | `token-management` |

Ask user to confirm or provide the correct name.

### 5. Deep code inspection

**Read the changed files** to understand:
- The complete flow from entry point to response/effect
- Business rules and constraints
- Error handling behavior
- Configuration that affects behavior

**For REST features, verify the endpoint exists:**
1. Identify the service class being modified
2. Find controllers: `grep -rn "ServiceName" src/main/java --include="*Controller.java"`
3. Check OpenAPI/Swagger specs if available
4. Document the actual endpoint path and method

**For Kafka features, document:**
- Topic name (from `@KafkaListener` annotation or properties)
- Event types handled
- Processing behavior and side effects

### 6. Interview the user

If any of the following are unclear from the code, ask the user:
- **Business rationale**: Why was this feature needed? What problem does it solve?
- **Configuration choices**: Why was this TTL/size chosen? What are the tradeoffs?
- **Edge cases**: Are there any special behaviors or constraints not obvious from code?

### 7. Generate documentation

- Create `docs/features/[feature-name].md` using the appropriate template
- Update `docs/features.md` to include the new feature in the table of contents
- Use today's date in the `updated` frontmatter field
- Write files directly—do not ask for approval

### 8. Handle existing documentation

If documentation already exists for the feature:
- Read the existing file
- Update relevant sections based on new changes
- Preserve content that remains accurate
- Update the date in frontmatter

### 9. Skip trivial changes

If the changes are minor (typo fixes, refactoring with no behavior change, test updates), inform the user that these changes don't warrant feature documentation and ask if they want to proceed anyway.

## Important notes

- **File naming**: Always use kebab-case based on behavior (e.g., `user-permission-lookup.md`)
- **No owners field**: Do not include an `owners` field in the frontmatter
- **Behavioral focus**: Describe what the system does, not how the code works
- **Caching is an aspect**: Document caching within the feature it optimizes, not as separate feature
- **Configuration relevance**: Only document configuration that affects the feature's behavior
- **External context**: Mention other modules/services that interact with this feature

## Example analysis

**Example 1: REST endpoint feature with caching optimization**

Changes observed:
- New `UserPermissionCacheService` with `@Cacheable`
- Cache eviction via `UserPermissionsCacheEvictor`
- Event handlers for cache invalidation
- Cache configuration in `application.yml`

**Step 1: Find the entry point**
```bash
grep -rn "UserPermissionCacheService" src/main/java --include="*Controller.java"
# Found: PermissionsUsersController uses CapabilityService which uses UserPermissionCacheService
```

**Step 2: Identify the feature**
- Entry point: `GET /permissions/users/{id}`
- Observable behavior: Returns user permissions
- Caching is an optimization aspect

**Step 3: Document as `user-permission-lookup.md`**
- What: Returns permissions for a user
- Why: Authorization checks need quick permission lookups
- Entry point: REST endpoint table
- Caching: Section describing the optimization
- Events: Cache eviction triggers

**Example 2: Kafka consumer feature (no REST endpoint)**

Changes observed:
- New `@KafkaListener` for capability events
- Event processing logic
- No controller changes

**Step 1: Identify entry point**
- Topic: `mgr-tenant-entitlements.capability`
- Event types: CREATE, UPDATE, DELETE

**Step 2: Document as `capability-event-processing.md`**
- What: Processes capability changes from external systems
- Why: Keeps local capability registry synchronized
- Entry point: Kafka consumer table
- Event processing section with behavior details

**Example 3: Adding behavior to existing feature**

Changes observed:
- Added cache eviction calls to existing event handlers
- No new entry points

**Step 1: Clarify with user**
> "The changes add cache eviction to existing event handlers. Is this a new feature or enhancing an existing one?"

User: "Enhancing existing user permission lookup with cache eviction."

**Step 2: Update existing documentation**
- Update `user-permission-lookup.md`
- Add/update "Caching" section with eviction behavior
- Update the date
