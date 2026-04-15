# Argus Backends

Argus is the deep code analysis side of the system. It investigates the actual codebase against tasks dispatched by Hermes.

You can choose the backend that best fits your setup.

---

## `local`

No external CLI required. Uses a built-in agent loop with filesystem tools and an OpenAI-compatible API.

Good default for getting started.

```toml
[argus]
backend = "local"
model = "gpt-5.4"
base_url = "https://api.openai.com/v1"

[argus.local]
max_iterations = 25    # per-task agent loop cap
max_read_bytes = 200000
```

---

## `codex`

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
# path = "/usr/local/bin/codex"  # optional: explicit binary path
```

Binary resolution order:

1. Explicit path from config (`argus.codex.path`)
2. System `PATH`
3. Local `node_modules/.bin/codex`

---

## `claude_code`

Uses [Claude Code](https://docs.anthropic.com/claude/docs/claude-code).

```toml
[argus]
backend = "claude_code"
model = "claude-sonnet-4-6"

[argus.claude_code]
# path = "/usr/local/bin/claude"  # optional: explicit binary path
```

Invoked as:

```bash
claude -p <prompt> --model <argus_model> --output-format text
```

with `cwd` set to your code root.
