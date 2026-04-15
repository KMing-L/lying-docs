# Generating GitHub Issue Drafts

Pass `--gen-issue` to automatically draft a GitHub issue after analysis:

```bash
lyingdocs analyze --doc-path docs/ --code-path . --gen-issue
```

LyingDocs uses Hermes to synthesize findings into a single, polite GitHub issue and saves it to `issue.json` in the output directory.

---

## issue.json format

The file contains two fields:

| Field | Description |
| --- | --- |
| `title` | A short issue title |
| `body` | GitHub-flavored Markdown issue body |

The body lists all findings with code references, doc references, and a note acknowledging possible false positives.

---

## Posting with the `gh` CLI

```bash
gh issue create \
  --title "$(jq -r '.title' output/issue.json)" \
  --body  "$(jq -r '.body'  output/issue.json)"
```

This makes LyingDocs useful not only as an audit tool, but as a bridge into repository maintenance workflows.
