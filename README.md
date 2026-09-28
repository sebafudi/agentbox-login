# Agentbox Login

A terminal home screen for SSH + Zellij: start coding agents, jump between sessions, search by topic or command, and watch CPU/RSS. Built with Go and Bubble Tea; **not** a replacement SSH server.

### Workspace · start or rejoin

![Spacious workspace with launch actions and recent sessions](screenshots/home.png)

### Search · sessions and resource history

![Spacious session search with CPU and RSS history charts](screenshots/search.png)

### Preferences · filters, density, history span

![Spacious preferences with density selected](screenshots/preferences.png)

*All screenshots show synthetic sessions in Spacious density. No live machine data is pictured.*

## Get started

Requires Linux (`/proc` for metrics), Go 1.26, and [Zellij](https://zellij.dev/). [omp](https://github.com/can1357/oh-my-pi) and Claude Code are optional; their launch actions need the corresponding commands on `PATH`. The launcher starts new sessions in `~/code`.

```sh
git clone https://github.com/sebafudi/agentbox-login.git
cd agentbox-login
mise install                  # or provide Go 1.26 yourself
mkdir -p ~/.local/bin ~/code
go build -p 2 -o ~/.local/bin/agentbox-login .
~/.local/bin/agentbox-login
```

`/` searches; `p` opens preferences; `o/r` and `c/l` start/resume omp and Claude; `s` creates a Zellij shell; `h` exits to Bash. Arrows + Enter or mouse clicks open sessions. Ctrl-G/B/V toggle agents/shells/saved; Ctrl-T cycles the history window. Esc on the home screen requests logout (exit code 2). Settings persist under `~/.local/state/agentbox-login/`.

To launch on interactive SSH Bash logins, add this to `~/.bashrc` (with `~/.local/bin` on `PATH`):

```bash
if [[ -n $SSH_CONNECTION && -z $ZELLIJ && -z $NO_ZELLIJ && -t 0 && -t 1 && $TERM != dumb ]]; then
  agentbox-login
  case $? in 2) exit 0 ;; esac
fi
```

This leaves `sshd` alone. If the binary fails, Bash remains available; set `NO_ZELLIJ=1` to bypass the workspace. `agentbox-login --refresh` can run periodically to keep the session cache warm; `agentbox-login --find-agent` opens the search-only picker inside Zellij. To bind the picker to `Ctrl-B`, `Ctrl-F`, add this to Zellij's `keybinds { normal { ... } }` block (replace `YOU`):

```kdl
bind "Ctrl f" {
    Run "/home/YOU/.local/bin/agentbox-login" "--find-agent" {
        floating true
        close_on_exit true
        name "find sessions"
    };
    SwitchToMode "Locked";
}
```

Session state and foreground command detection are best-effort; CPU/RSS history is sampled from `/proc`. Snapshots can contain session names, paths and commands: **never publish `~/.local/state/agentbox-login/`**.

`cmd/agentbox-facts` is a separate, host-specific inventory collector, **not required** for the workspace. Its generated Markdown may include private machine/network details; do not commit it. Build it with `go build -p 2 -o ~/.local/bin/agentbox-facts ./cmd/agentbox-facts` only if you want to adapt it for your host.

Tests: `go test -p 2 ./...`.
