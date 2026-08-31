# Development Plan

## Delivery strategy

Implement AntigravityClaw in gated waves. `/ia_orchestrating_swarms` may parallelize tasks inside a wave only when their dependencies do not overlap.

The target environment is the existing OpenClaw VM, where Antigravity CLI is already installed and authenticated. Do not add re-authentication or replace the installed CLI as part of V1.

## Wave 0 — Environment contract and protocol capture

Deliverables:

- verify `agy` executable/version on target VM;
- harmless authenticated headless JSON smoke test;
- verify `stream-json` session behavior;
- verify `--sandbox` works on the VM;
- capture representative success/error/permission-soft-deny event fixtures;
- record actual CLI capabilities used by the gateway.

### Gate G0 — CLI viability

Pass only if:

- authenticated headless call succeeds without interactive login;
- JSON output parses deterministically;
- streaming session emits `init`, zero or more `step_update`, and terminal `result` events;
- a forced error produces a non-success Antigravity status and is classifiable;
- a permission-requiring operation demonstrates the documented soft-deny behavior without being misclassified as task success;
- sandbox invocation can start.

No implementation wave may depend on undocumented TUI automation.

## Wave 1 — Repository/runtime foundation

Deliverables:

- Node.js package/runtime bootstrap;
- MCP server entrypoint;
- configuration loader;
- logger with redaction hook;
- test runner, lint/type-check policy, CI workflow;
- fake `agy` executable/fixture for deterministic tests.

### Gate G1 — deterministic local harness

Pass only if:

- `npm test` (or selected equivalent) runs in CI without real Antigravity credentials;
- fake CLI can emit JSON, NDJSON, stderr, delays, malformed lines, and exit codes;
- MCP server starts and answers a basic capability call.

## Wave 2 — Agy process adapter

Deliverables:

- one-shot JSON adapter;
- persistent stream-json process adapter;
- NDJSON parser and event dispatcher;
- stdout/stderr separation;
- timeout/cancel/cleanup;
- normalized status/error model;
- conversation ID extraction.

### Gate G2 — protocol fidelity

Pass only if tests cover:

- normal one-shot success;
- structured output success;
- streaming multi-turn session;
- malformed NDJSON;
- process non-zero exit;
- terminal `ERROR`, `CANCELED`, `INTERRUPTED`, and `WAITING` statuses;
- timeout and SIGTERM/SIGKILL escalation;
- soft-denied permission signal with exit 0;
- no zombie child process after cancellation/failure.

## Wave 3 — Project registry and workspace security

Deliverables:

- project registry schema;
- canonical path validation;
- optional managed worktree allocation;
- project-level verification allowlist;
- git status/diff helpers;
- `AGENTS.md`/`GEMINI.md` discovery.

### Gate G3 — workspace confinement

Pass only if:

- registered project access succeeds;
- unregistered roots are rejected;
- `..`, symlink, and alternate-path traversal attempts cannot escape the project/worktree boundary;
- git helper operations act only on the resolved project root;
- verification commands outside the allowlist are rejected.

## Wave 4 — MCP execution contract

Deliverables:

- `agy_capabilities`;
- `agy_health`;
- `agy_project_inspect`;
- `agy_plan` with structured schema;
- `agy_run`;
- `agy_continue`;
- `agy_session_status`;
- `agy_session_cancel`;
- `agy_diff`;
- `agy_test`;
- `agy_session_logs`.

### Gate G4 — OpenClaw usable API

Pass only if an MCP client can execute an end-to-end fake project workflow:

1. inspect project;
2. request plan;
3. run implementation;
4. continue same Antigravity conversation;
5. inspect diff;
6. run allowlisted tests;
7. retrieve logs/status;
8. cancel a deliberately long-running session.

Every response must validate against its documented schema.

## Wave 5 — Observability, redaction, and failure taxonomy

Deliverables:

- structured event log;
- request/session correlation IDs;
- bounded log retention;
- secret redaction;
- failure classes: gateway / CLI / permission / downstream MCP / verification / timeout / cancellation;
- basic metrics counters and durations.

### Gate G5 — safe diagnostics

Pass only if seeded tokens, bearer strings, API keys, and environment secrets are absent from:

- MCP responses;
- persisted event logs;
- stderr excerpts;
- test snapshots;
- failure messages.

## Wave 6 — Downstream Antigravity MCP diagnostics

Deliverables:

- parse global/workspace `mcp_config.json` safely;
- list/inspect effective servers;
- diagnose executable/path/config issues;
- optional direct MCP initialization/tool-list probe for stdio servers;
- redact configured headers/env values.

This extends existing issue #1 but should be implemented as AntigravityClaw functionality, not as a patch to Google's `agy` executable.

### Gate G6 — MCP diagnosis

Pass only if fixtures cover local stdio, remote URL, disabled server, disabled tools, missing executable, malformed config, and secret-bearing configuration without leakage.

## Wave 7 — Target VM deployment

Deliverables:

- install/package procedure;
- runtime config location;
- systemd unit or OpenClaw-managed stdio launch contract;
- health command;
- log location/rotation;
- rollback procedure;
- OpenClaw MCP configuration snippet.

### Gate G7 — target VM smoke

Pass only if the deployed server on the OpenClaw VM can:

- locate the already-installed authenticated `agy`;
- complete `agy_health`;
- inspect a registered test project;
- run a sandboxed harmless task;
- return a structured result to an MCP client;
- restart without losing persisted completed session metadata;
- avoid requiring new interactive authentication.

## Wave 8 — Real repository canary

Use a low-risk dedicated fixture/canary repository or a disposable worktree.

### Gate G8 — production readiness

Pass only if:

- plan -> implement -> diff -> test -> continue workflow succeeds;
- deliberate test failure is reported as verification failure, not gateway failure;
- deliberate policy violation is blocked;
- cancellation cleans the child process;
- logs contain enough correlation evidence to diagnose the run;
- no unreviewed push/merge operation occurs.

## Release criterion

V1 is releasable only when G0-G8 pass and all P0 blocking/high-priority issues are closed. Downstream MCP diagnostics may ship as V1.1 if G0-G5 and G7-G8 are complete.
