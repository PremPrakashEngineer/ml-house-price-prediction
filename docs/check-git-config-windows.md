# Checking Git Configuration on Windows — Step by Step

This guide walks through verifying a Git installation and inspecting its configuration on a Windows machine. All commands work in **PowerShell**, **Command Prompt (CMD)**, and **Git Bash** unless noted otherwise.

---

## Table of Contents

1. [Open a terminal](#step-1--open-a-terminal)
2. [Confirm Git is installed](#step-2--confirm-git-is-installed)
3. [Understand the configuration levels](#step-3--understand-the-configuration-levels)
4. [List the full configuration and where each value comes from](#step-4--list-the-full-configuration-and-its-sources)
5. [Check your identity (name and email)](#step-5--check-your-identity)
6. [Check line-ending settings](#step-6--check-line-ending-settings)
7. [Check the credential helper](#step-7--check-the-credential-helper)
8. [Check the default editor](#step-8--check-the-default-editor)
9. [Check the default branch name](#step-9--check-the-default-branch-name)
10. [Check Windows-specific settings](#step-10--check-windows-specific-settings)
11. [Check network settings (proxy and SSL)](#step-11--check-network-settings)
12. [Check SSH setup (if you use SSH remotes)](#step-12--check-ssh-setup)
13. [Check repository-level settings](#step-13--check-repository-level-settings)
14. [Fix or change a setting](#step-14--fix-or-change-a-setting)
15. [Quick checklist](#quick-checklist)
16. [Troubleshooting](#troubleshooting)

---

## Step 1 — Open a terminal

Use any of the following:

- **PowerShell**: press `Win + X` → *Terminal* or *Windows PowerShell*
- **Command Prompt**: press `Win + R`, type `cmd`, press Enter
- **Git Bash**: right-click in a folder → *Open Git Bash here* (installed with Git for Windows)

> Tip: To inspect the **system-level** config file in Step 4 and edit it in Step 14, you may need to run the terminal **as Administrator**.

---

## Step 2 — Confirm Git is installed

```powershell
git --version
```

Expected output (version will vary):

```
git version 2.47.0.windows.1
```

Find out **which** `git.exe` is being used (useful if more than one is installed):

```powershell
# PowerShell
Get-Command git | Select-Object -ExpandProperty Source

# CMD
where git
```

Typical location: `C:\Program Files\Git\cmd\git.exe`

If you see `'git' is not recognized as an internal or external command`, Git is not installed or not on your `PATH` — see [Troubleshooting](#troubleshooting).

---

## Step 3 — Understand the configuration levels

Git reads settings from several files. A value at a more specific level **overrides** the same value from a broader level.

| Level | Flag | Applies to | Default file location on Windows |
|---|---|---|---|
| Portable / ProgramData | — | All users (Git for Windows only) | `C:\ProgramData\Git\config` |
| System | `--system` | All users on the machine | `C:\Program Files\Git\etc\gitconfig` |
| Global | `--global` | Your Windows user account | `C:\Users\<you>\.gitconfig` (`%USERPROFILE%\.gitconfig`) |
| Local | `--local` | One repository | `<repo>\.git\config` |
| Worktree | `--worktree` | One worktree (rarely used) | `<repo>\.git\config.worktree` |

**Priority (lowest → highest):** System → Global → Local → Worktree

---

## Step 4 — List the full configuration and its sources

This single command is the most useful check. It shows every effective setting, which file it came from, and which scope it belongs to:

```powershell
git config --list --show-origin --show-scope
```

Example output:

```
system  file:C:/Program Files/Git/etc/gitconfig    core.autocrlf=true
system  file:C:/Program Files/Git/etc/gitconfig    credential.helper=manager
global  file:C:/Users/prem/.gitconfig              user.name=Prem Prakash
global  file:C:/Users/prem/.gitconfig              user.email=prem@example.com
local   file:.git/config                           core.bare=false
```

> `--show-scope` requires Git 2.26 or later. On older versions, use `git config --list --show-origin`.

View one level at a time:

```powershell
git config --system --list
git config --global --list
git config --local --list      # run inside a repository
```

Open a config file in your editor to view it directly:

```powershell
git config --global --edit
```

Or show the file paths:

```powershell
# PowerShell
notepad $env:USERPROFILE\.gitconfig

# CMD
notepad %USERPROFILE%\.gitconfig
```

---

## Step 5 — Check your identity

Every commit records a name and email, so these must be set.

```powershell
git config user.name
git config user.email
```

To see **where** each value is defined (useful when a repo uses a different identity):

```powershell
git config --show-origin user.name
git config --show-origin user.email
```

If nothing is printed, the value is not set. Set it globally:

```powershell
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

---

## Step 6 — Check line-ending settings

Windows uses CRLF line endings, while Linux and macOS use LF. A wrong setting causes noisy diffs and "every line changed" problems.

```powershell
git config core.autocrlf
git config core.eol
```

| `core.autocrlf` value | Behavior | Recommended for |
|---|---|---|
| `true` | Converts LF → CRLF on checkout, CRLF → LF on commit | Most Windows users (the Git for Windows default) |
| `input` | Converts CRLF → LF on commit only | Cross-platform projects that want LF everywhere |
| `false` | No conversion | Projects that manage line endings with `.gitattributes` |

Also check whether the repository has a `.gitattributes` file, which overrides `core.autocrlf` for the paths it covers:

```powershell
# PowerShell
Get-Content .gitattributes

# CMD
type .gitattributes
```

---

## Step 7 — Check the credential helper

The credential helper stores your HTTPS passwords or tokens so you don't have to type them every time.

```powershell
git config credential.helper
```

Expected on a modern Git for Windows install:

```
manager
```

Verify Git Credential Manager is present:

```powershell
git credential-manager --version
```

To view or remove saved credentials, open **Windows Credential Manager**:

`Control Panel` → `User Accounts` → `Credential Manager` → `Windows Credentials` → look for entries starting with `git:`

Or from the command line:

```powershell
cmdkey /list | findstr git
```

---

## Step 8 — Check the default editor

This editor opens for commit messages, interactive rebases, etc.

```powershell
git config core.editor
```

If empty, Git falls back to the editor chosen during installation (often Vim). Common Windows options:

```powershell
# Visual Studio Code
git config --global core.editor "code --wait"

# Notepad++
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"

# Notepad
git config --global core.editor notepad
```

---

## Step 9 — Check the default branch name

Controls the branch name used by `git init`.

```powershell
git config init.defaultBranch
```

If empty, Git uses `master`. To use `main`:

```powershell
git config --global init.defaultBranch main
```

---

## Step 10 — Check Windows-specific settings

```powershell
git config core.longpaths     # allow paths longer than 260 characters
git config core.symlinks      # whether symlinks are created as real symlinks
git config core.fscache       # file system cache for faster status on Windows
git config core.filemode      # usually false on Windows
git config core.ignorecase    # usually true on Windows (case-insensitive file system)
```

Recommended values:

| Setting | Recommended | Why |
|---|---|---|
| `core.longpaths` | `true` | Prevents `Filename too long` errors in deep folder trees |
| `core.fscache` | `true` | Speeds up `git status` on large repositories |
| `core.filemode` | `false` | NTFS doesn't track Unix executable bits |
| `core.ignorecase` | `true` | Matches Windows file system behavior |

Enable long paths if needed:

```powershell
git config --global core.longpaths true
```

---

## Step 11 — Check network settings

### Proxy

```powershell
git config http.proxy
git config https.proxy
```

Also check environment variables that Git honors:

```powershell
# PowerShell
Get-ChildItem Env: | Where-Object Name -match "proxy"

# CMD
set | findstr /i proxy
```

Remove a stale proxy:

```powershell
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### SSL backend

```powershell
git config http.sslBackend
```

| Value | Meaning |
|---|---|
| `schannel` | Uses the Windows certificate store (works with corporate certificates) |
| `openssl` | Uses Git's bundled CA certificates |

If you get certificate errors on a corporate network, `schannel` usually fixes them:

```powershell
git config --global http.sslBackend schannel
```

> Never disable SSL verification (`http.sslVerify false`) as a permanent fix — it exposes you to man-in-the-middle attacks.

---

## Step 12 — Check SSH setup

Only needed if your remotes look like `git@github.com:user/repo.git`.

List your SSH keys:

```powershell
# PowerShell
Get-ChildItem $env:USERPROFILE\.ssh

# CMD
dir %USERPROFILE%\.ssh
```

You should see a key pair such as `id_ed25519` and `id_ed25519.pub`.

Check which SSH program Git uses:

```powershell
git config core.sshCommand
```

If empty, Git uses its bundled OpenSSH. To use Windows' built-in OpenSSH (so the Windows `ssh-agent` service works):

```powershell
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

Test the connection (example for GitHub):

```powershell
ssh -T git@github.com
```

Expected: `Hi <username>! You've successfully authenticated...`

---

## Step 13 — Check repository-level settings

Run these **inside a repository folder**:

```powershell
cd C:\path\to\your\repo
```

Remote URLs:

```powershell
git remote -v
```

Current branch and upstream tracking:

```powershell
git branch -vv
```

Local overrides (anything here beats global settings):

```powershell
git config --local --list
```

### "Dubious ownership" / safe.directory

If Git reports `detected dubious ownership in repository`, check the allowed directories:

```powershell
git config --global --get-all safe.directory
```

Add the repository if you trust it:

```powershell
git config --global --add safe.directory "C:/path/to/your/repo"
```

> Use forward slashes `/` in the path.

---

## Step 14 — Fix or change a setting

| Task | Command |
|---|---|
| Set a value globally | `git config --global <key> "<value>"` |
| Set a value for one repo | `git config --local <key> "<value>"` |
| Set a value system-wide (Admin) | `git config --system <key> "<value>"` |
| Read a value | `git config <key>` |
| Read a value and its source | `git config --show-origin <key>` |
| Remove a value | `git config --global --unset <key>` |
| Remove all values for a multi-valued key | `git config --global --unset-all <key>` |
| Edit a config file directly | `git config --global --edit` |

> Git 2.46+ also supports newer subcommand syntax such as `git config get user.name` and `git config set --global user.name "Your Name"`. The classic syntax shown above still works on all versions.

---

## Quick checklist

Copy and paste this block into PowerShell to print the key settings in one go:

```powershell
Write-Host "`n=== Git version ===";        git --version
Write-Host "`n=== Git location ===";       (Get-Command git).Source
Write-Host "`n=== Identity ===";           git config --show-origin user.name; git config --show-origin user.email
Write-Host "`n=== Line endings ===";       git config --show-origin core.autocrlf
Write-Host "`n=== Credential helper ===";  git config --show-origin credential.helper
Write-Host "`n=== Editor ===";             git config --show-origin core.editor
Write-Host "`n=== Default branch ===";     git config --show-origin init.defaultBranch
Write-Host "`n=== Long paths ===";         git config --show-origin core.longpaths
Write-Host "`n=== SSL backend ===";        git config --show-origin http.sslBackend
Write-Host "`n=== Proxy ===";              git config --show-origin http.proxy
Write-Host "`n=== All settings ===";       git config --list --show-origin --show-scope
```

Manual checklist:

- [ ] `git --version` prints a version
- [ ] `user.name` and `user.email` are set
- [ ] `core.autocrlf` matches your project's needs (usually `true`)
- [ ] `credential.helper` is `manager`
- [ ] `core.editor` is an editor you know how to use
- [ ] `init.defaultBranch` is set (e.g. `main`)
- [ ] `core.longpaths` is `true`
- [ ] No unexpected proxy settings
- [ ] SSH keys exist and `ssh -T` succeeds (if using SSH)
- [ ] No surprising overrides in `git config --local --list`

---

## Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| `'git' is not recognized...` | Git not installed or not on `PATH` | Install from <https://git-scm.com/download/win> or `winget install --id Git.Git -e`, then reopen the terminal |
| A setting won't change | A higher-priority level overrides it | Run `git config --show-origin --show-scope <key>` to find which file sets it |
| `Permission denied` editing system config | Not running as Administrator | Reopen the terminal with *Run as administrator* |
| Commits show the wrong author | Local repo overrides global identity | `git config --local --list` and unset or change `user.name` / `user.email` |
| Every line shows as changed in diffs | Line-ending mismatch | Review `core.autocrlf` and `.gitattributes`; then `git add --renormalize .` |
| `Filename too long` | 260-character path limit | `git config --global core.longpaths true` |
| `SSL certificate problem` | Corporate proxy / custom CA | `git config --global http.sslBackend schannel` |
| Repeated password prompts | Credential helper missing or stale | Set `credential.helper manager` and clear old `git:` entries in Windows Credential Manager |
| `detected dubious ownership` | Folder owned by another user | `git config --global --add safe.directory "C:/path/to/repo"` |
| `Permission denied (publickey)` | SSH key missing or not added to the host | Generate a key with `ssh-keygen -t ed25519`, add the `.pub` key to your Git host, and retest with `ssh -T` |
