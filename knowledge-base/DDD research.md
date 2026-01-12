# Document-driven development for AI-consumable feature documentation

Writing documentation before code transforms how AI agents understand and maintain software projects. This research reveals that **behavioral feature documentation**—capturing what features do and why, rather than how they're implemented—becomes the critical artifact when AI agents are expected to read, understand, and update project knowledge. The most effective approach combines **Architecture Decision Records (ADRs) adapted for features**, **Markdown with YAML frontmatter** for AI parseability, and **synchronization tools like Swimm** that detect when documentation drifts from code.

For FOLIO's Java microservices on Spring/Quarkus, the recommended structure places feature documentation in `docs/features/` with each feature having dedicated files for overview, behavioral specification, and business invariants—all in Markdown format that both humans and LLMs process efficiently.

## README-driven development establishes documentation-first practices

Tom Preston-Werner's 2010 manifesto on README-driven development establishes the foundational principle: **"Write your README first. First. As in, before you write any code."** This forces clarity of thought before implementation begins, when motivation is highest and changing specifications costs nothing.

The methodology yields three key benefits for AI-assisted workflows. First, documentation created at project inception captures the *rationale* behind decisions—exactly what AI agents need to understand features deeply. Second, the README becomes a specification that drives implementation, ensuring code and documentation start aligned. Third, team members (including AI agents) can begin work on dependent features before implementation completes.

**Document-driven development** extends this philosophy: from a user's perspective, if a feature isn't documented, it doesn't exist, and if documented incorrectly, it's broken. This framing transforms documentation from afterthought to first-class artifact—precisely the mindset required for AI-maintained documentation systems.

## Architecture Decision Records capture the "why" that code cannot express

ADRs, endorsed at Thoughtworks' highest "Adopt" recommendation, solve the problem of institutional memory in long-lived projects. Michael Nygard's original 2011 template captures decisions in short, conversational text files:

```markdown
# ADR-001: Cache heavy patron query results

## Status
Accepted

## Context
The patron eligibility query joins 7 tables and executes 3 subqueries, 
taking 800ms average. This query runs on every checkout, creating 
unacceptable latency during peak hours. The underlying data changes 
only when patron records are modified.

## Decision
We will cache query results in-memory with no time-based expiration. 
Cache entries will be invalidated when any patron entity is mutated 
(create, update, delete). Other features requiring patron eligibility 
data will filter this cache rather than executing the database query.

## Consequences
- Checkout latency reduced to <50ms (positive)
- Memory usage increases ~2MB per 1000 active patrons (neutral)
- All patron mutations must trigger cache invalidation (negative)
- Features depending on this cache inherit its consistency model (neutral)
```

This example demonstrates **behavioral documentation**: it explains *what* the caching does, *why* the query is expensive, and *what constraints* other developers must respect—without describing implementation patterns or code structure.

**MADR (Markdown Any Decision Records)** extends the template with explicit decision drivers, considered options, and pros/cons analysis. The key insight from researcher Olaf Zimmermann: ADRs work for *any* significant decision, not just architectural ones. Teams that rename the directory from `adr/` to `decisions/` find themselves documenting vendor choices, planning decisions, and **feature-level decisions**—exactly the behavioral documentation this project requires.

## Living documentation keeps knowledge synchronized with reality

Cyrille Martraire's "Living Documentation" framework establishes four principles for documentation that stays accurate: **reliable** (co-exists with code in version control), **low effort** (automation minimizes maintenance), **collaborative** (multiple contributors work together), and **insightful** (provides valuable knowledge, not just data).

The **docs-as-code** methodology operationalizes these principles:

- Write documentation in plain text formats (Markdown, AsciiDoc)
- Store alongside code in Git repositories
- Use identical workflows: pull requests, code reviews, CI/CD
- Automate validation with linters and link checkers
- Publish through static site generators (MkDocs, Docusaurus)

