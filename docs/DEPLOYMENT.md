# Deployment Plan

## Target

Deploy AntigravityClaw on the existing OpenClaw VM. Antigravity CLI is already installed and authenticated there; deployment must preserve that working installation and cached authentication.

## Recommended V1 topology

Prefer MCP stdio launched directly by OpenClaw:

```text
OpenClaw
  -> node /opt/antigravityclaw/dist/server.js
       -> agy headless child process
```

This avoids exposing a network listener and keeps the initial trust boundary local to the VM.

A Streamable HTTP transport can be added later only if remote access is required.

## Filesystem layout

Suggested:

```text
/opt/antigravityclaw/          application release
/etc/antigravityclaw/          non-secret configuration
/var/lib/antigravityclaw/      session metadata/worktrees if managed here
/var/log/antigravityclaw/      bounded redacted logs, if not journald
```

The service account must use the same user context or an explicitly validated credential context in which the existing `agy` authentication works. Do not copy authentication files blindly between users.

## Configuration

Configuration should define:

- `agyExecutable` (default discovered from PATH);
- project registry;
- workspace/worktree root;
- default sandbox behavior;
- maximum concurrent Antigravity processes;
- default/maximum turn timeout;
- cancellation grace period;
- allowed verification commands per project;
- log retention/size limit;
- redaction patterns;
- optional model/agent allowlists.

Do not store API tokens or Antigravity authentication secrets in the repository.

## Deployment stages

### D0 — Preflight

On the target user account:

- `command -v agy`;
- capture `agy` version;
- run a harmless authenticated headless JSON call;
- run a sandboxed harmless call;
- verify Node runtime and repository/package prerequisites;
- record permissions on project/worktree roots.

Fail deployment if the existing `agy` environment is not healthy. Do not attempt automatic re-authentication.

### D1 — Install application

- checkout/build a tagged commit or release artifact;
- install production dependencies with lock-file enforcement;
- copy generated runtime files to `/opt/antigravityclaw`;
- create configuration under `/etc/antigravityclaw`;
- validate config before enabling OpenClaw integration.

### D2 — Local MCP smoke

Use an MCP inspector/test client or equivalent local harness to verify:

- server initialization;
- tool discovery;
- `agy_capabilities`;
- `agy_health`;
- registered project inspection.

### D3 — OpenClaw integration

Configure OpenClaw to launch AntigravityClaw as an MCP stdio server. Keep the first enabled tool set minimal until canary gates pass.

### D4 — Canary

Run against a disposable repository/worktree. Verify plan, implementation, diff, test, continuation, cancellation, and redacted logging.

### D5 — Broaden project registry

Register real repositories one at a time after the canary succeeds. Each project must declare its test/build commands and any special policy constraints.

## Rollback

Rollback must not modify the Antigravity installation/auth state.

1. Disable/remove the AntigravityClaw MCP entry from OpenClaw.
2. Stop any managed Antigravity child processes.
3. Restore the previous AntigravityClaw release/config.
4. Preserve logs and session metadata for diagnosis.
5. Re-run the local MCP smoke suite before re-enabling.

## Operational limits for V1

- no automatic `git push` or merge;
- no generic shell MCP tool;
- no arbitrary external workspace paths;
- no `--dangerously-skip-permissions` default;
- cap concurrent `agy` processes;
- cap log size and response excerpts;
- bound every child process with a timeout/cancellation policy.
