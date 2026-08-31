# AGENTS.md

## Mission

Build AntigravityClaw as a small, secure MCP execution gateway between OpenClaw and the Google Antigravity CLI.

## Non-negotiable architecture

1. Use Antigravity CLI headless mode. Do not automate the TUI with pseudo-terminal keystrokes.
2. The core adapter must support `agy --input-format stream-json --output-format stream-json` for warm multi-turn sessions.
3. Persist and expose Antigravity `conversation_id` separately from AntigravityClaw session IDs.
4. Run tasks only inside an explicitly registered project root or managed git worktree.
5. Default to sandboxed/scoped execution. Never make `--dangerously-skip-permissions` the normal path.
6. Capture stdout, stderr, process exit code, stream events, terminal Antigravity status, and permission-denial evidence independently.
7. MCP handlers must return normalized structured results; callers must not parse free-form Antigravity prose to determine success.
8. No generic arbitrary-shell MCP tool. Verification commands must be bounded by project configuration/policy.
9. Secrets must never be emitted in logs, MCP responses, fixtures, or test snapshots.
10. Every feature issue must include tests and a gate before dependent issues proceed.

## Engineering workflow

Use an explore -> plan -> implement -> verify sequence.

Before changing behavior:

- inspect the relevant source and tests;
- identify the issue acceptance criteria;
- write or update tests for the contract;
- implement the smallest coherent slice;
- run unit tests, integration tests, lint/type checks, and any issue-specific acceptance command;
- include actual command output/result in the completion report.

## Swarm execution

This repository is designed to be implemented via `/ia_orchestrating_swarms`.

The orchestrator should:

- treat GitHub issues as the task graph;
- respect `Blocked by` relationships and gate issues;
- parallelize only independent workstreams;
- assign one integration owner for shared process/session abstractions;
- require tests before marking an issue complete;
- stop a dependent wave when a gate fails;
- prefer small PRs mapped 1:1 to implementation issues;
- avoid parallel edits to the same core files unless explicitly coordinated.

Recommended concurrent roles:

- CLI protocol researcher / fixture owner
- MCP contract + schema owner
- session/process lifecycle owner
- project/worktree security owner
- observability/redaction owner
- deployment/systemd owner
- test/integration gate owner

## Definition of done

A feature is done only when:

- acceptance criteria are satisfied;
- automated tests cover success and failure paths;
- no known secret leakage is present;
- relevant documentation is updated;
- CI is green;
- the issue's gate criteria are demonstrably met.


## V1 environment contract

- Permitted environments are only openclaw-local-host and proxmox-local-lxc.
- The definitive G0, G7, and G8 target is proxmox-local-lxc; remote VM/VM3 execution is prohibited.
- The gateway must enforce an explicit project/worktree path allowlist and per-project verification-command allowlist.
- Authentication is owned by the approved service context; never copy tokens or trigger automatic login.
- Issue #4 is blocked until G0 PASS is evidenced in the definitive LXC target.
