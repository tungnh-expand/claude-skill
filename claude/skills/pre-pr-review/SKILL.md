---
name: pre-pr-review
description: Unified pre-PR quality gate. Orchestrates commit-check + convention-check + Flutter verification and saves a Markdown report. Trigger on "pre-pr-review", "/pre-pr-review", "review trước PR", "chạy pre-pr-review", "PR前チェック", "PR前レビュー".
---

# Pre-PR Review (Flutter / mobile ttp)

**MANDATORY: Before doing anything, read these three files in full:**
1. `.claude/skills/commit-check/SKILL.md` — commit message rules (Rule 1)
2. `.claude/skills/convention-check/SKILL.md` — Dart code convention rules (Rules 2–6)
3. This file — orchestration workflow and report template

**Never generate the report from memory.**

**IMPORTANT — no AI attribution:** The review report MUST NOT mention AI, Claude, an LLM, or automated generation in any form. Do NOT add lines such as "Generated with Claude", co-authored-by trailers, or any tool credit. The report MUST read as if written by a human reviewer.

---

## Step 1 — Determine the base branch

If the user did not specify a target branch, ask:
> "このブランチの派生元ブランチは何ですか？（例: develop-20260511）"

Then get the commit list:
```bash
git log $(git merge-base HEAD <target-branch>)..HEAD --oneline
```

Show the commit list to the user before proceeding.

**If a PR number is given instead:**
```bash
git log --merges --all | grep "<PR-number>"
git diff <merge-commit>^1 <merge-commit>
```

---

## Step 2 — Get the full diff

```bash
git diff $(git merge-base HEAD <target-branch>)...HEAD
```

Only flag lines starting with `+` (additions/modifications). Context lines and `-` removals are out of scope.

Get the file list too:
```bash
git diff $(git merge-base HEAD <target-branch>)...HEAD --name-only
```

---

## Step 3 — Run convention checks

Apply **Rule 1** from `.claude/skills/commit-check/SKILL.md` to all commits.

Apply **Rules 2–6** from `.claude/skills/convention-check/SKILL.md` to all changed files. Skip generated files as defined in that skill.

Check all rules simultaneously against the diff. For each violation record:
- File path and line number
- The offending snippet (one line)
- A concrete fix suggestion (not "rename this" — say exactly what to change)

**Only flag lines the diff adds or modifies.** If a nearby unchanged line is egregious and the author could reasonably fix it in the same PR, mention it in 検討.

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

## Step 5 — Produce the report

Output the report in **two sections: Japanese first, then Vietnamese**. See the report format below.

---

## Step 6 — Save the result

Get the current branch name and sanitize it for use as a filename (replace `/` with `-`):

```bash
git rev-parse --abbrev-ref HEAD
# e.g. feature/TTP_VN-1417 → feature-TTP_VN-1417
```

Filename: `PR_<sanitized-branch-name>_pre-pr-review_<yyyyMMdd>.md`

Example: `PR_feature-TTP_VN-1417_pre-pr-review_20260527.md`

Save the full report (both JP and VN sections) to `.claude/output/`:

```bash
mkdir -p .claude/output
```

---

## Priority levels

- **要修正 / Cần sửa** — clear, unambiguous violations: wrong casing, prohibited abbreviation, dead code, WHAT comment, malformed commit message, missing required documentation, test convention violation
- **検討 / Xem xét** — judgment calls: borderline abbreviations (`auth`, `config`), names that may be valid given domain knowledge, optional documentation candidates

When in doubt, prefer 検討 over 要修正. The author knows the codebase.

---

## Report format

### Japanese version

```markdown
# コードレビュー結果

**ブランチ:** `<branch>` → `<target-branch>`  
**対象:** <N> commits, <N> files  
**判定:** 要修正 <N>件 / 検討 <N>件

| 優先度 | 件数 |
| ------ | ---- |
| 要修正 | N    |
| 検討   | N    |

## 要修正

- [ ] **`<file>:<line>`** — [<カテゴリ>]
  - <説明と修正方法>
- [ ] **`commit <hash>`** — [コミット]
  - <説明と修正提案>

## 検討

- [ ] **`<file>:<line>`** — [<カテゴリ>]
  - <説明>

## 問題なし

- <ルール名>: OK

## 検証

- [ ] flutter analyze → OK / NG / 未実行
- [ ] flutter test → OK / NG / 未実行 (<理由>)
- [ ] flutter build ios --simulator → OK / NG / 未実行
```

カテゴリ: `命名` `コメント` `略語` `識別子` `コミット` `テスト` `ドキュメント`

---

### Vietnamese version

```markdown
# Kết quả review code

**Nhánh:** `<branch>` → `<target-branch>`  
**Phạm vi:** <N> commits, <N> files  
**Kết luận:** Cần sửa <N> mục / Xem xét <N> mục

| Ưu tiên | Số mục |
| ------- | ------ |
| Cần sửa | N      |
| Xem xét | N      |

## Cần sửa

- [ ] **`<file>:<line>`** — [<category>]
  - <Mô tả và cách sửa>
- [ ] **`commit <hash>`** — [Commit]
  - <Mô tả và đề xuất sửa>

## Xem xét

- [ ] **`<file>:<line>`** — [<category>]
  - <Mô tả>

## Không có vấn đề

- <rule name>: OK

## Kiểm tra

- [ ] flutter analyze → OK / NG / Chưa chạy
- [ ] flutter test → OK / NG / Chưa chạy (<lý do>)
- [ ] flutter build ios --simulator → OK / NG / Chưa chạy
```

Category labels: `Naming` `Comments` `Abbreviation` `Identifier` `Commit` `Test` `Documentation`

---

## What to avoid

- **Don't nitpick unchanged code.** Only review what the diff adds or modifies.
- **Don't add unlisted rules.** No line-length, import-order, or style preferences beyond the defined rules.
- **Don't be dogmatic on judgment calls.** Use 検討 for borderline cases.
- **Don't restate the rule list.** Focus on findings — a short category label is enough.
- **Don't mention AI, Claude, or automation** anywhere in the report output.

---

## Gate rule

Do not create a PR if any of the following are true:
- Any `要修正` item is unchecked
- `flutter analyze` fails
- `flutter build ios --simulator` fails