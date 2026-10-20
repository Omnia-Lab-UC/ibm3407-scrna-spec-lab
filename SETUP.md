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
| **Do** | [Path A](#path-a--claude-code-default), then skip to [step 5](#5-install-openspec) | [Path B](#path-b--local-model-ollama--opencode-fallback) (steps 1–4), then step 5 |

Either way you also install **OpenSpec** (the spec workflow), clone this repo,
and run one verification command (steps 5–8).

> Install commands move over time. If one fails, check the tool's official docs
> (linked in each section) — but the commands below were current at the time of
> writing.

---

## 0. Prerequisites

- **Python 3.11+** and `pip`. Check: `python --version` (or `python3`).
- **Node.js 20.19+** (needed by OpenSpec). Check: `node --version`.
- **git**.
- **Path B only:** ~8 GB free RAM for the recommended model (a lighter fallback
  is noted below).

---

## Path A — Claude Code (default)

This is the path for the class: you have access to Claude through the lab
organization. **Skip steps 1–4** (Ollama, model, opencode) entirely.

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

Done — **skip to [step 5](#5-install-openspec)**.

---

## Path B — local model (Ollama + opencode) (fallback)

Use this path only if you **can't** use Claude. Everything runs locally — no
accounts, no cloud, no GPU required.

### 1. Install Ollama

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

### 2. Pull the coder model and give it a 16k context window

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

### 3. Install opencode

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

### 4. Point opencode at your local Ollama

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

### 5. Install OpenSpec

OpenSpec is the spec-driven workflow. See
**<https://github.com/Fission-AI/OpenSpec>**.

```bash
npm install -g @fission-ai/openspec@latest
```

Verify:

```bash
openspec --version
```

Don't run `openspec init` yet — you'll do that inside the repo in step 7.

### 6. Clone this repo and install Python deps

```bash
git clone <this-repo-url> ibm3407-scrna-spec-lab
cd ibm3407-scrna-spec-lab

python -m venv .venv
# macOS / Linux:
source .venv/bin/activate
# Windows (PowerShell):
#   .venv\Scripts\Activate.ps1

pip install -r requirements.txt
```

### 7. Initialize OpenSpec

From inside the repo:

```bash
openspec init
```

When prompted for your AI tool, choose **Claude Code** (Path A) or **OpenCode**
(Path B).

### 8. Verify

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
- **Fixture not found** — regenerate it:
  `python scripts/generate_synthetic_fixture.py`.
- **`claude` asks you to pay / says no access (Path A)** — you're logged into
  the wrong account or org. Run `/logout` inside `claude`, then log in again
  with the account that accepted the lab invite.
- **opencode can't reach Ollama (Windows)** — use `127.0.0.1` instead of
  `localhost` (see step 4).
- **Model too slow / out of memory (Path B)** — switch to `deepseek-coder:6.7b`
  (step 2).
