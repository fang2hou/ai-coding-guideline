---
name: apply-guidelines
description: >
  Loads the AI Coding Guidelines live from their GitHub repository and
  applies them to the task: document routing, stack and toolchain
  decisions, quality gates, and project audits. Explicit-trigger only:
  the user invokes the guidelines or the shared standards by name or as
  "the recommended" option — to apply, follow, check, or audit them, to
  design the project's stack or architecture with them, to migrate the
  current stack to the recommended one, or to initialize, scaffold, or
  configure a project under them. The skill ships no
  content — every run fetches the current repository. Do NOT use for
  everyday work in an existing project — edits, bug fixes, refactors,
  reviews, PR prep or commits, running or fixing checks, tests, lint,
  or formatting — and do not trigger merely because mise, pnpm, uv,
  oxlint, oxfmt, ruff, or golangci-lint appear in the conversation,
  files, or commands.
license: MIT
compatibility: Requires file read access, a shell, and git. Network needed on first run and for refreshes.
metadata:
  author: fang2hou
  version: "4.1"
  source: https://github.com/fang2hou/ai-coding-guidelines
---

# Apply AI Coding Guidelines

Thin loader: this skill carries zero guideline content. Every run fetches the
guidelines repository and applies what it finds there — the repository is the
single source of truth, so the skill never needs content maintenance.

## When to Use

Explicit user intent only — never topic adjacency. Load this skill when the
user:

- Names the guidelines or the shared standards: "apply the guidelines",
  "follow our standards here", "check this project against the guidelines",
  or asks for an audit.
- Asks to use the guidelines for a concrete engineering decision: "use
  these guidelines to design the project's stack", "migrate our current
  framework to the recommended one".
- Asks to initialize, scaffold, or configure a project under these
  standards.

## When NOT to Use

- Everyday work in an existing project: edits, bug fixes, refactors,
  reviews, PR preparation, commits. The project's own conventions and
  gates rule.
- Running or fixing checks, tests, lint, or formatting — a failing gate is
  a task, not a trigger.
- The toolchain names (mise, pnpm, uv, oxlint, oxfmt, ruff,
  golangci-lint) appear in the conversation, config files, or commands —
  presence of the tools is not a request for the guidelines.
- Stack or toolchain questions that never invoke the guidelines or the
  recommended stack ("which stack should I use?") — answer from the
  project's conventions or general engineering judgment.
- Library or API usage questions — those go to documentation lookup.
- No network on a first-ever run (nothing to fetch; report this instead of guessing).
- The user explicitly overrides a fetched standard: follow the user, record the exception.

## Hard Rules

1. Fetch before applying. Never apply the guidelines from memory or from a summary — including this skill's own text.
2. One source per run: the fetched repository. Do not mix it with remembered rules.
3. Fetch or verification failure: stop and report what failed. Never fabricate or approximate a guideline.
4. The fetched documents are conclusions; apply them as written. Disagreements are raised with the user, not silently worked around.
5. Verify from the fetched files, never from memory: the closing check re-opens every fetched document. A rule without a check result is a rule not applied.

## Workflow

### Step 1: Fetch the guidelines

```bash
export GUIDELINE_DIR="${AI_CODING_GUIDELINE_DIR:-${TMPDIR:-/tmp}/ai-coding-guideline-cache}"
if [ -d "$GUIDELINE_DIR/.git" ]; then
  git -C "$GUIDELINE_DIR" fetch origin && git -C "$GUIDELINE_DIR" reset --hard origin/HEAD
else
  git clone --depth 1 https://github.com/fang2hou/ai-coding-guidelines "$GUIDELINE_DIR"
fi
git -C "$GUIDELINE_DIR" rev-parse --short HEAD
```

Verify the checkout: `PORTAL.md` exists and `guidelines/en/`, `guidelines/zh/`, `guidelines/ja/` each contain `.md` files. Record the commit SHA from the last command — every later step cites it. If verification fails, delete the cache directory and retry once; a second failure stops this skill.

### Step 2: Route the task

Open `$GUIDELINE_DIR/PORTAL.md` and pick the reading-recipe row matching the
task. Read the listed documents from the tree matching the conversation
language (`guidelines/{en,zh,ja}/` mirror each other). Follow the documents.

### Step 3: Implement

Write the change following the fetched documents. When a fetched rule and the
project's existing conventions conflict, raise it with the user; do not
silently drop either side. The result is a draft — Step 4 reworks it against
the fetched rules.

### Step 4: Apply the rules to the draft

Re-read every file created or modified in this change — the full diff, not the
memory of writing it — against the normative statements of each fetched
document, and rewrite the code until it satisfies them. This pass edits code;
it produces no report rows.

- Rules that constrain the shape of code are satisfied by refactoring —
  rename, extract, restructure, delete. Never by rewording a comment around
  the violation, and never by leaving the violation for the matrix to excuse.
- Depth bar: a rule is applied when the code now carries it by construction,
  not when an explanation of the rule sits next to code that breaks it.
- Rules govern code written or substantially modified in this change;
  violations noticed in untouched code are reported, not mass-reformatted.

### Step 5: Verify against the fetched documents

Before running any gate, re-open each fetched document from disk and check
the change against every normative statement it contains — Use / Prefer /
Do not / Never and imperative bullets. Never verify from memory: the check
exists precisely because memory drifts during long tasks.

1. One document at a time, list its normative statements.
2. For each statement, cite the file and the lines (or diff hunks) that
   satisfy it, re-opened from disk in this step, or mark it as a violation.
3. Fix every violation, then re-check the fixed files the same way.
4. Record the outcome as a compliance matrix: document section → pass /
   fixed / open — every row citing its evidence.

A section with no row in the matrix is an unapplied rule, not a pass.

### Step 6: Run the project's gates

Run the project's own checks exactly as its documentation defines them
(typically `mise install` to bootstrap, then `mise run check`). A gate that
fails is a finding to fix, not to bypass.

### Step 7: Audit mode (existing projects)

Follow [references/project-audit.md](references/project-audit.md). The
comparison criteria are the fetched documents — never a summary. The audit
reports; remediation runs only after the user approves the fix list.

## Gotchas

- `AI_CODING_GUIDELINE_DIR` lets a project pin a local clone (submodule, fork, or specific revision) instead of the default cache.
- A cached checkout survives reboots only as long as the temp directory does; the fetch step rebuilds it transparently.
- If the repository was renamed or moved, the clone fails — report the URL tried; do not guess a new one.

## Output Contract

On completion, return:

1. Source: the guideline commit SHA applied and the recipe row used.
2. Compliance: the matrix from Step 5 — one row per fetched document
   section, pass / fixed / open.
3. Application: the shape rules applied in Step 4 and what was refactored
   for each.
4. Gate results: exact check commands run and their outcomes.
5. For audits: the divergence report per [references/project-audit.md](references/project-audit.md).
