# effectorHQ RFCs

[![License: CC-BY-4.0](https://img.shields.io/badge/license-CC--BY--4.0-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/effectorHQ/.github/blob/main/CONTRIBUTING.md)

**[中文文档 →](./README.zh.md)**

---

This repository tracks **Requests for Comments (RFCs)** for the effectorHQ ecosystem — the formal process for proposing substantive changes to the Effector specification, tooling, and governance.

## When to Write an RFC

Not everything needs an RFC. Use this process for:

- Changes to the [Effector Spec](https://github.com/effectorHQ/effector-spec) (new types, manifest fields, lifecycle stages)
- New cross-cutting features that affect multiple repos
- Architectural decisions with long-term consequences
- Breaking changes to published APIs or formats
- Governance changes (decision-making, maintainer roles, community processes)

For bug fixes, documentation improvements, and localized changes — just open a PR on the relevant repo.

## RFC Process

### 1. Propose

Fork this repo. Copy `0000-template.md` to `text/NNNN-my-proposal.md` (use the next available number). Fill in the template. Open a pull request.

### 2. Discuss

The community reviews the RFC via PR comments. The author iterates on the proposal based on feedback. There is no fixed timeline — discussion continues until rough consensus emerges.

### 3. Decide

A maintainer marks the RFC as either:

- **Accepted** — Merged into `text/`. Implementation can proceed.
- **Postponed** — Good idea, wrong time. The PR stays open for future revisiting.
- **Rejected** — The PR is closed with a summary of reasons.

### 4. Implement

Accepted RFCs get a tracking issue in the relevant repo(s). The RFC author is not obligated to implement it — anyone can pick it up.

## Active RFCs

| RFC | Title | Status |
|-----|-------|--------|
| [0001](text/0001-effector-spec.md) | The Effector Specification v0.1.0 | Draft |

## Inspiration

This process draws from [Rust RFCs](https://github.com/rust-lang/rfcs), [React RFCs](https://github.com/reactjs/rfcs), and [Ember RFCs](https://github.com/emberjs/rfcs). Lightweight by design — we prefer building to writing, but some decisions need shared context first.

## License


This project is currently licensed under the Apache 2.0 License 。

[CC-BY-4.0](./LICENSE) — Same as all effectorHQ documentation.