The UK Government Digital Service reports that this approach enables shared ownership: "Anyone can submit a pull request against the documentation. GitHub's comment threads allow the team to discuss possible changes before merging."

For AI consumption, this methodology ensures documentation remains queryable, versionable, and tied to the code it describes—essential properties when AI agents must determine whether documentation reflects current implementation.

## Markdown dominates AI-readable formats for good reason

Research into LLM consumption patterns reveals **Markdown as the clear winner** for AI-parseable documentation:

| Format | AI Parseability | Token Efficiency | Best Use Case |
|--------|----------------|------------------|---------------|
| Markdown | Excellent | High | Prose, instructions, behavioral docs |
| YAML | Very good | High | Metadata, frontmatter, configuration |
| JSON | Very good | Medium | Structured data, schemas |
| AsciiDoc | Good | Medium | Complex technical documentation |
| HTML/XML | Poor | Low | Avoid for AI consumption |

Markdown's advantages stem from LLM training data: massive GitHub repositories, documentation sites, and technical writing all use Markdown. The format's hierarchical headings (`#`, `##`, `###`) provide structural understanding, while its minimal syntax maximizes information density per token.

**YAML frontmatter** adds machine-parseable metadata to Markdown files:

```yaml
---
feature_id: patron-cache
status: active
updated: 2025-01-08
owners: ["@checkout-team"]
depends_on: ["patron-service", "notification-service"]
invariants:
  - "Cache invalidates on any patron mutation"
  - "Other features must filter cache, not query database"
---
```

This combination—YAML metadata plus Markdown prose—gives AI agents both structured fields for programmatic access and natural language for behavioral understanding.

## AI agent context files establish project conventions

Modern AI coding assistants recognize specific files for project-level instructions:

