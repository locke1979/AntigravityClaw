# Architecture

## Context

Antigravity CLI already provides the programmatic primitives AntigravityClaw needs:

- one-shot headless execution with `-p`;
- JSON and NDJSON event output;
- structured output via `--json-schema`;
- persistent multi-turn sessions via stdin with `--input-format stream-json --output-format stream-json`;
- `conversation_id` for continuation;
- model, effort, and agent selection;
- sandbox mode;
- documented terminal statuses and process exit behavior.

AntigravityClaw therefore acts as a protocol adapter and policy boundary, not as a TUI automation layer.

## Components

```text
OpenClaw
  |
  | MCP stdio (V1)
  v
AntigravityClaw MCP Server
  |
  +-- Tool Router / Schemas
  +-- Project Registry
  +-- Policy Engine
  +-- Session Registry
  +-- Result Normalizer
  +-- Event / Log Store
  |
  v
Agy Process Adapter
  |
  +-- one-shot JSON runs
  +-- long-lived stream-json sessions
  +-- timeout / cancellation
  +-- stdout NDJSON parser
  +-- stderr classifier
  |
  v
agy CLI
  |
  v
registered workspace or managed worktree
```

## Session identities

Keep three identities distinct:

1. `request_id`: one MCP invocation.
2. `session_id`: AntigravityClaw-owned lifecycle record.
3. `conversation_id`: Antigravity-owned conversation identifier.

A session record should contain at least:

```json
{
  "session_id": "agc_...",
  "conversation_id": "...",
  "project": "MediaManagement",
  "workspace": "/srv/openclaw/worktrees/...",
  "state": "idle|running|completed|failed|cancelled",
  "pid": 1234,
  "created_at": "...",
  "updated_at": "...",
  "last_result": {},
  "log_refs": []
}
```

## Invocation modes

### One-shot

Use for capability probes, planning, bounded read-only inspection, and stateless operations:

```bash
agy -p "..." --output-format json --json-schema <schema> --sandbox --print-timeout <duration>
```

### Persistent session

Use for implementation loops where later prompts should reuse warmed context:

```bash
agy --input-format stream-json --output-format stream-json --sandbox
```

Send one `user` event per turn and wait for that turn's terminal `result` event before sending another.

## Normalized result contract

Every execution-oriented MCP tool should normalize Antigravity output into a shape similar to:

```json
{
  "request_id": "req_...",
  "session_id": "agc_...",
  "conversation_id": "...",
  "status": "completed|blocked|failed|cancelled",
  "agy_status": "SUCCESS",
  "process_exit_code": 0,
  "permission_blocked": false,
  "response": "...",
  "structured_output": {},
  "changed_files": [],
  "verification": [],
  "usage": {},
  "warnings": [],
  "log_refs": []
}
```

Do not infer task completion from `process_exit_code == 0` alone. Headless permission requests can be soft-denied while the CLI process still exits successfully.

## Initial MCP tools

### `agy_capabilities`

Return detected CLI version, supported wrapper capabilities, available models/agents where safe to query, sandbox availability, and gateway version.

### `agy_health`

Verify executable discovery, cached authentication with a harmless headless probe, JSON parsing, workspace access, and sandbox invocation.

### `agy_project_inspect`

Return the registered project metadata, git status, root/worktree path, allowed verification commands, and detected rule files (`AGENTS.md`, `GEMINI.md`).

### `agy_plan`

Run a read-oriented planning prompt and require structured plan output: summary, files/components, risks, acceptance checks, proposed tests, and whether implementation is safe to start.

### `agy_run`

Execute a bounded implementation turn inside the approved workspace. Inputs must include project and task/acceptance criteria. Return normalized evidence.

### `agy_continue`

Send an additional turn to an existing gateway session/conversation.

### `agy_session_status` / `agy_session_cancel`

Inspect or terminate a managed running process without exposing generic process control.

### `agy_diff`

Return git diff/stat from the managed workspace. Implement this in the gateway rather than by trying to automate Antigravity's `/diff` TUI panel.

### `agy_test`

Run only project-registered verification commands and return command, duration, exit code, stdout/stderr excerpts, and log reference.

### `agy_session_logs`

Return redacted event/diagnostic excerpts for a managed session.

## Security boundaries

- Registered project roots only.
- Resolve/canonicalize paths and reject traversal/symlink escapes.
- No unrestricted shell MCP tool.
- No direct exposure of Antigravity auth material.
- Default sandbox enabled for Antigravity command execution.
- Gateway policy must be at least as restrictive as Antigravity's own permissions.
- Redact likely credentials from stderr, tool parameters, environment-derived values, and persisted logs.
- Reject requests that attempt to set arbitrary environment variables unless explicitly allowlisted.

## Downstream MCP diagnostics

Antigravity itself can consume MCP servers from global/workspace configuration. Diagnostics for those servers are useful but secondary to the execution gateway. Implement them after the core execution/session contract so failures can be classified as:

1. AntigravityClaw/gateway failure;
2. Antigravity CLI failure;
3. Antigravity permission block;
4. downstream MCP server failure;
5. task/test failure.


## Deployment boundary and trust contract

V1 may run only as openclaw-local-host or proxmox-local-lxc. The definitive G0/G7/G8 environment is proxmox-local-lxc, with OpenClaw invoking the gateway locally over MCP stdio. There is no remote VM/VM3 execution path.

The LXC service configuration must contain an explicit allowlist of canonical project roots or managed worktree roots, plus an allowlist of bounded verification commands per project. Requests outside either allowlist are rejected before process creation. agy authentication belongs to the service user/context already approved for the LXC; credentials are neither copied nor re-authenticated by deployment. The gateway must redact secrets and keep the Antigravity policy at least as restrictive as the LXC policy.

G0 is definitive only when proven in the LXC. G7 and G8 run in that same LXC; issue #4 cannot start until G0 is PASS.
