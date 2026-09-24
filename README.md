# Claude Code + OpenRouter Setup

Interactive setup scripts for running **Claude Code through OpenRouter** while keeping your normal Claude Code / Anthropic login available.

This repository provides:

- `setup.ps1` — native Windows setup for PowerShell 5.1+ / 7+
- `setup.sh` — macOS, Linux, and WSL setup
- A separate `claude-or` launcher that routes Claude Code through OpenRouter
- Interactive OpenRouter API-key validation
- Automatic selection of current free OpenRouter models that support tool calling
- Optional Anthropic Claude models through OpenRouter
- Installation checks, live connection testing, and configuration reset

The scripts are designed so that OpenRouter configuration does **not** have to replace your normal `claude` command unless you explicitly choose that option.

---

## How it works

The setup creates a separate launcher:

```text
claude-or
```

Normal Claude Code remains:

```text
claude
```

By default:

```text
claude     → your normal Claude Code / Anthropic setup
claude-or  → Claude Code routed through OpenRouter
```

The OpenRouter launcher sets:

```text
ANTHROPIC_BASE_URL=https://openrouter.ai/api
ANTHROPIC_AUTH_TOKEN=<your OpenRouter key>
```

It also clears `ANTHROPIC_API_KEY` for the OpenRouter invocation so a real Anthropic API key from the current environment is not accidentally used.

The Windows script stores the OpenRouter key using Windows DPAPI. The Unix/macOS/WSL script stores it in a file protected with mode `600`.

---

# Requirements

## Windows

Supported:

- Windows
- PowerShell 5.1 or newer
- PowerShell 7+ also works
- Internet access
- Git for Windows may be required by Claude Code

The Windows script uses Anthropic's native installer when you select **Install Claude Code**.

## macOS / Linux

Supported:

- macOS
- Linux
- WSL

The Unix script expects:

- `bash`
- `curl`

For automatic free-model discovery, it also needs either:

- `jq`, or
- `python3`

If neither is available, you can enter a model ID manually.

## OpenRouter

You need your **own OpenRouter API key**.

Create one from the OpenRouter key settings page:

https://openrouter.ai/settings/keys

Do not share your API key or commit it to Git.

---

# Quick Start

## Windows PowerShell

Open PowerShell in the directory containing the scripts:

```powershell
powershell -ExecutionPolicy Bypass -File .\setup.ps1
```

Then use the interactive menu:

```text
[1] Install Claude Code
[2] Configure OpenRouter
[3] Check installation
[4] Reset / remove OpenRouter config
[5] Exit
```

Recommended first-time sequence:

```text
1 → Install Claude Code
2 → Configure OpenRouter
3 → Check installation
```

After configuration:

```powershell
claude-or
```

---

## macOS / Linux / WSL

Make the script executable:

```bash
chmod +x setup.sh
```

Run it:

```bash
./setup.sh
```

Then use:

```text
[1] Install Claude Code
[2] Configure OpenRouter
[3] Check installation
[4] Reset / remove OpenRouter config
[5] Exit
```

After configuration:

```bash
claude-or
```

---

# Windows: Detailed Setup

## 1. Start PowerShell

Open PowerShell and navigate to the directory containing `setup.ps1`.

For example:

```powershell
cd C:\path\to\claude-openrouter
```

## 2. Run the setup script

```powershell
powershell -ExecutionPolicy Bypass -File .\setup.ps1
```

The script displays:

```text
+------------------------------------------+
|     Claude Code + OpenRouter Setup       |
+------------------------------------------+
```

## 3. Install Claude Code

Choose:

```text
[1] Install Claude Code
```

If Claude Code is already installed, the script detects it and asks whether you want to reinstall/update it.

The installer invoked by the script is Anthropic's native installer:

```powershell
irm https://claude.ai/install.ps1 | iex
```

If Claude was installed successfully but is not immediately available, open a new PowerShell window.

