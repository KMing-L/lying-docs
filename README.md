<p align="center">
  <img src="assets/logo.png" alt="LyingDocs" width="200" />
</p>

<h1 align="center">LyingDocs</h1>

<p align="center">
  A trust layer for your repository.
</p>

<p align="center">
  Detect when your docs, code, configs, and examples stop agreeing with each other.
</p>

---

Modern repositories are read by more than humans.

They are read by teammates, new contributors, users, reviewers, downstream integrators — and increasingly by AI agents.

That only works if the repository can be trusted.

But trust quietly erodes over time:

- documentation describes features that were never shipped
- code behavior drifts away from the spec
- examples stop matching reality
- values claimed to be configurable are hardcoded deep in the codebase
- papers and implementation tell different stories

**LyingDocs is a trust layer for your repository.**  
It audits the gap between what your repo *says* and what your code *actually does* — before your users, contributors, or agents learn the wrong thing.

---

## Why LyingDocs exists

Every codebase accumulates invisible trust debt.

In the age of fast iteration and LLM-assisted development, teams now ship code and documentation faster than ever — but not always in sync. A repo may still look polished while becoming progressively less reliable as a source of truth.

That is the problem LyingDocs is built to solve.

LyingDocs is not just a documentation checker. It is a system for surfacing **repository misalignment**:

- docs that overclaim
- code paths that are undocumented
- specs that no longer match implementation
- "configurable" behavior that is actually fixed
- claims in papers or READMEs that cannot be supported by the code

The goal is simple:

> Keep your repository trustworthy for humans and machines.

---

## What LyingDocs does

LyingDocs deploys two autonomous agents against your repository:

- **Hermes** reads your documentation, plans an audit strategy, and decides what needs to be verified
- **Argus** investigates the actual codebase and reports what the code really does

Hermes then reconciles the two and writes a structured report of the mismatches it finds.

This lets you catch cases where your repository is no longer telling the truth about itself.

---

## How it works

### 1. Hermes reads what the repo claims

Hermes traverses your documentation and extracts claims, assumptions, and implementation promises from sources such as:

- docs/
- README files
- setup guides
- usage examples
- configuration references
- papers and research writeups

It then plans an audit by turning those claims into targeted investigation tasks.

### 2. Argus checks what the code actually does

Argus executes each task against your real codebase.

You can choose the backend that best fits your setup:

