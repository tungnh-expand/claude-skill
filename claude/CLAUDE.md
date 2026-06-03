## PR Workflow

> This workflow is shared across repos. Each repo has its own name:
> - `ttp-yomsubi-admin` → **hontai**
> - `ttp-yomsubi-web` → **web admin**
> - `ttp-flutter` → **mobile ttp**

```
pre-pr-review → fix 要修正 → re-check → create PR → upload report to SharePoint
```

### Skills

| Command | Mục đích |
|---|---|
| `/pre-pr-review` | Full gate: commit check + convention check + flutter verify + save report |
| `/commit-check` | Chỉ kiểm tra commit messages (Conventional Commits) |
| `/convention-check` | Chỉ kiểm tra Dart code conventions (naming, dead code, abbreviations, v.v.) |

### Step 1 — Pre-PR Review (before PR)

Run in Claude Code chat: **`/pre-pr-review`**

Claude sẽ tự động:
1. Hỏi branch gốc (branch mà bạn tạo branch hiện tại từ đó)
2. Chạy `git diff` để lấy toàn bộ thay đổi — không cần paste thủ công
3. Chạy `/commit-check` → kiểm tra tất cả commit messages
4. Chạy `/convention-check` → kiểm tra toàn bộ code thay đổi
5. Chạy `flutter analyze`, `flutter test`, `flutter build ios --simulator`
6. Gộp kết quả vào báo cáo và lưu vào `.claude/output/PR_<branch>_pre-pr-review_<yyyyMMdd>.md`

**Option A — from local branch (no PR number):**

1. Claude asks for the base branch. Example: `feature/TTP_VN-1417` was branched from `develop-202606`.
2. Claude runs `git diff $(git merge-base HEAD <base-branch>)` and analyzes all changed files automatically.

**Option B — from PR number (when PR already exists):**

1. Claude finds the merge commit via `git log --merges --all | grep "<PR-number>"`.
2. Claude runs `git diff <merge-commit>^1 <merge-commit>` and analyzes all changed files.

Priority levels: `要修正` (must fix) · `検討` (consider)

**The report is a quality gate — do not submit the PR if any `要修正` items remain, or if `flutter analyze` / `flutter build` fails.**

**MANDATORY: Before running pre-pr-review, always read the following skill files in full:**
- `.claude/skills/commit-check/SKILL.md` — commit message rules
- `.claude/skills/convention-check/SKILL.md` — Dart code convention rules
- `.claude/skills/pre-pr-review/SKILL.md` — orchestration workflow and report template

**Never generate the report from memory — always re-read the skill files first.**

### Step 2 — Fix 要修正 items, then re-run

After fixing, run `/pre-pr-review` again to confirm no items remain.

### Step 3 — Upload to SharePoint

Upload `.claude/output/PR_<branch>_pre-pr-review_<yyyyMMdd>.md` to SharePoint.
Do not attach it to the PR.