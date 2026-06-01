# dev-kit

Daily-use software engineering skills

## Installation

Add the marketplace, then install the plugin:

```bash
# Add plugin from the marketplace
claude plugin marketplace add ritvikneu/dev-kit
```

```bash
claude
/plugin install devkit@devkit
```

For local testing:

```bash
claude --plugin-dir /path/to/dev-kit
```

Once installed, skills are available as slash commands (e.g. `/raise-pr`, `/html-render`, `/rca`).

## Updating

To get the latest skills after a version bump:

```bash
/plugin update devkit
```

## Skills

- **raise-pr** — Raise a GitHub PR with zero friction: auto-creates a descriptive branch if on main/master, commits with a meaningful message, and opens a PR with a comprehensive description. Say "raise a PR", "create a PR", or use `/raise-pr`. Handles cherry-picks and conflict resolution.
- **html-render** — Converts any content (markdown, diffs, Jira, code flows, chat) into a self-contained interactive HTML file. Say "make this HTML", "visualize this", or use `/html-render`. Inspired by [The unreasonable effectiveness of HTML](https://thariqs.github.io/html-effectiveness/)
- **rca** — Writes a Root Cause Analysis as an HTML file after a debugging session. Appends updates without overwriting history. Use `/rca` or say "write RCA".

## Contributing

1. Create `skills/<name>/` with `SKILL.md`, `evaluations/`, and optionally `references/`
2. `SKILL.md` needs `name` and `description` frontmatter — the description drives automatic invocation
3. Add at least one evaluation JSON (see `skills/html-render/evaluations/` for the format)
4. For HTML-rendering skills, symlink `references/TEMPLATES.md` from `html-render` instead of duplicating CSS
5. Branch per skill (`skill/<name>`), PR into `main`

---

*html-render 
