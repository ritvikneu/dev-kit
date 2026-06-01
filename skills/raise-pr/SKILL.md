---
name: raise-pr
description: Raise a GitHub PR from current working changes — auto-creates a descriptive branch if on main/master, stages and commits with a meaningful message, then opens a PR with a comprehensive description. Also handles cherry-pick PRs: mirrors the original PR description and resolves conflicts. Use when user says "raise a PR", "create a PR", "open a PR", "make a PR", "push this as a PR", "ship this PR", "cherry-pick this", or wants to submit current changes to GitHub as a pull request.
---

# Raise a GitHub PR

Inspect git state → branch out if needed → commit → push → open PR with a full description.

## Step 1 — Inspect current state

Run in parallel:

```bash
git status
git diff HEAD
git log --oneline -10
git branch --show-current
```

## Step 2 — Decide on the branch

**Already on a feature branch** (not `main`, `master`, `develop`, `trunk`): keep it, go to Step 3.

**On a protected branch**: derive a branch name from the diff:

- Identify the dominant theme (feature, fix, refactor, docs, chore)
- Format: `<type>/<short-kebab-description>` — e.g. `feat/user-auth`, `fix/null-pointer-login`
- Keep under 50 chars, lowercase, no spaces

```bash
git checkout -b <branch-name>
```

## Step 3 — Stage and commit

Stage files relevant to the PR. Prefer named files over `git add -A` unless the whole tree belongs together.

Commit message rules:
- **Subject** (≤72 chars): `<type>(<scope>): <imperative description>` — present tense
- **Body**: explain *why*, not *what* (the diff shows what)
- **Footer**: issue references if applicable (`Closes #123`)

Types: `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `perf`

```bash
git add <files>
git commit -m "$(cat <<'EOF'
<type>(<scope>): <subject>

<body — optional, only if why is non-obvious>

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

## Step 4 — Push

```bash
git push -u origin <branch-name>
```

If rejected due to upstream divergence, **do not force-push** — report the conflict and ask the user.

## Step 5 — Cherry-pick handling (skip if not a cherry-pick)

If the user says this is a cherry-pick, or the branch/commit message references one:

**5a — Detect the source PR**

Find the original PR number from the commit message, branch name, or user context, then fetch its description:

```bash
gh pr view <original-pr-number> --json title,body
```

**5b — Resolve conflicts if present**

After `git cherry-pick <sha>`, if conflicts exist:

```bash
git status  # identify conflicted files
```

For each conflicted file, apply this logic:
- Accept **incoming** (`theirs`) if the cherry-pick introduces a net-new change with no semantic overlap
- Accept **current** (`ours`) if the base branch already has a newer version of the same logic
- **Merge manually** when both sides have distinct, non-overlapping additions — keep both, de-duplicate imports and declarations
- Never silently drop code; if the right resolution is ambiguous, show the user both sides and ask

After resolving:

```bash
git add <resolved-files>
git cherry-pick --continue
```

**5c — Mirror and annotate the PR description**

Use the original PR's body verbatim, then append at the bottom:

```markdown
---
**Cherrypick of #<original-pr-number>: <original-pr-title>**
```

The PR title should be: `[Cherrypick] <original-pr-title>`

Then continue to Step 6 to open the PR.

---

## Step 5 (standard) — Draft the PR

Analyze the full diff (`git diff main...HEAD`) and produce:

**Title** — ≤70 chars, no trailing period, matches commit subject but can be slightly more descriptive.

**Body:**

```markdown
## Summary
- <what changed, 2-4 bullets at feature level>

## Motivation
<1-3 sentences: the problem this solves or the goal it achieves>

## Changes
### <Area or file group>
- <specific change>
### <Area or file group>
- <specific change>

## How to test
- [ ] <concrete step a reviewer can run>
- [ ] <expected outcome>

## Notes
<Known limitations, follow-up TODOs, or anything reviewers should know. Omit if none.>
```

Drop any section that has nothing to say.

## Step 6 — Open the PR

```bash
gh pr create --title "<title>" --body "$(cat <<'EOF'
<body>
EOF
)"
```

Return the PR URL to the user.

## Guardrails

- Never force-push without explicit user approval.
- Never skip hooks (`--no-verify`) unless the user asks.
- Do not stage `.env`, credential files, or secrets — warn and stop if found.
- If `gh` is not installed or not authenticated, tell the user to run `gh auth login` and stop.
