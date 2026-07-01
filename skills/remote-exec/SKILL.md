---
name: remote-exec
description: Use this skill when the user needs to run repeated shell commands on a remote machine over SSH, including connection checks, stdin scripts, and optional tmux-backed persistent state. Do not use it for local-only commands or workflows that require storing passwords.
license: MIT
requires:
  bins:
    - ssh
  env:
    - REMOTE_EXEC_TARGET
metadata:
  author: vlln
  version: "0.1.0"
---

# Remote Exec

## Trigger Keywords

ssh, remote, remote-exec, rex, remote command, execute on remote, run on server, remote shell

## Capabilities

- Run commands on a remote host via `rex` (SSH with ControlMaster connection multiplexing).
- Check connectivity: `rex --check` prints remote hostname, user, and working directory.
- Pass multi-line scripts on stdin.
- `--tmux` preserves shell state (cd, exports, venv activation) across calls.
- Connection persistence via `ControlPersist` (default 10m, configurable via `REMOTE_EXEC_PERSIST`).

## Workflow

1. Activate `.rex.env` in the current workspace. If missing, create it:

   ```bash
   export REMOTE_EXEC_TARGET="user@host"
   export REMOTE_EXEC_PORT="22"
   export REMOTE_EXEC_PERSIST="10m"
   ```

   Source it before every `rex` call:

   ```bash
   source .rex.env
   ```

2. Verify the connection:

   ```bash
   rex --check
   ```

3. Run commands. Prefer direct `rex` for independent commands; use `--tmux` only when shell state must persist between calls.

### Tmux Mode

Use `--tmux` for tasks needing persistent shell state. Session comes from `REMOTE_EXEC_TMUX_SESSION`; override with `--tmux-session NAME`.

```bash
rex --tmux 'cd /path/to/project'
rex --tmux pwd
rex --tmux git status --short
```

## Gotchas

- `.rex.env` is the activation file for `rex`. Do not use or modify `.env`.
- `--tmux` creates a persistent tmux session on the remote host. Commands accumulate state — `cd` and environment changes persist between calls.
- `--tmux` requires `tmux` installed on the remote host.
- `BatchMode=yes` is forced internally — only SSH keys work, password auth is not supported.
- `ControlPersist` keeps the master SSH connection alive. If the target or port changes, the old socket may cause failures; delete `$REMOTE_EXEC_DIR` (default `$TMPDIR/remote-exec-ssh`) to reset.