### Git for Windows

The setup checks whether `git` is available.

If Git for Windows is missing and Claude Code requires Git Bash functionality, install Git for Windows from:

https://git-scm.com/

---

# Configure OpenRouter

Choose:

```text
[2] Configure OpenRouter
```

The script asks for your OpenRouter API key.

Input is hidden:

```text
Enter your OpenRouter API key (input hidden)
```

The key is checked for basic formatting and then validated against OpenRouter.

A normal OpenRouter key is expected to begin with:

```text
sk-or-
```

If OpenRouter explicitly rejects the key, the script asks whether you want to save it anyway.

---

# Choose a Model

After entering the API key, the setup provides three choices:

```text
[1] Free model (choose from OpenRouter's current free list)
[2] Enter a model ID manually
[3] Anthropic Claude via OpenRouter (paid credits, best compatibility)
```

## Option 1 — Free model

The script queries OpenRouter's current model list and filters for models that:

- have zero prompt pricing
- have zero completion pricing
- support tool calling

It then displays up to 15 models, ordered by context length.

Example format:

```text
[ 1] vendor/model-name:free                         128k context
[ 2] vendor/another-model:free                      64k context
...
[ m] Enter a model ID manually
```

Select the number corresponding to the model you want.

### Important

Free model availability, limits, and behavior can change on OpenRouter.

The setup script does not hard-code a permanent free-model list; it queries OpenRouter when you configure it.

---

## Option 2 — Manual model ID

Choose:

```text
[2] Enter a model ID manually
```

Then enter the OpenRouter model ID, for example:

```text
vendor/model:free
```

The script validates the model-ID format before saving it.

Use this option when:

- you already know the model ID
- the model is not shown in the automatically discovered list
- you want to use a specific OpenRouter model

---

## Option 3 — Anthropic Claude through OpenRouter

Choose:

```text
[3] Anthropic Claude via OpenRouter
```

This configures Claude Code to use Anthropic models through OpenRouter.

The configuration enables gateway model discovery and assigns the Anthropic model aliases used by the script, including Fable, Opus, Sonnet, Haiku, and the subagent model.

This option uses your OpenRouter credits and is not the same as logging directly into Anthropic.

---

# Where Configuration Is Stored

## Windows

The configuration directory is:

```text
%USERPROFILE%\.claude-openrouter
```

It contains:

```text
key.enc
config.json
claude-or.ps1
claude-or.cmd
```

The API key is stored in encrypted form using Windows DPAPI.

The encrypted key is intended to be readable by your Windows user on that machine.

The key is not stored in:

- a `.env` file
- the Windows registry
- plain text configuration

---

## macOS / Linux / WSL

The configuration directory is:

```text
~/.config/claude-openrouter
```

The main secret file is:

```text
~/.config/claude-openrouter/env
```

The script creates the directory with restrictive permissions and writes the environment file with:

```text
chmod 600
```

The launcher is:

```text
~/.local/bin/claude-or
```

The environment file contains the OpenRouter key, so **do not commit it, upload it, or share it**.

---

# Starting Claude Code through OpenRouter

Once setup is complete, run:

```bash
claude-or
```

On Windows PowerShell:

```powershell
claude-or
```

This launcher loads the OpenRouter configuration and then starts Claude Code.

You can pass normal Claude Code arguments through the launcher.

For example:

```bash
claude-or -p "Reply with the single word: OK"
```

The setup's own live test uses this same pattern.

---

# Verify the Configuration

Run the setup script again and choose:

```text
[3] Check installation
```

The check verifies:

1. Whether `claude` is installed
2. Whether the OpenRouter configuration exists
3. Which model is configured
4. Whether the saved key can be read
5. Whether OpenRouter accepts the key
6. Whether the `claude-or` launcher exists
7. Whether a conflicting `ANTHROPIC_API_KEY` is present

The script can also run a small live request through OpenRouter.

