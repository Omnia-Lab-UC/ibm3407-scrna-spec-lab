# Setup

Do this **before** the first session. Read this page on GitHub first, then work
top to bottom.

There are **two paths** for the AI coding agent. **This semester everyone uses
Path A (Claude Code)** through the lab's Claude organization. Path B is kept as a
fallback. Use it only if you've been told to, or if you can't get Claude access.

| | **Path A — Claude Code** *(default)* | **Path B — local model (Ollama + opencode)** *(fallback)* |
|---|---|---|
| **Who** | Everyone in the class this semester | Only if you can't use Claude |
| **Needs** | Internet + your Claude account | ~8 GB free RAM, no account, works offline |
| **Time** | ~10–15 minutes | ~30–45 minutes, mostly downloads |
| **Do** | [Path A](#path-a--claude-code-default) (A1–A2), then [steps 1–4](#both-paths-openspec-clone-verify) | [Path B](#path-b--local-model-ollama--opencode-fallback) (B1–B4), then [steps 1–4](#both-paths-openspec-clone-verify) |

Either way you also install **OpenSpec** (the spec workflow), clone this repo,
and run one verification command (steps 1–4).

> Install commands move over time. If one fails, check the tool's official docs
> (linked in each section) — but the commands below were current at the time of
> writing.

---

## 0. Prerequisites

Install whatever you're missing. Run each check in a terminal.

- **Python 3.12, 3.13, or 3.14.** Not 3.11 or older (scanpy needs 3.12+), and
  **not 3.15 yet**: some dependencies don't publish 3.15 builds, so
  `pip install` fails.
  - **Windows:** get the **Python install manager** from
    <https://www.python.org/downloads/>, then in a new terminal run
    `py install 3.14`. Check: `py -V:3.14 --version`.
  - **macOS:** the 3.14 installer from <https://www.python.org/downloads/macos/>,
    or `brew install python@3.14`. The `python3` that comes with Apple's
    developer tools is too old. Check: `python3 --version`.
  - **Linux:** your package manager (Ubuntu 24.04+, Debian 13+, and current
    Fedora are fine). On Debian/Ubuntu also run `sudo apt install python3-venv`.
    Check: `python3 --version`.

  You don't need to install `pip` separately. The virtual environment you
  create in [step 2](#2-clone-this-repo-and-install-python-deps) comes with its
  own.
- **Node.js 20.19+** (needed by OpenSpec). Check: `node --version`. Install the
  LTS from <https://nodejs.org/en/download>. On Linux, the distro's `nodejs`
  package is often too old, so use the nodejs.org instructions (nvm) instead.
- **git.** Check: `git --version`.
  - **Windows:** [Git for Windows](https://git-scm.com/downloads/win) (Claude
    Code also uses its Git Bash).
  - **macOS:** `xcode-select --install` (this also gives you `make`).
  - **Linux:** your package manager.
- **Disk:** ~1 GB for the Python dependencies. **Path B** also needs ~6 GB for
  the model and ~8 GB free RAM to run it (a lighter fallback is noted in B2).

---

## Path A — Claude Code (default)

This is the path for the class: you have access to Claude through the lab
organization. **Skip Path B** (B1–B4: Ollama, model, opencode) entirely.

### A1. Install Claude Code

See **<https://code.claude.com/docs/en/setup>**.

- **macOS / Linux / WSL:**
  ```bash
  curl -fsSL https://claude.ai/install.sh | bash
  ```
- **Windows (PowerShell):**
  ```powershell
  irm https://claude.ai/install.ps1 | iex
  ```
- **Windows (CMD):**
  ```batch
  curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
  ```

Open a **new** terminal, then verify:

```bash
claude --version
```

> If `claude` isn't found, the install directory isn't on your `PATH` yet — see
> [Troubleshoot installation](https://code.claude.com/docs/en/troubleshoot-install).
> On Windows, installing [Git for Windows](https://git-scm.com/downloads/win) is
> recommended so Claude Code can run Bash commands.

### A2. Log in with your lab account

```bash
claude
```

Follow the browser prompt and sign in with the **same account you used to
accept the lab invite**. If you belong to more than one organization, pick the
lab one. Type `/exit` to leave.

Done — **skip to [step 1](#1-install-openspec)**.

---

## Path B — local model (Ollama + opencode) (fallback)

Use this path only if you **can't** use Claude. Everything runs locally — no
accounts, no cloud, no GPU required.

### B1. Install Ollama

Ollama runs the local LLM. Get it from **<https://ollama.com/download>**.

- **macOS / Windows:** download and run the installer.
- **Linux:**
  ```bash
  curl -fsSL https://ollama.com/install.sh | sh
  ```

Verify it's running:

```bash
ollama --version
```

### B2. Pull the coder model and give it a 16k context window

Pull the recommended model:

```bash
ollama pull qwen2.5-coder:7b
```

Ollama defaults **every** model to a tiny 4k-token context window, which is too
small for an agent that reads files and edits code. Rebuild the model with a
larger context using a **Modelfile**.

Create a file named `Modelfile` (no extension) with exactly:

```dockerfile
FROM qwen2.5-coder:7b
PARAMETER num_ctx 16384
```

Then build a named variant from it:

```bash
ollama create qwen2.5-coder:7b-16k -f Modelfile
```

You now have a `qwen2.5-coder:7b-16k` model. Use **that** name in opencode below.

> **Low on memory?** Use `deepseek-coder:6.7b` instead — lighter, runs on smaller
> laptops. Pull it (`ollama pull deepseek-coder:6.7b`) and build a `-16k` variant
> the same way (`FROM deepseek-coder:6.7b`).
>
> **If opencode complains about context length**, your machine may allow a larger
> window — bump `num_ctx` to `32768` and rebuild.

### B3. Install opencode

opencode is the terminal AI coding agent. See **<https://opencode.ai/docs>**.

- **macOS / Linux:**
  ```bash
  curl -fsSL https://opencode.ai/install | bash
  ```
- **Windows:**
  ```powershell
  npm install -g opencode-ai
  ```

Verify:

```bash
opencode --version
```

### B4. Point opencode at your local Ollama

Create/edit opencode's config file:

- **macOS / Linux:** `~/.config/opencode/opencode.json`
- **Windows:** `%USERPROFILE%\.config\opencode\opencode.json`

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": { "baseURL": "http://localhost:11434/v1" },
      "models": {
        "qwen2.5-coder:7b-16k": { "name": "Qwen2.5 Coder 7B (16k)" }
      }
    }
  }
}
```

> **Windows note:** if opencode can't reach Ollama, replace `localhost` with the
> IPv4 loopback address — use `http://127.0.0.1:11434/v1`. On Windows,
> `localhost` sometimes resolves to IPv6 (`::1`), which Ollama isn't listening on.

Start opencode inside a project with `opencode`, and pick the
`qwen2.5-coder:7b-16k` model.

---

## Both paths: OpenSpec, clone, verify

### 1. Install OpenSpec

OpenSpec is the spec-driven workflow. See
**<https://github.com/Fission-AI/OpenSpec>**.

```bash
npm install -g @fission-ai/openspec@latest
```

Verify:

```bash
openspec --version
```

Don't run `openspec init` yet — you'll do that inside the repo in step 3.

### 2. Clone this repo and install Python deps

```bash
git clone https://github.com/Omnia-Lab-UC/ibm3407-scrna-spec-lab.git
cd ibm3407-scrna-spec-lab
```

Then create a virtual environment (`.venv`) and install the Python dependencies
into it. The venv comes with its own `pip`.

- **macOS / Linux:**
  ```bash
  python3 -m venv .venv
  source .venv/bin/activate
  pip install -r requirements.txt
  ```
  If `python3 --version` isn't 3.12–3.14, use the versioned command instead,
  e.g. `python3.14 -m venv .venv`.
- **Windows (PowerShell):**
  ```powershell
  py -V:3.14 -m venv .venv
  .venv\Scripts\Activate.ps1
  pip install -r requirements.txt
  ```
  Use `-V:3.12` or `-V:3.13` if that's the version you installed. If activation
  fails with *running scripts is disabled on this system*, run
  `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` once and try again. In
  CMD, activate with `.venv\Scripts\activate.bat` instead.

> **Activate the venv in every new terminal** (`source .venv/bin/activate`, or
> `.venv\Scripts\Activate.ps1` on Windows) before running `make verify` or
> starting `claude` / `opencode`. Your prompt shows `(.venv)` when it's active.
> While it is, plain `python` and `pip` use the venv on every OS.

### 3. Initialize OpenSpec

From inside the repo:

```bash
openspec init
```

When prompted for your AI tool, choose **Claude Code** (Path A) or **OpenCode**
(Path B).

### 4. Verify

```bash
make verify
```

or, if you don't have `make` (common on Windows):

```bash
python scripts/verify.py
```

You should see:

```
Environment OK — the agent can read the repo and run the tests.
```

If you do, you're ready. See [WORKSHOP.md](WORKSHOP.md) for the session steps.

---

### Troubleshooting

- **`make verify` fails on import** — did you activate the venv and
  `pip install -r requirements.txt`?
- **`pip install` says `No matching distribution found`, or tries to build
  matplotlib/h5py from source** — your Python is too old or too new. In the
  venv, run `python --version`. If it isn't 3.12–3.14, delete the `.venv`
  folder and redo step 2 with a supported Python.
- **`npm install -g` fails with `EACCES` (macOS/Linux)** — don't use `sudo`.
  Install Node the nodejs.org way (nvm), which keeps global packages in your
  home directory.
- **Fixture not found** — regenerate it:
  `python scripts/generate_synthetic_fixture.py`.
- **`claude` asks you to pay / says no access (Path A)** — you're logged into
  the wrong account or org. Run `/logout` inside `claude`, then log in again
  with the account that accepted the lab invite.
- **opencode can't reach Ollama (Windows)** — use `127.0.0.1` instead of
  `localhost` (see B4).
- **Model too slow / out of memory (Path B)** — switch to `deepseek-coder:6.7b`
  (B2).
