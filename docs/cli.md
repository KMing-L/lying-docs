# CLI Reference

## Commands

### `lyingdocs analyze`

Run a full documentation-code audit.

```bash
lyingdocs analyze --doc-path <docs> --code-path <code> [options]
```

**Examples:**

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

# Generate GitHub issue drafts after analysis
lyingdocs analyze --doc-path docs/ --code-path . --gen-issue
```

### `lyingdocs version`

Display the installed version.

```bash
lyingdocs version
```

---

## Flags

| Flag | Description |
| --- | --- |
| `--doc-path` | Path to documentation root |
| `--code-path` | Path to code root |
| `-o`, `--output` | Output directory for report artifacts |
| `--config` | Explicit path to a config file |
| `--resume` | Resume a previously interrupted analysis |
| `--gen-issue` | Generate a GitHub issue draft from findings |
| `--hermes-model` | Model name for Hermes |
| `--hermes-base-url` | API base URL for Hermes |
| `--argus-backend` | Argus backend: `codex`, `claude_code`, or `local` |
| `--argus-model` | Model name for Argus |
| `--argus-base-url` | API base URL for Argus |
| `--argus-codex-provider` | Provider flag passed to Codex CLI |
| `--argus-codex-wire-api` | Wire API for Codex backend (`responses` or `chat`) |
| `--max-dispatches` | Max number of Argus tasks per run |
| `--max-iterations` | Max Hermes agent loop iterations |

---

## Output artifacts

Each analysis run produces the following files in the output directory:

| File | Description |
| --- | --- |
| `Misalignment_Report.md` | Structured report of all findings |
| `findings.jsonl` | Per-finding records (one JSON object per line) |
| `doc_index.json` | Documentation inventory built during audit |
| `argus_task_NNN.txt` | Raw output from each Argus task |
| `workspace_state.json` | Resume checkpoint |
| `pipeline.log` | Full execution log |
| `issue.json` | GitHub issue draft (only with `--gen-issue`) |
