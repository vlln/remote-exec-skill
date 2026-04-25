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
- `REMOTE_EXEC_TARGET` in the environment or nearest `.env`, unless `--target` is passed
- Remote `tmux` only when using `--tmux`

The helper script reads the nearest `.env` file from the current directory or its ancestors. It only imports keys beginning with `REMOTE_EXEC_`.

## Configuration

```sh
REMOTE_EXEC_TARGET=user@example.com
REMOTE_EXEC_PORT=22
REMOTE_EXEC_PERSIST=10m
REMOTE_EXEC_TMUX_SESSION=remote-exec
```

`REMOTE_EXEC_PORT`, `REMOTE_EXEC_PERSIST`, and `REMOTE_EXEC_TMUX_SESSION` are optional. Use `REMOTE_EXEC_TMUX_SESSION` only when tmux mode is needed.

## Usage

Run a connection check:

```sh
skills/remote-exec/scripts/rex --check
```

Run a command:

```sh
skills/remote-exec/scripts/rex hostname
```

Override the target:

```sh
skills/remote-exec/scripts/rex --target user@example.com uname -a
```

Pass multi-line commands on stdin to avoid local quoting issues:

```sh
skills/remote-exec/scripts/rex <<'EOF'
cd /path/to/project
git status --short
EOF
```

Use tmux mode when shell state must persist across calls:

```sh
skills/remote-exec/scripts/rex --tmux-session work --tmux 'cd /srv/app'
skills/remote-exec/scripts/rex --tmux-session work --tmux pwd
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
