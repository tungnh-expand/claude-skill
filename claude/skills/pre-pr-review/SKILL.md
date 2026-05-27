---
name: pre-pr-review
description: Unified pre-PR quality gate. Orchestrates commit-check + convention-check + Flutter verification and saves a Markdown report. Trigger on "pre-pr-review", "/pre-pr-review", "review trước PR", "chạy pre-pr-review".
---

# Pre-PR Review (Flutter / mobile ttp)

**MANDATORY: Before doing anything, read these three files in full:**
1. `.claude/skills/commit-check/SKILL.md` — commit message rules (Rule 1)
2. `.claude/skills/convention-check/SKILL.md` — Dart code convention rules (Rules 2–6)
3. This file — orchestration workflow and report template

**Never generate the report from memory.**

---

## Step 1 — Get the diff

Ask the user:
> "Branch gốc là gì? (branch mà bạn tạo branch này từ đó — ví dụ: `develop-202606`)"

Then run:

```bash
# List changed files
git diff $(git merge-base HEAD <base-branch>) --name-only

# Full diff for convention check
git diff $(git merge-base HEAD <base-branch>)

# Commit messages since branching
git log $(git merge-base HEAD <base-branch>)..HEAD --pretty=format:"%H %s"

# Full body for each commit
git log -1 --pretty=format:"%B" <hash>
```

**If a PR number is given instead:**
```bash
git log --merges --all | grep "<PR-number>"
git diff <merge-commit>^1 <merge-commit>
```

---

## Step 2 — Run commit-check

Apply **Rule 1** from `.claude/skills/commit-check/SKILL.md` to all commits retrieved in Step 1.

Collect all findings.

---

## Step 3 — Run convention-check

Apply **Rules 2–6** from `.claude/skills/convention-check/SKILL.md` to all changed files retrieved in Step 1.

Skip generated files as defined in that skill. Collect all findings.

---

## Step 4 — Flutter verification

Run each command and record the result:

```bash
flutter analyze

flutter test

flutter build ios --simulator --no-codesign
```

If `flutter test` has prerequisites or takes too long, note it and skip — document in the report.

---

## Step 5 — Save report

Get the current branch name and sanitize it for use as a filename (replace `/` with `-`):

```bash
git rev-parse --abbrev-ref HEAD
# e.g. feature/TTP_VN-1417 → feature-TTP_VN-1417
```

Filename: `PR_<sanitized-branch-name>_pre-pr-review_<yyyyMMdd>.md`

Example: `PR_feature-TTP_VN-1417_pre-pr-review_20260527.md`

Save to: `.claude/output/`

```bash
mkdir -p .claude/output
```

---

## Report template

```markdown
# Pre-PR Review Report

| Item | Value |
|---|---|
| Branch | `<current branch>` |
| Base branch | `<base branch>` |
| Author | `<git user.name>` |
| Repository | mobile ttp |
| Date | `<YYYY-MM-DD HH:MM>` |
| Model | claude-sonnet-4-6 |

---

## Convention Check

### Commit messages (Rule 1)

<Findings from commit-check, or "✓ All commits follow Conventional Commits.">

### Code (Rules 2–6)

#### 要修正 (Must fix)

- [ ] **`<file path>:<line>`** — Rule <N> (<rule name>). `<snippet>` → `<fix>`.

#### 検討 (Consider)

- [ ] **`<file path>:<line>`** — Rule <N> (<rule name>). `<snippet>` — <explanation>.

#### ✓ Clean rules

<Rules with no violations.>

---

## Flutter Verification

### flutter analyze

**Result:** ✓ Pass / ✗ Fail

<If fail: paste the error output.>

### flutter test

**Result:** ✓ Pass / ✗ Fail / ⚠️ Skipped (<reason>)

<If fail: paste the relevant error output.>

### flutter build ios --simulator

**Result:** ✓ Pass / ✗ Fail

<If fail: paste the error output.>

---

## Summary

| Category | Count |
|---|---|
| 要修正 (Must fix) | N |
| 検討 (Consider) | N |
| flutter analyze | ✓ / ✗ |
| flutter test | ✓ / ✗ / ⚠️ |
| flutter build | ✓ / ✗ |

**Gate status:** ✅ Ready to PR / ❌ Fix required before PR

<If ❌: one-line summary of what needs to be resolved.>
```

---

## Gate rule

Do not create a PR if any of the following are true:
- Any `要修正` item is unchecked
- `flutter analyze` fails
- `flutter build` fails