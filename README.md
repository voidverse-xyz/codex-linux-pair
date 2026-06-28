# Codex Remote Control

Pair a Linux machine with the **ChatGPT Codex app** so you can drive it remotely
from your phone — set up in one command.

## Quick start

Run this on the Linux machine you want to control:

```bash
python3 codex-pair.py
```

It prints a pairing code:

```text
PAIRING CODE:  XXXX-XXXX
```

Open the Codex app, choose **pair a device**, and enter the code. That's it.

## Requirements

- Linux with **Python 3**
- The **`codex` CLI** installed and logged in (`codex login`)
  ```bash
  curl -fsSL https://chatgpt.com/codex/install.sh | sh
  ```
- Internet access to `chatgpt.com` (including WebSocket connections)

No extra Python packages are needed — the script uses only the standard library.

## What the script does

`codex-pair.py` walks the whole host-side setup and reports `[ OK ]` / `[WARN]` /
`[FAIL]` for each step, so you can see exactly where things stand. In order, it:

1. **Checks the `codex` CLI** is installed and on your `PATH`.
2. **Reads your login** from `auth.json` and verifies the token is still valid
   server-side (a real API call, not just a local check).
3. **Checks connectivity** — runs `codex doctor` to confirm the WebSocket needed
   for remote control can reach `chatgpt.com`.
4. **Verifies the standalone install** that the remote-control daemon depends on.
5. **Starts the remote-control daemon** and confirms it reports `connected`.
6. **Enrolls this host** with ChatGPT and mints a **manual pairing code**.
7. **Waits for the phone to pair** (by default), then offers to enable autostart.

If a required step fails, the script stops and prints the exact command to fix
it — resolve it and run the script again.

### Start on boot

After a successful pair, the script asks whether to keep the machine
controllable across reboots (default is **no**, and it asks before installing
anything). If you agree, it picks the right mechanism automatically:

- **systemd** — installs a user service, enables it, and turns on linger so it
  starts at boot before you log in.
- **cron** — falls back to an `@reboot` job when no systemd user service is
  available.

## Options

```bash
python3 codex-pair.py --no-wait        # print the code and exit
python3 codex-pair.py --wait 120       # wait up to 120 seconds for pairing
CODEX_HOME=/custom/path python3 codex-pair.py
```

| Option             | What it does                                                        |
| ------------------ | ------------------------------------------------------------------- |
| `--no-wait`        | Print the pairing code and exit instead of waiting for the phone.   |
| `--wait <seconds>` | Wait up to N seconds for pairing (default: until the code expires). |
| `CODEX_HOME`       | Point at a non-default Codex configuration directory.               |

> Note: autostart is only offered when the script actually waits for and
> completes a pair, so it is skipped with `--no-wait`.

## Exit codes

- `0` — paired successfully, or `--no-wait` produced a code.
- `1` — a required check failed, or pairing timed out.

## Troubleshooting

- **A check failed.** The checklist prints `[FAIL]` with the reason and the exact
  command to fix it. Resolve it and run the script again.
- **The code expired.** Codes are short-lived. Just re-run the script to mint a
  fresh one.
- **Login expired or was revoked.** Sign in again:
  ```bash
  codex login --device-auth
  ```
- **Paired, but the host shows offline.** The pairing succeeded but the
  remote-control service isn't running. Start it manually:
  ```bash
  codex remote-control start
  ```
