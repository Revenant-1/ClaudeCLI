# Claude Code + OpenRouter Setup

**One interactive script to install Claude Code and run it through OpenRouter, without touching your normal Claude login.**

```text
claude      →  your normal Claude Code (Anthropic account)   ← unchanged
claude-or   →  Claude Code routed through OpenRouter          ← added by this setup
```

| Script | Platform |
|---|---|
| [`setup.ps1`](setup.ps1) | Native Windows (PowerShell 5.1+ or 7+) |
| [`setup.sh`](setup.sh) | macOS, Linux, WSL (and hands off to `setup.ps1` from Git Bash) |

> [!IMPORTANT]
> Claude Code itself is not free just because you installed it. OpenRouter offers some free models and routes, but availability, rate limits and privacy terms can change at any time. Nothing in this repo promises "free Claude Code".

---

## Contents

- [Features](#features)
- [Quick start](#quick-start)
- [How it works](#how-it-works)
- [Requirements](#requirements)
- [Choosing a model](#choosing-a-model)
- [Everyday use](#everyday-use)
- [Verify your setup](#verify-your-setup)
- [Where files are stored](#where-files-are-stored)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Security](#security)
- [Reset and uninstall](#reset-and-uninstall)

---

## Features

- **Installs Claude Code** with Anthropic's official native installer (asks before running it).
- **Bring your own key.** No key is embedded anywhere. Your key is validated against OpenRouter when you enter it.
- **Live free-model list.** Fetches OpenRouter's current models that cost $0 *and* support tool calling, so nothing goes stale in the script.
- **Three model modes:** a free model, a model ID you type, or Anthropic Claude via OpenRouter (paid credits).
- **Safe by default.** Separate `claude-or` launcher, key stored encrypted (Windows) or in a `600` file (Unix), never in a `.env` file.
- **Self-repair.** Windows menu option 5 fixes PATH and launcher problems automatically.
- **Check and reset.** Health check with optional live test, plus one-step removal of everything the scripts created.

---

## Quick start

### Windows

Open PowerShell in the folder containing the scripts:

```powershell
powershell -ExecutionPolicy Bypass -File .\setup.ps1
```

### macOS / Linux / WSL

```bash
chmod +x setup.sh
./setup.sh
```

### Then follow the menu

| # | Windows | macOS / Linux / WSL |
|---|---|---|
| 1 | Install Claude Code | Install Claude Code |
| 2 | Configure OpenRouter | Configure OpenRouter |
| 3 | Check installation | Check installation |
| 4 | Reset / remove OpenRouter config | Reset / remove OpenRouter config |
| 5 | **Fix PATH / launcher problems** | Exit |
| 6 | Exit | n/a |

**First time:** choose **1 → 2 → 3**, open a **new terminal** if the script changed your PATH, then run:

```bash
claude-or
```

Inside Claude Code, type `/status` and confirm the base URL is `https://openrouter.ai/api`.

---

## How it works

For each `claude-or` run, the launcher sets these environment variables **for that process only**:

| Variable | Value |
|---|---|
| `ANTHROPIC_BASE_URL` | `https://openrouter.ai/api` |
| `ANTHROPIC_AUTH_TOKEN` | your OpenRouter key |
| `ANTHROPIC_API_KEY` | cleared, so a real Anthropic key is never used by mistake |
| `ANTHROPIC_DEFAULT_*_MODEL` | your chosen model |
| `CLAUDE_CODE_SUBAGENT_MODEL` | your chosen model |

**Windows**

```text
claude-or → claude-or.cmd → claude-or-run.ps1 → claude.exe → OpenRouter
              (starts the .ps1 with -ExecutionPolicy Bypass, for that one process only)
```

**macOS / Linux / WSL**

```text
claude-or → ~/.local/bin/claude-or → loads ~/.config/claude-openrouter/env → claude → OpenRouter
```

Because the variables live only inside the launcher, closing the terminal or running plain `claude` puts you straight back on your normal Anthropic setup.

---

## Requirements

| | Windows | macOS / Linux / WSL |
|---|---|---|
| Shell | PowerShell 5.1+ (7+ works too) | `bash` |
| Tools | none extra; [Git for Windows](https://git-scm.com/) may be needed by Claude Code | `curl`; plus `jq` **or** `python3` for the free-model list (optional) |
| Network | internet access | internet access |

You also need **your own OpenRouter API key**: https://openrouter.ai/settings/keys

If neither `jq` nor `python3` is available on Unix, you can still type a model ID manually.

---

## Choosing a model

After you enter your key, option 2 offers:

```text
[1] Free model (choose from OpenRouter's current free list)
[2] Enter a model ID manually
[3] Anthropic Claude via OpenRouter (paid credits, best compatibility)
```

| Mode | What it does | Cost | Compatibility |
|---|---|---|---|
| **1. Free model** | Lists up to 15 current free models with tool-calling support, sorted by context length; you pick one (or press `m` to type an ID). | $0, but tight limits | Varies by model |
| **2. Manual ID** | You enter something like `vendor/model:free`. The format is validated. | Depends on the model | Varies by model |
| **3. Anthropic via OpenRouter** | Enables gateway model discovery and maps the Fable, Opus, Sonnet, Haiku and subagent aliases to Anthropic's latest models. | Billed to your OpenRouter credits | Best |

> [!WARNING]
> OpenRouter states that Claude Code is only guaranteed to work with Anthropic's first-party models. Other models, including free ones, can misbehave with tool calls. Free endpoints may also log prompts, so don't send secrets, credentials or private code through them.

To change the model or key later, just run **[2] Configure OpenRouter** again.

---

## Everyday use

```bash
claude-or                                   # interactive session via OpenRouter
claude-or -p "Reply with the single word: OK"   # one-off, non-interactive
claude-or --continue                        # any Claude Code argument is forwarded
claude                                      # your normal Anthropic setup
```

**If you were previously logged in to Claude Code with an Anthropic account,** run `/logout` once inside Claude Code, quit, and start `claude-or`. A cached login can cause auth-conflict or model-not-found errors.

### Optional: make plain `claude` use OpenRouter (Unix only)

During configuration on macOS/Linux/WSL you can choose to add a marked block to your shell profile so plain `claude` also uses OpenRouter. It is off by default, and Reset removes it.

| Shell / OS | Profile edited |
|---|---|
| zsh | `~/.zshrc` |
| bash on macOS | `~/.bash_profile` |
| bash on Linux / WSL | `~/.bashrc` |
| other POSIX shell | `~/.profile` |
| fish | skipped (use `claude-or`) |

Windows intentionally has no equivalent: it would require storing your key in a plain-text user environment variable.

---

## Verify your setup

Choose **Check installation** in the menu. It reports:

1. whether `claude` is installed and reachable (and where it lives)
2. whether the OpenRouter configuration exists, and on Unix that its permissions are `600`
3. the configured model and the last 4 characters of the key
4. whether OpenRouter accepts the key
5. whether `claude-or` exists and is on PATH; on Windows, which file it resolves to and the current execution policy
6. a warning if your session has a real `ANTHROPIC_API_KEY` (which plain `claude` would use)

It can then run **one tiny live request** through OpenRouter. This uses a small amount of quota or credits.

Finally, inside `claude-or` run `/status` and look for:

```text
Auth token: ANTHROPIC_AUTH_TOKEN
Anthropic base URL: https://openrouter.ai/api
```

---

## Where files are stored

### Windows: `%USERPROFILE%\.claude-openrouter`

```text
.claude-openrouter\
├── key.enc             your key, encrypted with Windows DPAPI (this Windows user, this PC only)
├── config.json         chosen model and environment settings
├── claude-or-run.ps1   the launcher logic
└── claude-or.cmd       the command you type
```

### macOS / Linux / WSL

```text
~/.config/claude-openrouter/env    your key and model settings (chmod 600, directory 700)
~/.local/bin/claude-or             the launcher
```

The key is never written to a `.env` file, a project folder, or your shell profile.

---

## Troubleshooting

### Quick lookup

| Symptom | Fix |
|---|---|
| Windows: `claude` not recognized | Menu **[5]**, then reopen PowerShell |
| Windows: `claude-or` not recognized | Menu **[5]**, or `$env:Path += ";$env:USERPROFILE\.claude-openrouter"` for this window |
| Windows: `running scripts is disabled` | Menu **[5]** (replaces an old blocked `claude-or.ps1`) |
| Windows: "Could not decrypt the saved key" | Menu **[2]** and re-enter the key |
| Unix: `claude` or `claude-or` not found | Add `~/.local/bin` to PATH, open a new terminal |
| OpenRouter rejected the key | Menu **[2]** and re-enter it |
| `/status` shows the wrong auth | `/logout`, quit, start `claude-or` again |
| Free model list won't load | Press `m` and type a model ID |

### Windows details

Start with menu **[5] Fix PATH / launcher problems**. It locates `claude.exe` even when PATH is wrong, adds it to your user PATH, rewrites the launcher files (no need to re-enter your key), adds the launcher folder to PATH, removes any old blocked `claude-or.ps1`, and reports your execution policy. Windows that were already open must be closed and reopened to see the PATH change.

**`claude` is not recognized.** Installed but not on PATH. Manual fix:

```powershell
$p = "$env:USERPROFILE\.local\bin"
[Environment]::SetEnvironmentVariable('Path', [Environment]::GetEnvironmentVariable('Path','User') + ";$p", 'User')
$env:Path += ";$p"
claude --version
```

If it is still missing, run `Get-ChildItem $env:USERPROFILE\.local\bin`. An empty folder means you should run **[1]** again.

**`claude-or` is not recognized.** The launcher lives in `%USERPROFILE%\.claude-openrouter`, and that exact folder must be on PATH. Adding your *project* folder to PATH does nothing for it.

```powershell
dir "$env:USERPROFILE\.claude-openrouter"   # expect claude-or.cmd, claude-or-run.ps1, config.json, key.enc
```

If the folder doesn't exist, run **[2] Configure OpenRouter** first.

**`File ...claude-or.ps1 cannot be loaded because running scripts is disabled`.** PowerShell prefers `claude-or.ps1` over `claude-or.cmd`, and its default execution policy blocks `.ps1` files. Current versions avoid this entirely by naming the script `claude-or-run.ps1` and starting it only from `claude-or.cmd` with `-ExecutionPolicy Bypass` for that one process, so **no policy change is needed**. If you have an older install, run **[5]**. You may optionally allow local scripts for your user only:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

If a Group Policy controls that setting it can't be changed, but `claude-or.cmd` still works.

### macOS / Linux / WSL details

```bash
ls -l ~/.local/bin/claude-or     # launcher exists?
echo "$PATH"                     # is ~/.local/bin in it?
```

If needed, add this to your profile and open a new terminal:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

### Live test fails

Run **Check installation** and confirm: `claude` is installed, the config exists, the key is accepted, the model ID is valid, `claude-or` resolves, and `/status` shows the OpenRouter base URL. Free models can also be rate-limited or temporarily unavailable.

---

## FAQ

**Is Claude Code free with this?**
No. The scripts install Claude Code and connect it to OpenRouter. Some OpenRouter models are free; the terms can change.

**Does this replace my normal `claude`?**
No. Your Anthropic setup keeps working. Only `claude-or` uses OpenRouter (unless you opt in on Unix).

**Can I switch models without re-running setup?**
Re-running **[2]** is the dependable way. Arguments such as `--model <id>` are forwarded to Claude Code, but the launcher also pins model variables, so results depend on Claude Code's own behavior. Whatever you use must be a valid OpenRouter model ID.

**Why does the script clear `ANTHROPIC_API_KEY`?**
So a real Anthropic key in your environment can never take precedence over the OpenRouter token during `claude-or`.

**Why a separate `claude-or-run.ps1` on Windows?**
So PowerShell resolves `claude-or` to the `.cmd`, which bypasses the execution policy for itself only. See [Windows details](#windows-details).

**Can I share the script with a key already inside?**
Please don't. Anyone who received it could spend your credits. Every user should create their own key.

**WSL or native Windows?**
Running `./setup.sh` *inside WSL* installs the Linux build inside WSL, which is separate from native Windows. Running it from Git Bash detects native Windows and offers to launch `setup.ps1` (keep both files in the same folder).

---

## Security

- The scripts contain **no** embedded key. Every user brings their own.
- Never paste your key into GitHub, issues, screenshots or logs.
- **Windows:** the key is stored encrypted with DPAPI, readable only by your Windows user on that PC, and decrypted only when `claude-or` starts.
- **macOS / Linux / WSL:** the key lives in a `600` file inside a `700` directory, and is passed to `curl` for validation without appearing in the process list.
- The installers run Anthropic's official scripts (`https://claude.ai/install.sh` and `https://claude.ai/install.ps1`) and ask for confirmation first. Read them if you want to know exactly what they do.
- The scripts never change your system-wide PowerShell execution policy.

---

## Reset and uninstall

Menu option **[4] Reset / remove OpenRouter config** removes what these scripts created and undoes PATH or profile lines they added. It does **not** uninstall Claude Code.

| Platform | Removed |
|---|---|
| Windows | `%USERPROFILE%\.claude-openrouter` and its PATH entry |
| macOS / Linux / WSL | `~/.config/claude-openrouter`, `~/.local/bin/claude-or`, and the marked profile block |

To remove Claude Code itself on Unix, delete `~/.local/bin/claude` and `~/.local/share/claude`. On Windows, see Anthropic's Claude Code documentation.

---

## Project layout

```text
.
├── setup.ps1    Windows installer / configurator
├── setup.sh     macOS, Linux, WSL installer / configurator
└── README.md    this file
```