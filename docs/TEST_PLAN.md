# Test Plan

## Test philosophy

Most CI tests must not require live Antigravity credentials. Use a deterministic fake `agy` process to exercise protocol, timeout, failure, and redaction behavior. Reserve live `agy` tests for the OpenClaw local host/LXC deployment gates.

## Test layers

### 1. Unit tests

Cover:

- config parsing/defaults;
- project registry validation;
- path canonicalization;
- command allowlist matching;
- NDJSON parsing;
- Antigravity status normalization;
- stderr classification;
- permission-soft-deny detection;
- redaction;
- session state transitions;
- result schema validation.

### 2. Process-adapter integration tests with fake CLI

The fake `agy` fixture must support scenarios selected by arguments/environment or fixture input:

- one-shot JSON success;
- structured output;
- streaming init/step/result;
- multiple turns in one process;
- emitted `conversation_id`;
- delayed response;
- stdout chunking;
- stderr warnings;
- permission soft-denial with exit 0;
- malformed JSON line;
- terminal `ERROR` with exit 1;
- `CANCELED`, `INTERRUPTED`, `WAITING`, `INVALID`;
- child that ignores SIGTERM to test escalation;
- oversized output/logs;
- secrets embedded in parameters/output for redaction tests.

### 3. MCP contract tests

Start AntigravityClaw and test through an MCP client:

- initialize/list tools;
- valid and invalid tool arguments;
- stable result schemas;
- session creation/continuation/status/cancel;
- project confinement errors;
- allowlisted and denied verification commands;
- log retrieval with redaction;
- concurrent sessions up to configured limit;
- rejection/backpressure above the limit.

### 4. Git/worktree tests

Use temporary git repositories to verify:

- clean/dirty status;
- diff capture;
- changed-file detection;
- managed worktree creation/removal if implemented;
- untracked files;
- symlink/path escape defense;
- no operations outside registered root.

### 5. Live Antigravity LXC smoke tests

Run only on the target LXC or an explicitly configured self-hosted runner:

- discover `agy`;
- authenticated harmless `-p` JSON call;
- `--json-schema` call;
- `stream-json` start and two turns;
- `--conversation` continuation where appropriate;
- `--sandbox` harmless command path;
- forced invalid model or equivalent deterministic CLI error;
- permission-soft-deny probe using a harmless operation requiring approval;
- timeout/cancellation against a safe test prompt/process.

Never put authentication material in CI artifacts.

## Acceptance workflow fixture

Create a tiny temporary repository containing a deliberately failing function and tests. The fake/live acceptance sequence should exercise:

1. `agy_project_inspect`;
2. `agy_plan`;
3. `agy_run` to modify the fixture;
4. `agy_diff`;
5. `agy_test`;
6. `agy_continue` with test feedback;
7. final passing test;
8. `agy_session_logs`.

## Negative/security suite

Required cases:

- project name not registered;
- absolute path outside registry;
- `../` traversal;
- symlink escape;
- command injection attempt in verification command;
- arbitrary environment override attempt;
- secret-like strings in stdout/stderr/tool parameters;
- malformed/oversized stream events;
- hostile Antigravity free-text response attempting to alter gateway policy;
- concurrent session exhaustion;
- repeated child-process crash;
- stale session recovery after server restart.

## Release evidence

Each gate issue must record:

- exact test command;
- pass/fail status;
- relevant CI run or log reference;
- target commit SHA;
- unresolved deviations, if any.

A swarm agent may not mark a gate complete based only on a prose claim that tests passed.


## Environment gates

Live agy validation is permitted only on the approved local host or, definitively, in proxmox-local-lxc. G0, G7, and G8 evidence must come from the LXC; remote VM/VM3 probes are out of scope. Tests must verify project/worktree confinement, verification-command allowlisting, authentication reuse without token copying/login, redaction, and rollback. Issue #4 remains blocked until G0 PASS.