| Tool | Context File | Location |
|------|-------------|----------|
| Claude Code | CLAUDE.md | Project root |
| Gemini CLI | GEMINI.md | Hierarchical (root + subdirs) |
| GitHub Copilot | .github/copilot-instructions.md | .github/ |
| Cursor | .cursor/rules/*.mdc | .cursor/rules/ |
| Universal | AGENTS.md | Project root |

**AGENTS.md** emerges as a universal standard, with tools that don't natively support it accepting symlinks: `ln -s AGENTS.md CLAUDE.md`. Over **20,000 GitHub repositories** now include this file.

The **llms.txt** specification (proposed September 2024 by Jeremy Howard) provides a site-wide index for AI consumption:

```markdown
# FOLIO Checkout Module

> Handles patron checkout operations including eligibility verification, 
> item lending, and due date calculation.

## Features
- [Patron Caching](./docs/features/patron-cache.md): In-memory caching of heavy patron eligibility queries
- [Due Date Calculation](./docs/features/due-dates.md): Loan period and calendar-aware date computation

## Architecture Decisions
- [ADR-001](./docs/adr/0001-patron-cache.md): Cache strategy for patron queries
```

This structure allows AI agents to quickly identify what documentation exists and navigate to relevant content.

## Feature documentation requires behavioral schemas, not code documentation

The critical distinction for this project: **feature documentation describes behavior and rationale, not implementation**. A behavioral documentation schema for features:

```markdown
---
feature_id: patron-cache
title: Patron Eligibility Cache
status: active
created: 2024-06-15
updated: 2025-01-08
owners: ["@checkout-team"]
---

# Patron eligibility cache

## What it does
Caches the results of patron eligibility queries in memory, eliminating 
database access during checkout operations. Other features that need 
patron eligibility data filter this cache rather than executing queries.

## Why it exists
The patron eligibility query joins 7 tables across the patron, holds, 
fines, and blocks domains. Average execution time is 800ms, creating 
unacceptable latency when 50+ patrons check out simultaneously during 
class changeover periods. Since patron data changes infrequently 
(~2% of patrons per day), caching provides dramatic performance 
improvement with minimal staleness risk.

## Business rules and constraints
- Cache has **no time-based expiration**—entries live until explicitly invalidated
- Any mutation to patron-related entities (patron records, blocks, fines, holds) 
  **must trigger cache invalidation** for affected patrons
- Features requiring patron eligibility data **must filter the cache** rather 
  than querying the database directly
- Cache warming occurs at service startup, loading all active patrons

## Invariants
- A patron's cached eligibility status always reflects their state at the 
  time of last cache refresh or invalidation
- The cache never contains eligibility data for deleted patrons
- All services filtering the cache observe the same eligibility state 
  (no per-service caching)

## Dependencies
- **Requires**: patron-service (source of patron data)
- **Triggers**: notification-service (cache invalidation events)
- **Consumed by**: checkout-service, hold-request-service, fine-service

## Related decisions
- [ADR-001](../adr/0001-patron-cache.md): Initial cache strategy selection
- [ADR-007](../adr/0007-cache-invalidation.md): Event-based invalidation approach
```

This template captures everything an AI agent needs to understand the feature without describing Spring annotations, Java patterns, or API signatures.

## Microservice architectures benefit from layered documentation

FOLIO's existing documentation structure demonstrates effective patterns for microservice projects:

**Layer 1: User documentation** organized by functional area (access, acquisitions, metadata management)

**Layer 2: Developer documentation** organized by task type (getting started, guides, reference, tutorials)

**Layer 3: Module-level documentation** living alongside source code in each repository

**Quarkus explicitly adopts the Diátaxis framework**, separating documentation into four distinct types:

- **Concepts/Explanations**: Understanding-oriented, covering *why* things work as they do
- **How-to guides**: Goal-oriented directions for specific problems
- **Reference**: Technical descriptions, often auto-generated
- **Tutorials**: Learning-oriented step-by-step lessons

For behavioral feature documentation, the **Concepts/Explanations** category is primary. This documentation type analyzes features from multiple perspectives, includes design decisions and historical context, and explains technical constraints—precisely the content AI agents need.

Cross-service behavior documentation requires special attention in microservice architectures. Recommended patterns include:

- **Documentation assembly files** describing service intersections
- **Event choreography documentation** showing cross-service flows
- **Business invariant catalogs** documenting rules that span services
- **Saga pattern documentation** for distributed transactions

## Synchronization tools detect when documentation drifts from code

**Swimm** leads the market in documentation synchronization with a patented algorithm that automatically updates documentation when code changes:

- Links code snippets directly within documentation
- Detects when linked code changes and either auto-updates (minor changes) or flags for human review (significant changes)
- Integrates with CI/CD to block PRs when documentation is outdated
- Provides IDE plugins surfacing relevant documentation while coding

**Mintlify** offers AI-native documentation with GitHub/GitLab sync, automatically detecting code changes and suggesting documentation updates. Its AI-powered writer generates documentation from code comments and function signatures.

**DocuWriter.ai** monitors connected repositories and generates documentation update suggestions, with n8n integration for workflow automation.

For Java/Spring projects, these tools complement existing documentation infrastructure. The recommended integration:

```yaml
name: Documentation Validation
on: [pull_request]
jobs:
  docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Validate feature documentation
        run: |
          npx markdownlint-cli docs/features/**/*.md
          npx swimm verify
      - name: Check for missing feature docs
        run: |
          # Custom script checking new features have documentation
          ./scripts/check-feature-docs.sh
