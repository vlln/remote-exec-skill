<h1 align="center">remote-exec</h1>

<p align="center">
  <strong>Run repeated shell commands on a remote machine over SSH with OpenSSH connection reuse.</strong><br/>
  Execute shell commands remotely with stdin scripts and optional tmux-backed persistent state. No passwords, just keys.
</p>

<p align="center">
  <a href="https://github.com/vlln/remote-exec-skill/stargazers"><img src="https://badgen.net/github/stars/vlln/remote-exec-skill?label=%E2%98%85" alt="GitHub stars" /></a>
  <img src="https://badgen.net/badge/license/MIT/blue" alt="MIT" />
  <img src="https://badgen.net/badge/spec/Agent%20Skills/8257D0" alt="Agent Skills spec" />
</p>

---

## Installation

### [skit](https://github.com/vlln/skit) (Recommended)

```bash
skit install https://github.com/vlln/remote-exec-skill/tree/main/skills/remote-exec
```

### Manually

| Agent | Command |
|-------|---------|
| **Claude Code** | `cp -r skills/remote-exec .claude/skills/` |
| **Codex** | `cp -r skills/remote-exec ~/.codex/skills/` |
| **OpenCode** | `git clone https://github.com/vlln/remote-exec-skill.git ~/.opencode/skills/remote-exec-skill` |
| **Kimi** | `cp -r skills/remote-exec ~/.kimi/skills/` |

---

## Skills

| Skill | Description |
|-------|-------------|
| [remote-exec](skills/remote-exec/SKILL.md) | Run repeated remote shell commands over SSH, with stdin scripts and optional tmux-backed persistent state. |

## Requirements

- Local OpenSSH client: `ssh`
- SSH access to the target host
- `REMOTE_EXEC_TARGET` in the environment, unless `--target` is passed
- Remote `tmux` only when using `--tmux`

The helper script reads configuration from the current environment only. For repeated use, create a project-local `.rex.env`, source it in your shell, and let your agent check or update it when remote-exec is needed.

---

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