The live test consumes a small amount of OpenRouter quota/credits.

---

# Verify from Inside Claude Code

After starting:

```bash
claude-or
```

run:

```text
/status
```

The configuration should show:

```text
Auth token: ANTHROPIC_AUTH_TOKEN
```

and the Anthropic base URL should be:

```text
https://openrouter.ai/api
```

This confirms that the OpenRouter launcher is providing the authentication and endpoint configuration.

---

# Important: Existing Anthropic Login

If you have previously logged into Claude Code directly with an Anthropic account, the setup recommends running:

```text
/logout
```

inside Claude Code once.

Then quit Claude Code and start it again.

A cached Anthropic login can otherwise cause authentication conflicts or model-not-found errors when switching to the OpenRouter configuration.

A typical clean transition is:

```text
Claude Code
    ↓
/logout
    ↓
quit Claude Code
    ↓
claude-or
    ↓
/status
```

---

# Keeping Both Setups

The recommended configuration is:

```text
claude
  └── normal Anthropic setup

claude-or
  └── OpenRouter setup
```

This lets you switch between them without rewriting your normal Claude Code configuration.

For example:

```bash
claude
```

uses your normal configuration.

```bash
claude-or
```

uses OpenRouter.

This separation is the default behavior of the setup scripts.

---

# Making `claude` Use OpenRouter by Default

The Unix/macOS/WSL script provides an optional choice:

```text
Make OpenRouter the default for 'claude'?
```

If you answer yes, the script adds the OpenRouter environment configuration to your shell profile.

Then:

```bash
claude
```

will use the OpenRouter configuration by default.

If you answer no, keep using:

```bash
claude-or
```

for OpenRouter.

## Windows

The Windows script deliberately does **not** enable this behavior automatically.

It explains that putting the OpenRouter key directly into user environment variables would store the key in plain text there.

The safer Windows default is therefore:

```text
claude     → normal Claude setup
claude-or  → OpenRouter
```

---

# WSL

If you run:

```bash
./setup.sh
```

inside WSL, the script treats WSL as Linux and installs/configures the Linux-side Claude Code environment inside WSL.

This is separate from native Windows PowerShell.

For native Windows, use:

```powershell
.\setup.ps1
```

If the Bash script detects native Windows through Git Bash/MSYS/Cygwin, it attempts to hand off to:

```text
setup.ps1
```

when `powershell.exe` is available.

---

# macOS and Linux Shell Profile Behavior

The script detects the shell and chooses the appropriate profile:

| Shell / OS | Profile |
|---|---|
| zsh | `~/.zshrc` |
| bash on macOS | `~/.bash_profile` |
| bash on Linux | `~/.bashrc` |
| other POSIX shell | `~/.profile` |
| fish | Profile editing is skipped |

If you choose to add `~/.local/bin` to PATH, the script adds:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

The script marks its profile changes with:

```text
# >>> claude-openrouter >>>
...
# <<< claude-openrouter <<<
```

This allows the reset operation to remove the lines that the script itself added.

After a profile change, open a new terminal so the change takes effect.

---

# Reset / Remove OpenRouter Configuration

Choose:

```text
[4] Reset / remove OpenRouter config
```

## Windows

The reset operation removes:

- OpenRouter configuration
- encrypted API key
- `claude-or` launcher
- the setup directory from PATH, if it was added by the script

It **does not uninstall Claude Code**.

The Windows configuration directory removed is:

```text
%USERPROFILE%\.claude-openrouter
```

## macOS / Linux / WSL

The reset operation removes:

- OpenRouter environment configuration
- `claude-or`
- profile lines added by the setup script

It does **not** uninstall Claude Code.

The script notes that a native Claude Code installation can separately be removed by deleting its native installation files if that is what you intend.

---

# Security

## API keys

Always use your own OpenRouter key.

Never:

- paste the key into GitHub
- commit the key
- put the key in a README
- send the key to another person
- put the key in source control
- include it in screenshots or logs

