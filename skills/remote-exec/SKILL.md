---
name: remote-exec
description: Use this skill when the user needs to run repeated shell commands on a remote machine over SSH, including connection checks, stdin scripts, and optional tmux-backed persistent state. Do not use it for local-only commands or workflows that require storing passwords.
license: MIT
compatibility: Requires OpenSSH locally. Remote tmux is required only when using tmux mode.
metadata:
  skit:
    version: 0.1.0
    requires:
      bins:
        - ssh
      env:
        - REMOTE_EXEC_TARGET
    keywords:
      - ssh
      - remote-command
---

# Remote Exec

## When To Use

Use for repeated remote shell commands over SSH. Prefer normal `rex` (`<path-to-skills>/remote-exec/scripts/rex`) for independent commands; use tmux mode only when command state must persist between calls.

## Workflow

1. Identify the SSH target from the user request, `.env`, or `REMOTE_EXEC_TARGET`. If missing, ask.
2. Check the connection:

   ```bash
   rex --check
   ```

3. If auth fails, check/create a key, then ask the user to run `ssh-copy-id "$REMOTE_EXEC_TARGET"` manually:

   Do not overwrite an existing private key. If `ssh-copy-id` is unavailable, show the public key and tell the user to add it to remote `~/.ssh/authorized_keys`.

4. Run normal commands through `rex`. Use tmux mode only when persistent shell state is needed.

## Usage

```bash
REMOTE_EXEC_TARGET="user@host"
REMOTE_EXEC_TMUX_SESSION="remote-exec"  # when use tmux only
```

Pass shell scripts on stdin to avoid local quoting issues.

```bash
rex <<'EOF'
cd /path/to/project
git status --short
EOF
```

Use `--target user@host` to override `.env` or `REMOTE_EXEC_TARGET`.

## Tmux mode

Use `--tmux` for tasks that need persistent shell state such as `cd`, exported variables, activated environments, or long-lived interactive context. The default session comes from `REMOTE_EXEC_TMUX_SESSION`; override it with `--tmux-session NAME`.

```bash
rex --tmux 'cd /path/to/project'
rex --tmux pwd
rex --tmux git status --short
rex --tmux-session work --tmux pwd
```

## Rules

- Do not ask for or store remote passwords.
- Do not overwrite existing private keys.
- Do not modify unrelated `.env` keys.
- Use stdin scripts for multi-line commands to avoid local quoting bugs.
- Treat tmux as remote state: use it only when the user needs persistent `cd`, exports, activated environments, or long-running interactive context.
