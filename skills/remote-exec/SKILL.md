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

## When To Use

Use for repeated remote shell commands over SSH. Prefer direct `rex` commands for independent commands; use tmux mode only when command state must persist between calls.

## Workflow

1. Use `.rex.env` in the current workspace as the activation file. If it is missing, create it with the `rex` command on `PATH` and known `REMOTE_EXEC_*` values:

   ```bash
   export REMOTE_EXEC_TARGET="user@host"
   export REMOTE_EXEC_PORT="22"
   export REMOTE_EXEC_PERSIST="10m"
   ```

   Ensure `rex` is available on `PATH` before sourcing.

2. Identify the SSH target from the user request, an already sourced `.rex.env`, or `REMOTE_EXEC_TARGET`. If missing, ask.
3. Activate the configuration before using `rex`:

   ```bash
   source .rex.env
   ```

   Tell the user they can also source this file in their shell to avoid typing the target repeatedly.

4. Check the connection:

   ```bash
   rex --check
   ```

5. If auth fails, check/create a key, then ask the user to run `ssh-copy-id "$REMOTE_EXEC_TARGET"` manually.

   Do not overwrite an existing private key. If `ssh-copy-id` is unavailable, show the public key and tell the user to add it to remote `~/.ssh/authorized_keys`.

6. Run normal commands through `rex`. Use tmux mode only when persistent shell state is needed.

## Usage

Set the target and optional tmux session:

```bash
export REMOTE_EXEC_TARGET="user@host"
export REMOTE_EXEC_TMUX_SESSION="remote-exec"  # only when using tmux
```

Prefer storing these exports in `.rex.env` and sourcing it before running `rex`.

Pass shell scripts on stdin to avoid local quoting issues:

```bash
rex <<'EOF'
cd /path/to/project
git status --short
EOF
```

Use `--target user@host` to override `REMOTE_EXEC_TARGET`.

## Tmux Mode

Use `--tmux` for tasks that need persistent shell state such as `cd`, exported variables, activated environments, or long-lived interactive context. The default session comes from `REMOTE_EXEC_TMUX_SESSION`; override it with `--tmux-session NAME`.

```bash
rex --tmux 'cd /path/to/project'
rex --tmux pwd
rex --tmux git status --short
rex --tmux-session work --tmux pwd
```

## Gotchas

- Do not ask for or store remote passwords.
- Do not overwrite existing private keys.
- Use `.rex.env` for `rex` activation and configuration; do not read or modify `.env`.
- Use stdin scripts for multi-line commands to avoid local quoting bugs.
- Treat tmux as remote state: use it only when the user needs persistent `cd`, exports, activated environments, or long-running interactive context.