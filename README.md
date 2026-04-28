# remote-exec

Agent Skill for running repeated shell commands on a remote machine over SSH with OpenSSH connection reuse.

This repository stores skills under `skills/`. Each skill follows the [Agent Skills specification](https://agentskills.io/specification) and can be used by skills-compatible agents.

## Skills

| Skill | Description |
|-------|-------------|
| [`remote-exec`](skills/remote-exec) | Run repeated remote shell commands over SSH, with stdin scripts and optional tmux-backed persistent state. |

## Installation

Recommended: use `skit` for reproducible installs, lock files, and requirement diagnostics.

### skit

Install this skill:

```sh
skit install ./skills/remote-exec
```

Install all discoverable skills from this repository:

```sh
skit install . --all
```

### Manual

Copy `skills/remote-exec` into your agent's skills directory, then restart the agent if required.

Common locations:

- Codex CLI: `~/.codex/skills/remote-exec`
- Claude Code: `.claude/skills/remote-exec` in the project, or the configured user skills directory
- OpenCode: `~/.opencode/skills/remote-exec`

## Requirements

- Local OpenSSH client: `ssh`
- SSH access to the target host
- `REMOTE_EXEC_TARGET` in the environment, unless `--target` is passed
- Remote `tmux` only when using `--tmux`

The helper script reads configuration from the current environment only. For repeated use, create a project-local `.rex.env`, source it in your shell, and let your agent check or update it when remote-exec is needed.

## Configuration

```sh
export PATH="/path/to/skills/remote-exec/scripts:$PATH"
export REMOTE_EXEC_TARGET=user@host
export REMOTE_EXEC_PORT=22
export REMOTE_EXEC_PERSIST=10m
export REMOTE_EXEC_TMUX_SESSION=remote-exec
```

`REMOTE_EXEC_PORT`, `REMOTE_EXEC_PERSIST`, and `REMOTE_EXEC_TMUX_SESSION` are optional. Use `REMOTE_EXEC_TMUX_SESSION` only when tmux mode is needed.

Activate the configuration before running commands:

```sh
source .rex.env
```

## Usage

Run a connection check:

```sh
rex --check
```

Run a command:

```sh
rex hostname
```

Override the target:

```sh
rex --target user@host uname -a
```

Pass multi-line commands on stdin to avoid local quoting issues:

```sh
rex <<'EOF'
cd /path/to/project
git status --short
EOF
```

Use tmux mode when shell state must persist across calls:

```sh
rex --tmux-session work --tmux 'cd /srv/app'
rex --tmux-session work --tmux pwd
```

## Validation

Inspect the skill package:

```sh
skit inspect ./skills/remote-exec
```

Check the helper CLI:

```sh
bash skills/remote-exec/scripts/rex --help
```

## License

MIT
