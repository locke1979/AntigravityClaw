# AntigravityClaw

AntigravityClaw is an MCP execution gateway that lets OpenClaw delegate bounded software-engineering work to the Google Antigravity CLI already installed and authenticated on the OpenClaw local host/LXC.

The server does **not** automate the Antigravity TUI. It uses Antigravity CLI's documented headless protocol (`--output-format json|stream-json`, `--input-format stream-json`, `--conversation`, `--json-schema`, `--sandbox`) and exposes stable, higher-level MCP tools to OpenClaw.

## Goal

Provide a deterministic, auditable execution lane:

```text
OpenClaw / orchestrator
        |
        | MCP
        v
AntigravityClaw
  - project registry
  - session manager
  - policy / timeout enforcement
  - event + stderr capture
  - structured result normalization
        |
        | agy headless stream-json
        v
Google Antigravity CLI
        |
        v
approved project workspace / git worktree
```

## Design principles

- Use Antigravity's native headless protocol; never drive slash commands by terminal keystrokes.
- Keep OpenClaw at the orchestration/control layer and Antigravity at the implementation/execution layer.
- Restrict work to registered project roots or managed git worktrees.
- Prefer `request-review`/scoped permission rules and `--sandbox`; do not use `--dangerously-skip-permissions` in normal operation.
- Treat Antigravity `stderr`, stream events, exit code, and terminal `status` as separate signals. In particular, a headless permission soft-denial can coexist with process exit code 0.
- Persist `conversation_id` so later MCP calls can continue the same Antigravity context.
- Return machine-readable results, verification evidence, changed files, and artifact/log references.
- Make every substantive implementation task end in local verification commands.

## Planned MCP surface

Initial V1 tools:

- `agy_capabilities`
- `agy_health`
- `agy_project_inspect`
- `agy_plan`
- `agy_run`
- `agy_continue`
- `agy_session_status`
- `agy_session_cancel`
- `agy_diff`
- `agy_test`
- `agy_session_logs`

Later:

- worktree lifecycle helpers
- fork/alternative implementation workflows
- MCP configuration diagnostics for Antigravity's own downstream MCP servers
- policy/risk summaries
- deployment metrics and circuit breakers

## Source of truth

See:

- `docs/ARCHITECTURE.md`
- `docs/DEVELOPMENT_PLAN.md`
- `docs/DEPLOYMENT.md`
- `docs/TEST_PLAN.md`
- `docs/ORCHESTRATION.md`
- `AGENTS.md`

Official Antigravity documentation used by this project:

- https://antigravity.google/docs/home/
- https://antigravity.google/docs/cli/headless/
- https://antigravity.google/docs/cli/permissions/
- https://antigravity.google/docs/cli/sandbox/
- https://antigravity.google/docs/cli/mcp/
- https://antigravity.google/docs/cli/best-practices/

## Development target

The implementation is intended for the approved OpenClaw local host or Proxmox local LXC where `agy` is already installed and authenticated. Bootstrap work should verify that environment, not replace or re-authenticate it.

The repository is planned as a Node.js MCP server with an adapter around the `agy` child process and a testable fake-CLI fixture for CI.


## V1 environment decision

The only permitted runtime environments are openclaw-local-host and proxmox-local-lxc. The definitive V1 deployment and G0 validation target is proxmox-local-lxc; remote execution is not part of this architecture. The LXC must use an explicit project/worktree allowlist, bounded verification-command allowlists, and its own validated service-user authentication context. Do not copy tokens or perform automatic login. G7 and G8 are LXC gates. Issue #4 remains blocked until G0 is PASS.