The scripts themselves do not contain an embedded OpenRouter key.

## Windows

The key is converted to a SecureString and persisted using Windows DPAPI.

The launcher decrypts it only when starting Claude Code.

## macOS / Linux / WSL

The key is stored in:

```text
~/.config/claude-openrouter/env
```

with:

```text
600
```

permissions.

That means the file is intended to be readable/writable only by its owner.

---

# Important OpenRouter / Free Model Considerations

If you select a free model, the setup warns about several limitations:

- Free models can have tight rate limits.
- Daily limits can change.
- Free endpoints may log prompts.
- Tool-calling behavior may differ between models.
- Claude Code is not guaranteed to behave correctly with arbitrary non-Anthropic models.

For that reason, do **not** send secrets, credentials, private source code, or other sensitive material through a free endpoint unless you have independently verified the endpoint's privacy and logging behavior.

For Claude-specific compatibility, using Anthropic's first-party models through OpenRouter is the configuration the script identifies as having the strongest compatibility.

---

# OpenRouter Credits

If you select:

```text
Anthropic Claude via OpenRouter
```

the requests are billed against your OpenRouter credits.

The setup does not provide or manage credits for you.

---

# Troubleshooting

## `claude: command not found`

Run the setup again and choose:

```text
1. Install Claude Code
```

If installation has completed but the command is still unavailable, open a new terminal.

On Linux/macOS, make sure:

```text
~/.local/bin
```

is on PATH.

You can check:

```bash
echo "$PATH"
```

On Windows:

```powershell
Get-Command claude
```

---

## `claude-or: command not found`

### Windows

Check whether the configuration directory was added to your user PATH.

Open a new PowerShell window after setup.

You can also run the generated launcher directly from:

```text
%USERPROFILE%\.claude-openrouter\claude-or.cmd
```

### macOS/Linux/WSL

Check:

```bash
ls -l ~/.local/bin/claude-or
```

Then:

```bash
echo "$PATH"
```

If necessary, add:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

to the appropriate shell profile.

---

## OpenRouter rejected the API key

Run:

```text
[2] Configure OpenRouter
```

and enter the key again.

Check that:

- the key is complete
- there are no spaces
- the key is active
- it belongs to your OpenRouter account
- it normally starts with `sk-or-`

The setup explicitly treats HTTP `401` and `403` responses as key rejection.

---

## Key cannot be decrypted on Windows

The Windows encrypted key is tied to the Windows user/machine context used to create it.

If the setup reports:

```text
Could not decrypt the saved key
```

re-run:

```text
[2] Configure OpenRouter
```

and save a new key.

---

## `/status` shows the wrong authentication

First check whether you previously authenticated directly with Anthropic.

Inside Claude Code:

```text
/logout
```

Then quit and relaunch:

```bash
claude-or
```

Run:

```text
/status
```

The OpenRouter launcher should provide:

```text
Auth token: ANTHROPIC_AUTH_TOKEN
```

---

## `ANTHROPIC_API_KEY` conflict

The OpenRouter launcher deliberately clears:

```text
ANTHROPIC_API_KEY
```

before starting Claude Code.

The installation checker also warns if your current shell has a real Anthropic API key.

This matters because you do not want a different authentication variable unexpectedly taking precedence.

---

## Free model list cannot be loaded

The setup needs network access to retrieve OpenRouter's current model list.

On macOS/Linux/WSL, automatic model discovery also needs either:

```text
jq
```

or:

```text
python3
```

If the list cannot be retrieved, choose:

```text
m
```

and enter a model ID manually.

On Windows, the PowerShell implementation handles the OpenRouter model list directly.

---

## Live test fails

Run the installation checker:

```text
[3] Check installation
```

Verify:

1. `claude` is installed
2. the OpenRouter configuration exists
3. the saved key is accepted
4. the selected model is valid
5. `claude-or` exists
6. `/status` reports the OpenRouter base URL

