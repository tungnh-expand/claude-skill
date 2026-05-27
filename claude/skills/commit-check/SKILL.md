---
name: commit-check
description: Checks one or more commit messages against the team's Conventional Commits convention. Trigger when the user asks to check, review, or validate commit messages — phrases like "check my commit", "is this commit message OK", "review commit history", "validate commits". Works with a pasted message, a range, or will auto-fetch recent commits from git log.
---

# Commit Message Check

Audits commit messages against Conventional Commits 1.0 and produces a prioritized report.

## How to get the commits

**Do not ask the user to paste anything first.** Follow these steps automatically:

1. If the user provided a commit message directly → use it as-is.
2. If the user specified a branch → run:
   ```bash
   git log <base-branch>..<current-branch> --pretty=format:"%H %s"
   ```
3. If no input is given → run:
   ```bash
   git log -10 --pretty=format:"%H %s"
   ```
   Show the list and ask the user which commits to review (or "all of them").

Retrieve the full message (subject + body + footer) for each commit:
```bash
git log -1 --pretty=format:"%B" <hash>
```

## The rule — App Conventional Commits 2.0

The project uses a customized Conventional Commits standard to integrate with ticket codes (TTP_VN). Below are the 3 accepted formats:

**Format 1: Full Type, Scope, and Ticket (Recommended for new features)**
<type>(<optional scope>): TTP_VN-<number> <subject>

**Format 2: Quick bug fix with Ticket**
fix: TTP_VN-<number> <subject>

**Format 3: Shortened with only Ticket and Subject (Direct Ticket Reference)**
TTP_VN-<number> <subject>

**Accepted types (For Format 1 & 2):** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

A trailing `!` (e.g. `feat!:`) or a `BREAKING CHANGE:` footer marks a breaking change.

**Recommended scopes:** `auth`, `ui`, `state`, `repo`, `network`, `cache`, `deps`, `ios`, `android`, `theme`, `i18n`, `nav`, or specific feature names.

**Subject line must:**
- Always include the ticket code (e.g., `TTP_VN-1417`) immediately before the description.
- Start the description (after the ticket code) with a lowercase letter.
- Be under ~72 characters in total length (including the ticket code).
- Use imperative mood ("add X", not "added X" or "adds X").
- Not end with a period.

**Examples:**

| Bad | Problem | Suggested fix |
|---|---|---|
| `Added login screen` | Missing ticket code, past tense | `feat(auth): TTP_VN-1417 add login screen` |
| `fix: Fixed the bug` | Missing ticket code, vague wording | `fix: TTP_VN-1418 prevent duplicate item on double-tap` |
| `TTP_VN-1419: Updates` | Extra `:`, vague subject | `TTP_VN-1419 bump riverpod to 2.6` |
| `feat(ui): TTP_VN-1420 Added button` | Past tense (Added) | `feat(ui): TTP_VN-1420 add submit button to cart` |

## Severity

Two levels — consistent with the full `pre-pr-review` report:

- **要修正 (Must fix)** — missing type, wrong casing, past/present-third-person tense, empty subject, subject > 72 chars, ends with period.
- **検討 (Consider)** — no scope when one is obvious, vague subject that can be improved, missing body for a complex change.

## Output format

```markdown
# Commit message audit

**Commits reviewed:** N
**Result:** N 要修正 · N 検討

## Findings

- **`<hash short>` `<original subject>`**
  - [要修正] Missing type. Suggest: `feat(auth): add login screen`.
  - [検討] No scope — this change is clearly in the `auth` module.

- **`<hash short>` `<original subject>`** ✓ No issues.

## Summary

<One-line takeaway — e.g. "2 of 5 commits need a type fix before merging.">
```

If all commits are clean, say so plainly.