# RFC-0001: The Effector Specification

- **Status:** Draft
- **Author:** effectorHQ Contributors
- **Created:** 2026-03-09
- **Updated:** 2026-03-09
- **Tracking Issue:** TBD
- **Full Spec:** [effectorHQ/effector-spec](https://github.com/effectorHQ/effector-spec)

## Summary

This RFC proposes the Effector Specification v0.1.0 — a formal standard that defines what an Effector is, how it's packaged, distributed, composed, and consumed by AI agent runtimes. The spec introduces `effector.toml` as a universal manifest format and defines six canonical Effector types: skill, extension, workflow, workspace, bridge, and prompt.

## Motivation

The AI agent ecosystem has a fragmentation problem. Every runtime invents its own format for capabilities:

- OpenClaw uses `SKILL.md` (YAML frontmatter + markdown body) for skills, `openclaw.plugin.json` for extensions, `pipeline.yml` for Lobster workflows, and `SOUL.md`/`AGENTS.md`/`TOOLS.md` for workspace configuration
- Claude Agent SDK has its own tool definition format
- MCP defines tools via JSON-RPC schema
- LangChain, CrewAI, AutoGen, and others all have incompatible capability formats

These formats solve the same problem — packaging a discrete capability so an AI agent can use it — but they can't interoperate. A skill built for OpenClaw doesn't work in Claude Desktop without a manual bridge. A MCP tool doesn't know about Lobster pipelines.

The Effector Spec proposes a **unifying layer** that sits above all of these formats. Not a replacement — a wrapper. The `effector.toml` manifest describes the capability's identity, type, dependencies, permissions, and runtime bindings. Each runtime reads its own binding section. One package, multiple runtimes.

### Why now

1. **Critical mass.** OpenClaw's ClawHub has 3,286+ skills. The format works. It's time to generalize.
2. **MCP momentum.** Anthropic's Model Context Protocol is becoming the bridge standard. We need a packaging format that speaks MCP natively.
3. **Community demand.** effectorHQ Discussion #1 announced the Effector concept. The spec makes it concrete.

## Detailed Design

The full specification lives at [effectorHQ/effector-spec](https://github.com/effectorHQ/effector-spec) and consists of seven documents:

| Document | Purpose |
|----------|---------|
| 00 — Overview | What an Effector is, design principles, conceptual model |
| 01 — Manifest | The `effector.toml` format (required fields, optional fields, runtime bindings) |
| 02 — Types | Six canonical types with semantics and OpenClaw mappings |
| 03 — Lifecycle | Create → validate → package → publish → discover → install → execute |
| 04 — Composition | Dependencies, composition patterns, capability negotiation |
| 05 — Runtime Binding | How runtimes consume Effectors (OpenClaw reference, MCP, generic) |
| 06 — Security | Permission model, trust levels, sandboxing |

### Core Design Decisions

**1. TOML over JSON or YAML.** TOML has unambiguous semantics (no YAML type coercion gotchas), better readability than JSON for configuration, and first-class support for nested tables. The manifest should be readable in under 30 seconds.

**2. Backward compatibility is non-negotiable.** Every existing `SKILL.md` on ClawHub is automatically a valid skill Effector without any changes. Runtimes infer the manifest from SKILL.md frontmatter when no `effector.toml` exists.

**3. Six types, not fewer, not more.** The taxonomy is based on actual OpenClaw architecture — each type maps to a real primitive (skill → SKILL.md, extension → Plugin SDK, workflow → Lobster, workspace → Workspace-as-Kernel, bridge → openclaw-mcp, prompt → template). New types require an RFC.

**4. Multi-runtime bindings.** A single manifest carries `[runtime.openclaw]`, `[runtime.mcp]`, `[runtime.claude-agent-sdk]`, etc. Each runtime reads only its section. This is the key innovation — one package, N runtimes.

**5. Composition model.** Effectors declare dependencies on other Effectors with semver constraints. Workflows compose skills. Workspaces bundle skills + extensions + prompts. Dependencies are resolved at install time (like npm).

### Manifest Example

```toml
[effector]
name = "github-pr-review"
version = "1.2.0"
type = "skill"
description = "Automated PR review with code analysis"
license = "MIT"
emoji = "🔍"
tags = ["github", "code-review"]
min-spec-version = "0.1.0"

[effector.permissions]
network = true
subprocess = true
env-read = ["GITHUB_TOKEN"]

[runtime.openclaw]
format = "skill.md"
entry = "SKILL.md"

[runtime.openclaw.requires]
bins = ["gh"]
env = ["GITHUB_TOKEN"]

[runtime.mcp]
format = "mcp-tool"
entry = "mcp/adapter.js"
transport = "stdio"
```

### Type Compatibility Matrix

| Dependent ↓ / Dependency → | skill | extension | workflow | workspace | bridge | prompt |
|----|---|---|---|---|---|---|
| **skill** | ✓ | — | — | — | — | ✓ |
| **extension** | ✓ | ✓ | — | — | — | ✓ |
| **workflow** | ✓ | ✓ | ✓ | — | — | ✓ |
| **workspace** | ✓ | ✓ | ✓ | — | — | ✓ |
| **bridge** | ✓ | — | — | — | — | — |
| **prompt** | — | — | — | — | — | ✓ |

## Drawbacks

1. **Another standard.** The XKCD "standards" problem. Mitigated by: backward compatibility (existing skills work unchanged), multi-runtime design (not replacing, wrapping), and building from a real ecosystem (3,286 skills, not zero).

2. **TOML adoption.** Some developers prefer YAML or JSON. Mitigated by: manifest inference (no TOML required for simple skills), and tooling (`create-effector` generates manifests automatically).

3. **Spec maintenance burden.** A formal spec requires ongoing governance. Mitigated by: the RFC process (this repo) and a deliberately small initial scope (v0.1.0 is draft, not stable).

4. **Runtime adoption uncertainty.** Other runtimes may not adopt the standard. Mitigated by: the bridge type (bridges translate, even if runtimes don't adopt natively) and OpenClaw as reference implementation with real traction.

## Alternatives

1. **Extend SKILL.md frontmatter only.** Rejected because: YAML frontmatter can't cleanly express multi-runtime bindings, composition, and permissions. The format would become unwieldy. TOML is better for this.

2. **Use MCP as the universal format.** Rejected because: MCP is a wire protocol, not a packaging format. It doesn't express composition, permissions, workspace configuration, or workflow orchestration. MCP is one binding target, not the whole picture.

3. **Use OCI container format.** Rejected because: Effectors are mostly text files (SKILL.md, pipeline.yml, SOUL.md). Container overhead is massive overkill. The analogy is conceptual (OCI standardized containers; Effectors standardize capabilities), not format-level.

4. **No spec at all — just build tools.** Rejected because: without a shared vocabulary and format, the ecosystem fragments. "Effector" needs a definition that tooling can implement against.

## Unresolved Questions

1. **Registry API.** The spec defines what gets published, not how. The registry API (for ClawHub integration, npm bridging, etc.) needs a separate RFC.

2. **Version negotiation.** When a runtime supports spec v0.1.0 but an Effector requires v0.2.0, what happens? The `min-spec-version` field exists but the negotiation protocol is TBD.

3. **Signed packages.** The security model defines trust levels but doesn't specify a signing mechanism. Should we use Sigstore, PGP, or something else?

4. **Custom types.** The spec defines six types. Should runtimes be able to register custom types? If so, how does discovery work?

5. **Monorepo Effectors.** Can a single repo contain multiple Effectors? The manifest currently assumes one per root. Monorepo support needs design work.

## Implementation Plan

### Phase 1: Spec + Tooling (Current)

- [x] `effector-spec` — v0.1.0 draft (7 spec documents + JSON schema + examples)
- [x] `create-effector` — CLI scaffolder for all 6 types
- [ ] `rfcs` — RFC governance (this document)
- [ ] Integrate `effector.toml` validation into `skill-lint`
- [ ] Update `openclaw-mcp` to read `effector.toml` when present

### Phase 2: Reference Implementations

- [ ] Publish a real skill Effector with both `SKILL.md` and `effector.toml`
- [ ] Build a bridge Effector that translates between OpenClaw and Claude Agent SDK
- [ ] Create a workspace Effector that bundles skills + personality config

### Phase 3: Ecosystem Adoption

- [ ] Propose MCP binding format to the MCP community
- [ ] Build registry API spec (separate RFC)
- [ ] Explore adoption with other runtimes (LangChain, CrewAI, etc.)

---

*This RFC corresponds to the full specification at [effectorHQ/effector-spec](https://github.com/effectorHQ/effector-spec). Discussion happens on the PR for this RFC and in [effectorHQ Discussions](https://github.com/orgs/effectorHQ/discussions).*
