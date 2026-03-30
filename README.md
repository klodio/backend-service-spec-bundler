# Backend Service Spec Bundler

A Claude Code skill that generates complete, implementation-ready specification bundles for backend services. You describe an idea; it runs an interactive discovery session, then produces a full suite of spec documents. Note that it produces *no implementation code*, just specs.

It's highly opinionated because I don't care about your love for JSON. 

## What It Produces

```
your-project/
  SPEC.md                              # Authoritative technical specification
  ARCHITECTURE.md                      # C4 diagrams (Mermaid) and structural overview
  BEHAVIORS_AND_EXPECTATIONS.md        # Behavioral contracts (trigger/expected/rationale)
  CONTRIBUTING.md                      # Non-negotiable developer rules
  features/                            # BDD Gherkin feature files
    <feature-area>.feature
    ...
  docs/adr/                            # Architecture Decision Records
    0001-<decision>.md
    ...
```

All documents are cross-referenced: feature files link back to SPEC and B&E sections, ADRs reference the SPEC sections they impact, and so on.

## Installation

Copy or clone this repository into your Claude Code skills directory:

```
~/.claude/skills/backend-service-spec-bundler/
```

Or, if using a project-level skills directory:

```
<your-repo>/.claude/skills/backend-service-spec-bundler/
```

## Usage

Invoke the skill from Claude Code by passing it your idea as an argument:

```
/backend-service-spec-bundler a real-time inventory tracking service for warehouses
```

```
/backend-service-spec-bundler REST API gateway that aggregates data from three upstream microservices
```

```
/backend-service-spec-bundler CLI tool that monitors Postgres replication lag and alerts via PagerDuty
```

You can also invoke it with no argument and describe your idea when prompted.

## How It Works

The skill runs in three phases:

### Phase 1: Interactive Discovery (2-4 rounds)

The skill asks you questions in batches of 3-5 to understand your project. It covers:

- **Core understanding** -- What problem does this solve? Who uses it? What are the must-haves? What's out of scope?
- **Boundaries & constraints** -- External integrations, hard constraints, deployment model, interface type, wire format.
- **Behaviors & edge cases** -- Happy paths, failure modes, phased delivery plans, non-functional requirements.
- **Clarification** -- Resolves ambiguities before generating anything.

The skill won't proceed to generation until you confirm it has enough context.

### Phase 2: Spec Bundle Generation

Documents are generated in dependency order — each builds on the previous:

1. **SPEC.md** -- The authoritative blueprint. Glossary, constraints table, module decomposition, API definitions, data model, security model, error handling, performance targets, testing strategy.
2. **ARCHITECTURE.md** -- C4 diagrams (Context, Container, Component) in Mermaid. Sequence diagrams for major flows. Deployment and security architecture.
3. **BEHAVIORS_AND_EXPECTATIONS.md** -- Desired behaviors, undesired behaviors, edge cases, performance expectations, and guardrails. Every entry is structured as trigger/expected/rationale.
4. **BDD Feature Files** -- One `.feature` file per major feature area, in Gherkin format. Tagged with phases (`@phase1`, `@phase2`) when applicable.
5. **ADRs** -- One per significant architectural decision made during discovery.
6. **CONTRIBUTING.md** -- Developer rules derived from the project's constraints and decisions.

### Phase 3: Review & Iteration

After generation, you can review any document and request changes. The skill iterates on specs, not code.

## Design Philosophy

This skill has strong opinions:

- **No comfort-zone defaults.** It won't auto-suggest Python, JavaScript, or JSON because they're popular. It asks what matters (type safety, performance, ecosystem) and recommends based on your actual constraints. Rust, Go, Java, Kotlin, C++, Zig, and C# are all first-class options.

- **Simplicity bias.** Every piece of infrastructure must justify its existence. The skill will challenge you if you reach for Postgres when SQLite might suffice, or Kafka when an in-process queue would work. It starts from the leanest viable architecture and adds complexity only when requirements demand it.

- **No serialization defaults.** It won't assume JSON for wire formats. Binary formats (SBE, FlatBuffers, Protobuf, Cap'n Proto, MessagePack, CBOR) are evaluated alongside JSON based on payload characteristics.

- **Specs, not code.** The output is specifications only. If you ask it to write implementation code, it will decline. The specs are designed to be detailed enough that any competent engineer — or an AI coding agent — can implement from them without asking clarifying questions.

## Customizing the Templates

The `templates/` directory contains structural templates for each document type. These define the **format**, not the content — content comes entirely from the discovery session.

To customize:
- Edit the templates to add domain-specific sections
- Remove sections that never apply to your projects
- The skill will also adapt on its own: it omits irrelevant template sections and adds new ones when the project calls for it

## License

MIT