```

## Teams achieving AI-assisted documentation share common patterns

**Uber's "Genie" AI copilot** handles 70,000+ support interactions by accessing internal documentation via RAG, saving approximately 13,000 engineering hours by answering technical queries autonomously. The key insight: comprehensive, well-structured documentation becomes the knowledge base that makes AI assistance possible.

**Monzo Bank** embedded documentation updates into their CI/CD process, requiring documentation changes to go through the same review process as code. This governance ensures documentation quality.

Effective teams share these practices:

1. **Make documentation updates required** before merging feature PRs
2. **Use AI prompts consistently** for documentation generation:
   ```
   "After implementing [FEATURE], update:
   1. docs/features/[feature]/overview.md - behavioral description
   2. docs/features/[feature]/invariants.md - business rules
   3. CHANGELOG.md - entry under 'Unreleased'"
   ```
3. **Create AI instruction files** (AGENTS.md) specifying documentation requirements
4. **Maintain "learnings docs"** capturing insights after each implementation
5. **Run documentation validation in CI** blocking merges when docs are outdated

## Recommended implementation for FOLIO microservices

Based on this research, the recommended documentation structure for FOLIO's Java/Spring/Quarkus modules:

```
module-root/
├── docs/
│   ├── features/                    # Behavioral feature documentation
│   │   ├── patron-cache/
│   │   │   ├── overview.md          # What and why
│   │   │   ├── behavior.md          # Detailed behavioral spec
│   │   │   └── invariants.md        # Business rules and constraints
│   │   └── checkout-flow/
│   │       └── ...
│   ├── adr/                         # Architecture/feature decisions
│   │   ├── 0001-patron-cache.md
│   │   └── template.md
│   ├── cross-service/               # Multi-service behaviors
│   │   └── checkout-notification-flow.md
│   └── llms.txt                     # AI documentation index
├── AGENTS.md                        # AI agent instructions
├── README.md                        # Module overview
└── src/
```

**Feature documentation template** (docs/features/[name]/overview.md):

```markdown
---
feature_id: [feature-name]
title: [Human-readable title]
status: [active|deprecated|experimental]
updated: [YYYY-MM-DD]
owners: ["@team-name"]
depends_on: ["service-a", "service-b"]
---

# [Feature name]

## What it does
[2-3 sentence description of observable behavior, not implementation]

## Why it exists
[Business rationale, user problem solved, performance/scale justification]

## Business rules
- [Rule 1 - plain language constraint]
- [Rule 2]

## Invariants
- [Condition that must always hold]
- [Another invariant]

## Dependencies and interactions
- **Requires**: [upstream services/features]
- **Consumed by**: [downstream services/features]
- **Events published**: [if applicable]
- **Events consumed**: [if applicable]

## Related decisions
- [Link to relevant ADR]
```

**AGENTS.md template** for AI agent instructions:

```markdown
# FOLIO Module AI Instructions

## Documentation Requirements
When implementing or modifying features:
1. Create/update docs/features/[feature-name]/overview.md
2. Document business rules and invariants explicitly
3. Update cross-service documentation if behavior spans services
4. Create ADR for significant design decisions

## Documentation Style
- Describe WHAT features do and WHY, not HOW they're implemented
- Focus on behavioral contracts, not code patterns
- Include business context and rationale
- State constraints and invariants explicitly

## Feature Documentation Location
All feature documentation lives in docs/features/[feature-name]/
Each feature needs at minimum an overview.md file.
```

## Conclusion: behavioral documentation becomes the AI knowledge layer

The convergence of document-driven development methodologies, AI-optimized formats, and synchronization tooling creates a viable system for AI-maintained documentation. The key insight is that **behavioral documentation—capturing what features do and why rather than implementation details—provides exactly the knowledge AI agents need** to understand, modify, and maintain software systems.

For FOLIO's microservices, success requires:

1. **Adopting Markdown with YAML frontmatter** as the universal documentation format
2. **Creating AGENTS.md files** specifying documentation requirements for AI agents
3. **Extending ADRs to feature decisions** capturing rationale and constraints
4. **Integrating Swimm or similar tools** for synchronization detection
5. **Making documentation updates mandatory** in the PR review process
6. **Structuring features/ directories** with consistent templates focused on behavior

The documentation becomes a contract that AI agents can read, understand, and update—transforming maintenance from a human-only burden to a collaborative human-AI workflow where both parties share the same source of truth about what the system does and why.