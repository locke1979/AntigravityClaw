# `/ia_orchestrating_swarms` Execution Guide

## Objective

Use `/ia_orchestrating_swarms` to implement AntigravityClaw from the GitHub issue graph without allowing parallel agents to destabilize shared process/session code.

## Orchestrator bootstrap prompt

The orchestrator should begin from this repository and use the following constraints:

```text
Implement AntigravityClaw using the GitHub issue graph as the source of work.
Read README.md, AGENTS.md, docs/ARCHITECTURE.md, docs/DEVELOPMENT_PLAN.md,
docs/DEPLOYMENT.md and docs/TEST_PLAN.md before assigning work.

Respect all Blocked by / Gate relationships. Do not start a dependent wave until its gate passes.
Parallelize only independent issues. Use one integration owner for shared session/process abstractions.
Each implementation issue should produce a small PR with automated tests and exact verification evidence.
Do not use TUI keystroke automation. Use Antigravity headless JSON/stream-json protocol.
Do not introduce automatic git push/merge or dangerously-skip-permissions as a default.
The target openclaw-local-host already has authenticated agy; deployment must verify and reuse it.
```

## Wave allocation

### Wave 0

Single owner plus reviewer:

- openclaw-local-host protocol/capability capture;
- fixture specification based on observed behavior.

This is deliberately not highly parallel because all later adapters depend on these observations.

### Wave 1

Parallel workstreams:

- runtime/package/CI bootstrap;
- fake `agy` fixture;
- initial MCP server skeleton/config schema.

Integration owner reconciles package structure and public interfaces.

### Wave 2

Parallelize by adapter concern after interfaces are frozen:

- one-shot JSON adapter;
- stream-json parser/session process;
- timeout/cancellation/error taxonomy tests.

Shared process lifecycle code must have a single owner.

### Wave 3

Parallel workstreams:

- project registry/config validation;
- path/worktree confinement;
- git diff/status helper;
- verification-command policy.

Security tests are an independent reviewer lane.

### Wave 4

Split MCP tools into independent groups:

- capability/health/project-inspect;
- plan/run/continue;
- session status/cancel/logs;
- diff/test.

Contract/schema reviewer owns consistency across all tool outputs.

### Wave 5

Parallel workstreams:

- structured logging/correlation;
- redaction;
- metrics/failure taxonomy;
- adversarial diagnostics tests.

### Wave 6

Downstream MCP diagnostics can proceed largely independently after the core adapter is stable.

### Wave 7-8

Deployment owner plus test/gate owner. Real openclaw-local-host canary actions should be serialized to prevent agents from racing on the same deployment or worktree.

## PR policy

Preferred mapping: one implementation issue -> one PR.

Every PR description should include:

- issue link;
- architecture impact;
- acceptance criteria checklist;
- exact tests run;
- security implications;
- whether live `agy` was used;
- any deviation from the issue contract.

## Gate policy

Gate issues are verification tasks, not implementation buckets.

A gate may close only when:

- all prerequisite implementation issues are merged/available in the integration branch;
- required commands have actually run;
- evidence is attached;
- failures are classified and either fixed or explicitly accepted by project owner.

If a gate fails, the orchestrator should open focused repair issues rather than silently expanding the gate issue into a large implementation task.

## Conflict avoidance

Agents should claim file ownership before parallel editing. Suggested ownership boundaries:

- `src/agy/**`: process/session integration owner;
- `src/mcp/**`: MCP contract owner;
- `src/projects/**`: project security owner;
- `src/logging/**`: observability owner;
- `test/fixtures/**`: fixture owner;
- `deploy/**`: deployment owner.

If two active issues require the same core file, serialize them unless the integration owner explicitly coordinates the change.

## Stop conditions

The orchestrator must stop/raise a gate failure when:

- real CLI behavior contradicts the assumed protocol;
- openclaw-local-host authentication is unavailable;
- sandbox cannot start and no approved equivalent containment exists;
- secrets appear in logs/responses;
- path confinement can be bypassed;
- child processes survive cancellation unexpectedly;
- structured MCP response contracts become incompatible across tools.


## Binding environment decision

The only allowed execution environments are openclaw-local-host and openclaw-local-host. Use openclaw-local-host as the definitive target for G0, G7, and G8; remote execution is prohibited. The orchestrator must require openclaw-local-host project/worktree allowlist, verification-command allowlist, and approved service-user authentication context before scheduling live work. Issue #4 is blocked until G0 PASS.


## Definitive current-lane binding (2026-08-31)

The current design is **openclaw-local-host-only**. OpenClaw launches AntigravityClaw as a **local MCP stdio** server; AntigravityClaw invokes the existing **`/home/claw/.local/bin/agy` 1.1.22** as a bounded child process using the approved project/worktree canonical path as the child-process `cwd`. There is no current-lane VM, CT, LXC, Proxmox, remote-execution, or network-MCP transport. G0, G7, and G8 are explicitly local-host-only. Historical references elsewhere in this document are retained as history or non-current diagnostic examples and do not authorize this lane.