- **`codex`** — [OpenAI Codex CLI](https://github.com/openai/codex) subprocess
- **`claude_code`** — [Claude Code](https://docs.anthropic.com/claude/docs/claude-code) CLI subprocess (`claude -p`)
- **`local`** — built-in minimal agent loop using filesystem tools and any OpenAI-compatible API directly

### 3. LyingDocs reports the trust gaps

Hermes reconciles documented claims with observed implementation behavior and outputs a report of misalignments.

These findings can then be reviewed by maintainers, turned into issues, and eventually enforced in CI.

---

## Positioning

LyingDocs is best thought of as:

- a **trust layer** for your repo
- a **docs-to-code alignment guard**
- a **pre-user warning system** for misleading documentation
- a future **CI / GitHub Action quality gate** for repository truthfulness

It is not meant to be a tool you manually open every day.

It is meant to become something your repository runs automatically:

- on pull requests
- before releases
- during scheduled audits
- before docs deployment
- as part of your GitHub Actions workflow

---

## Installation

```bash
pip install lyingdocs
````

---

## Quick Start

```bash
export OPENAI_API_KEY="sk-..."

lyingdocs analyze --doc-path docs/ --code-path . -o output/audit
```

This performs a full audit of your repository and produces a report describing where documentation and implementation no longer align.

---

## Example use cases

Use LyingDocs when you want to answer questions like:

* Does the README still reflect the real behavior of the project?
* Are our examples and quickstarts still valid?
* Did code change without the docs changing with it?
* Are we claiming configuration that does not actually exist?
* Does our paper describe behavior the implementation does not support?
* Can an AI agent trust this repository as a source of truth?

---

## Misalignment categories

| Category           | Description                                           |
| ------------------ | ----------------------------------------------------- |
| **LogicMismatch**  | Code contradicts documentation                        |
| **PhantomSpec**    | Documentation describes non-existent features         |
| **ShadowLogic**    | Important code behavior exists but is undocumented    |
| **HardcodedDrift** | Supposedly configurable values are actually hardcoded |

These categories represent different ways repository trust breaks down.

---

## Configuration

LyingDocs loads configuration from multiple sources, with later sources overriding earlier ones:

1. **Built-in defaults** (OpenAI API, gpt-5.4)
2. **Config file** — `lyingdocs.toml` in project root, or `~/.config/lyingdocs/config.toml`
3. **Environment variables** / `.env`
4. **CLI arguments**

Hermes and Argus are configured independently, so you can use:

* a cheaper planning model for Hermes
* a stronger coding / investigation model for Argus
* different providers or endpoints for each agent

### Config file example

Example configs live in [tests/configs](https://github.com/KMing-L/lying-docs/tree/main/tests/configs).

```toml
[hermes]
model = "gpt-5.4"
base_url = "https://api.openai.com/v1"
# api_key_env = "OPENAI_API_KEY"  # optional — defaults to OPENAI_API_KEY

[argus]
backend = "local"           # "codex" | "claude_code" | "local"
model = "gpt-5.4"
base_url = "https://api.openai.com/v1"
# api_key_env = "OPENAI_API_KEY"

# Only read when argus.backend = "codex"
[argus.codex]
provider = "openai"
wire_api = "responses"
# path = "/usr/local/bin/codex"   # optional: explicit codex binary path

# Only read when argus.backend = "claude_code"
[argus.claude_code]
# path = "/usr/local/bin/claude"  # optional: explicit claude binary path

# Only read when argus.backend = "local"
[argus.local]
max_iterations = 25         # per-task agent loop cap
max_read_bytes = 200000     # per read_file call

[limits]
max_dispatches = 20         # max Argus dispatches per Hermes run
max_iterations = 50         # max Hermes loop iterations
argus_task_timeout = 1200   # seconds per Argus task (codex / claude_code backends)
token_budget = 524288       # Hermes context budget before compression
```

### Environment variables

| Variable                 | Description                                    |
| ------------------------ | ---------------------------------------------- |
| `OPENAI_API_KEY`         | Required unless overridden via `api_key_env`   |
| `HERMES_MODEL`           | Hermes model name                              |
| `HERMES_BASE_URL`        | Hermes API base URL                            |
| `ARGUS_BACKEND`          | `codex`, `claude_code`, or `local`             |
| `ARGUS_MODEL`            | Argus model name                               |
| `ARGUS_BASE_URL`         | Argus API base URL                             |
| `ARGUS_CODEX_PROVIDER`   | Codex backend provider                         |
| `ARGUS_CODEX_WIRE_API`   | Codex backend wire API (`responses` or `chat`) |
| `ARGUS_CODEX_PATH`       | Explicit path to `codex`                       |
| `ARGUS_CLAUDE_CODE_PATH` | Explicit path to `claude`                      |
| `ARGUS_TASK_TIMEOUT`     | Timeout per Argus task in seconds              |
| `TOKEN_BUDGET`           | Hermes context budget before compression       |

---

## Argus backends

Argus is the deep code analysis side of the system.

### `local`

No external CLI required.
Uses a built-in agent loop with filesystem tools and an OpenAI-compatible API.

Good default for getting started.

```toml
[argus]
backend = "local"
model = "gpt-5.4"
base_url = "https://api.openai.com/v1"
```

### `codex`

Uses [OpenAI Codex CLI](https://github.com/openai/codex).

```bash
npm install -g @openai/codex
```

```toml
[argus]
backend = "codex"

[argus.codex]
provider = "openai"
wire_api = "responses"
```

Resolution order:

1. explicit path from config
2. system `PATH`
3. local `node_modules/.bin/codex`

### `claude_code`

Uses [Claude Code](https://docs.anthropic.com/claude/docs/claude-code).

```toml
[argus]
backend = "claude_code"
model = "claude-sonnet-4-6"

[argus.claude_code]
# path = "/usr/local/bin/claude"
```

Invoked as:

```bash
claude -p <prompt> --model <argus_model> --output-format text
```

with `cwd` set to your code root.

---

## CLI reference

```bash
# Full analysis
lyingdocs analyze --doc-path docs/ --code-path . -o output/audit

# Choose Argus backend
lyingdocs analyze --doc-path docs/ --code-path . --argus-backend=local

# Different models for Hermes and Argus
lyingdocs analyze --doc-path docs/ --code-path . \
  --hermes-model gpt-5.4 \
  --argus-model gpt-5.4

# Resume interrupted analysis
lyingdocs analyze --doc-path docs/ --code-path . --resume

# Use an explicit config file
lyingdocs analyze --doc-path docs/ --code-path . --config myconfig.toml

# Generate GitHub issue drafts
lyingdocs analyze --doc-path docs/ --code-path . --gen-issue

# Show version
lyingdocs version
```

Available flags:

`--hermes-model`, `--hermes-base-url`, `--argus-backend {codex,claude_code,local}`, `--argus-model`, `--argus-base-url`, `--argus-codex-provider`, `--argus-codex-wire-api`, `--max-dispatches`, `--max-iterations`, `--config`, `--resume`, `--gen-issue`

---

## Generating GitHub issue drafts

Pass `--gen-issue` to automatically draft a GitHub issue after analysis:

```bash
lyingdocs analyze --doc-path docs/ --code-path . --gen-issue
```

LyingDocs uses Hermes to synthesize findings into a single, polite GitHub issue and saves it to `issue.json` in the output directory.

The file contains:

* **`title`** — a short issue title
* **`body`** — a GitHub-flavored Markdown issue body listing findings, code references, doc references, and a note acknowledging possible false positives

You can post it directly with the [`gh` CLI](https://cli.github.com/):

```bash
gh issue create \
  --title "$(jq -r '.title' output/issue.json)" \
  --body  "$(jq -r '.body'  output/issue.json)"
```

This makes LyingDocs useful not only as an audit tool, but as a bridge into repository maintenance workflows.

---

## GitHub Actions direction

LyingDocs is moving toward a natural next step:

**continuous trust enforcement inside GitHub Actions**

The long-term shape of the project is not “run this manually forever.”
The long-term shape is:

* run on pull requests
* comment on suspicious docs/code drift
* warn maintainers before release
* surface trust regressions early
* make repository truthfulness part of CI

That is where LyingDocs becomes most valuable: not only as an analyzer, but as infrastructure.

---

## Roadmap

* [x] **Multi-harness support** — Argus runs on Codex, Claude Code, or a built-in local agent
* [x] **Issue generation** — `--gen-issue` drafts GitHub issues from findings
* [ ] **GitHub Action integration** — run LyingDocs automatically in PRs and CI to catch trust regressions as they are introduced
* [ ] **One-session memory support** — Argus backends retain state across tasks for deeper multi-step investigations
* [ ] **Deeper analysis** — multi-hop reasoning across doc hierarchies and version-aware diffing to detect when code changed but docs did not
* [ ] **Paper mode** — treat academic papers as documentation and detect paper-to-code misalignment
* [ ] **Auto-fix mode** — Hermes proposes doc patches for human review and application

---

## For researchers

A paper is also documentation.

It is a human-language description of code, behavior, claims, and expected results — often written under deadline, and often drifting away from the implementation over time.

If you want to know whether:

* your repo matches your paper
* your claims are supported by the code
* another researcher can trust your implementation

then LyingDocs can help.

The problem is the same.
Paper is documentation for code.
LyingDocs is for papers too.

---

## Why “trust layer”

Because the problem is bigger than stale docs.

A repository becomes untrustworthy whenever its outward description and inward behavior drift apart.

That harms:

* users trying to adopt the project
* contributors trying to extend it
* maintainers trying to review changes
* researchers trying to reproduce results
* AI agents trying to understand the repo

LyingDocs exists to make that gap visible.

Not after users complain.
Before.

---

## License

MIT