The live test intentionally sends a tiny request and may consume a small amount of quota/credits.

---

# Command Reference

## Windows

Start setup:

```powershell
powershell -ExecutionPolicy Bypass -File .\setup.ps1
```

Start OpenRouter Claude Code:

```powershell
claude-or
```

Run a non-interactive test:

```powershell
claude-or -p "Reply with the single word: OK"
```

Check Claude installation:

```powershell
claude --version
```

---

## macOS / Linux / WSL

Start setup:

```bash
chmod +x setup.sh
./setup.sh
```

Start OpenRouter Claude Code:

```bash
claude-or
```

Run a non-interactive test:

```bash
claude-or -p "Reply with the single word: OK"
```

Check Claude installation:

```bash
claude --version
```

Check the launcher:

```bash
command -v claude-or
```

Check the protected configuration file:

```bash
ls -l ~/.config/claude-openrouter/env
```

Expected permissions:

```text
-rw------- 
```

or equivalent mode:

```text
600
```

---

# Configuration Layout

## Windows

```text
%USERPROFILE%\
└── .claude-openrouter\
    ├── key.enc
    ├── config.json
    ├── claude-or.ps1
    └── claude-or.cmd
```

## macOS / Linux / WSL

```text
~/
├── .config/
│   └── claude-openrouter/
│       └── env
└── .local/
    └── bin/
        └── claude-or
```

---

# Recommended Usage Pattern

If you want to keep your normal Claude Code account and OpenRouter available side by side, use:

```text
claude
```

for your normal Anthropic setup and:

```text
claude-or
```

whenever you want OpenRouter.

This is the default design of the scripts and avoids changing your normal Claude Code environment.

---

# What the Setup Does Not Do

The scripts do not:

- embed an OpenRouter API key
- upload your key to this repository
- modify your normal `claude` command by default
- require a `.env` file
- store the Windows key in the registry
- uninstall Claude Code when resetting OpenRouter configuration

The Unix script can optionally make `claude` use OpenRouter by modifying your shell profile. The Windows script intentionally does not enable this by default because of the plaintext environment-variable concern.

---

# Files

```text
.
├── setup.ps1
├── setup.sh
└── README.md
```

### `setup.ps1`

Native Windows interactive installer/configurator.

### `setup.sh`

Interactive macOS/Linux/WSL installer/configurator. It also detects Git Bash/MSYS/Cygwin on Windows and can hand off to the PowerShell setup.

### `README.md`

This documentation.

---

# Full First-Time Workflow

## Windows

```text
1. Open PowerShell
2. cd into the project directory
3. Run setup.ps1
4. Choose 1
5. Install Claude Code
6. Choose 2
7. Enter your OpenRouter key
8. Choose a model
9. Keep the separate `claude-or` launcher
10. Open a new terminal if PATH was changed
11. Run `claude-or`
12. Run `/status`
13. Confirm the OpenRouter base URL
```

## macOS / Linux / WSL

```text
1. Open a terminal
2. cd into the project directory
3. chmod +x setup.sh
4. Run ./setup.sh
5. Choose 1
6. Install Claude Code
7. Choose 2
8. Enter your OpenRouter key
9. Choose a model
10. Add ~/.local/bin to PATH if requested
11. Open a new terminal
12. Run `claude-or`
13. Run `/status`
14. Confirm the OpenRouter base URL
```

---

# Notes

The scripts intentionally keep the OpenRouter launcher separate from the normal Claude Code command unless you opt into making OpenRouter the default.

The Windows implementation encrypts the saved key with Windows DPAPI, while the macOS/Linux/WSL implementation protects the configuration file with filesystem permissions.

For free models, availability, limits, pricing, logging behavior, and supported tool-calling capabilities can change independently of this repository. Always verify the current OpenRouter model and endpoint behavior before using a model for sensitive or production work.
