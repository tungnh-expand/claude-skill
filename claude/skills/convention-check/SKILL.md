---
name: convention-check
description: Audits Dart/Flutter code changes against the team's coding conventions (naming, dead code, meaningful identifiers, abbreviations, public API docs) and returns a prioritized report. Trigger when the user asks to review, check, audit, or validate code changes — "check this PR", "review my code", "convention check", "lint this change". Auto-fetches the git diff; the user does not need to paste anything.
---

# Convention Check (Flutter/Dart — Code)

Audits code changes against the team's five Dart coding conventions and produces a prioritized, actionable report.

> For commit message checks, use `/commit-check` instead.

## How to get the diff

**Do not ask the user to paste a diff, and do not ask them to select files.** Retrieve and analyze everything automatically:

### Step 1 — Identify the base branch

- If the user provided a base branch or PR number, use it.
- Otherwise ask: *"What is the base branch? (e.g., main, develop)"*

### Step 2 — Get the full diff

```bash
# List changed files (for display only)
git diff $(git merge-base HEAD <base-branch>) --name-only

# Full diff for analysis
git diff $(git merge-base HEAD <base-branch>)
```

Analyze **all** changed files. Skip generated files automatically (see below) — do not ask the user to choose.

Only flag lines the diff **adds or modifies** (lines starting with `+`). Context lines and removed lines are out of scope.

## Files to ignore (auto-generated)

- `*.g.dart`, `*.freezed.dart`, `*.mocks.dart`, `*.gen.dart`, `*.config.dart`
- Anything under `build/`, `.dart_tool/`, `ios/Pods/`, `android/.gradle/`, `.flutter-plugins*`
- `pubspec.lock`

If the diff is dominated by generated files, mention this and focus on the hand-written changes.

---

## The five rules

Evaluate all five rules. Collect **all** violations before writing the report.

---

### Rule 1 — Naming conventions (camelCase / PascalCase / snake_case)

| Entity | Style | Examples |
|---|---|---|
| Variables, functions, parameters, methods | `lowerCamelCase` | `userCount`, `fetchProfile()`, `defaultTimeout` |
| Classes, enums, typedefs, extensions, mixins | `UpperCamelCase` | `UserService`, `OrderStatus`, `Disposable` |
| Files, directories, library names | `snake_case` | `user_service.dart`, `login_screen.dart` |
| Private identifiers | leading `_` | `_internalCache`, `_LoginFormState` |
| DB / persistence table names | `snake_case` | `user_accounts`, `order_items` |
| Import prefixes | `snake_case` | `import 'foo.dart' as my_prefix;` |

Key checks:
- **State classes:** `_MyWidgetState` for `MyWidget extends StatefulWidget` — flag missing leading underscore.
- **Constants:** `lowerCamelCase` (not `SCREAMING_SNAKE_CASE`).
- **Acronyms:** `HttpClient`, `parseJsonResponse`, `userId` — flag `HTTPClient`, `parseJSONResponse`, `userID`.
- **File names:** a file named `LoginScreen.dart` is wrong even if the class inside is correct → `login_screen.dart`.
- **Persistence layer:** SQLite/Drift/Isar/Hive table/collection names must be `snake_case`.

---

### Rule 2 — Remove unnecessary commented-out code

**Flag:** commented-out executable code that serves no purpose.
- `// final oldResult = legacyCompute(x);`
- Large `/* … widget tree … */` blocks left after a refactor.

**Don't flag:**
- `// TODO(name): explanation`
- `// reason why this code exists`
- `///` dartdoc comments
- License headers
- `// ignore: <lint_rule>` directives

Heuristic: removing the comment markers produces syntactically valid Dart of the same shape → it's dead code.

---

### Rule 3 — Use meaningful variable and function names

Flag:
- **Single-letter names outside tight scope** — loop `i`, short callback `e`, coordinates `x/y` are fine. `final d = await getUser()` at function scope is not.
- **Generic placeholders:** `data`, `info`, `tmp`, `temp`, `foo`, `bar`, `obj`, `val`, `result` — when a more specific name is obvious from context.
- **Type-in-name:** `userList` → `users`; `userMap` → `usersById`; `stringName` → the actual concept.
- **Generic widget names:** `Widget1`, `MyButton`, `CustomContainer` → `PrimaryButton`, `CartItemTile`.
- **Noise callbacks:** `onTap1`, `handler2` → name by what they do (`onLoginPressed`, `handleRefresh`).

Always propose a specific rename, not just "rename this."

---

### Rule 4 — No abbreviations (except widely-known ones: ID, URL, etc.)

**Allowed — do not flag:**
```
ID, URL, URI, HTTP, HTTPS, API, UI, UX, DB, SQL, HTML, CSS, JS, TS,
JSON, XML, YAML, CSV, PDF, CPU, GPU, RAM, IO, OS, UUID, ISO, UTC, TZ,
TLS, SSL, DNS, IP, TCP, UDP, REST, gRPC, CLI, SDK, CDN, JWT, OAuth,
MD5, SHA, CORS, env, iOS, MVVM, MVC, DI, IoC, ORM, BLoC, SVG,
PNG, JPG, GIF, RGB, RGBA, DP, SP, PX, FPS, APK, AAB, IPA, ADB, FCM,
APNs, ref (Riverpod Ref/WidgetRef only)
```

**Flag everything else:**

| Abbreviation | Fix |
|---|---|
| `usr` | `user` |
| `btn` | `button` |
| `msg` | `message` |
| `ctrl` | `controller` |
| `txt` / `txtCtrl` | `text` / `textController` |
| `nav` | `navigator` |
| `cfg` | `config` |
| `calc` | `calculate` |
| `pkg` | `package` |
| `pwd` / `pw` | `password` |
| `dur` | `duration` |
| `pos` | `position` |
| `idx` | `index` |
| `len` | `length` |
| `arr` | rename to what the list contains |
| `vm` | `viewModel` |
| `repo` | `repository` (unless referring to a Git repo) |

**Borderline → use 検討 (Consider):**
- `ctx` — very common for `BuildContext` but Effective Dart recommends `context`
- `auth`, `config` — real words in common use
- `bloc` — fine as a class suffix (`LoginBloc`); flag as a standalone variable

---

### Rule 5 — Document public APIs with clear comments

> **Scope: app code vs library code**
> This rule applies primarily to **shared/reusable code**: services, repositories, utilities, custom widgets used across features, and any code in `lib/core/` or `lib/shared/`. For feature-specific screens and widgets that are not reused, apply this rule with a lighter touch — only flag if the API is genuinely non-obvious.

Every qualifying **public** declaration must have a `///` dartdoc comment covering: what it does, parameters, return value, side effects/exceptions.

Use `///` (not `//` and not `/** */`). Reference other symbols with `[SymbolName]`.

**Flag undocumented:**
- Public classes, mixins, extensions, enums, typedefs in `core/`, `shared/`, or utility files.
- Public methods with non-obvious behavior.
- Public top-level functions, getters, constants with non-obvious purpose.
- Doc comments that just restate the name — `/// Gets the user.` on `getUser()` adds nothing.

**Don't flag:**
- Private members (leading `_`).
- Test files (`test/**`, `*_test.dart`, `integration_test/**`).
- Widget state classes (`_FooState`).
- Trivial overrides: `build`, `dispose`, `toString`, `==`, `hashCode`, `createState`.
- Feature-specific screens/widgets that are self-explanatory.
- Generated files.

---

## Severity

Two levels — consistent with `pre-pr-review`:

| Level | Examples |
|---|---|
| **要修正 (Must fix)** | Wrong casing, PascalCase file name, dead commented-out code, `HTTPClient`-style acronym, undocumented public utility |
| **検討 (Consider)** | Borderline abbreviations (`auth`, `config`, `ctx`), names that could be improved, missing doc on a feature widget |

When in doubt, prefer **検討** over **要修正**.

---

## Output format

```markdown
# Convention audit

**Scope:** <e.g. "5 Dart files (2 generated files skipped)">
**Base branch:** <branch>
**Result:** N 要修正 · N 検討

## 要修正 (Must fix)

- **`lib/features/auth/UserService.dart`** — Rule 1 (file naming). Rename to `user_service.dart`.
- **`lib/features/auth/user_service.dart:12`** — Rule 1 (class naming). `class userService` → `class UserService`.

## 検討 (Consider)

- **`lib/features/auth/user_service.dart:8`** — Rule 3 (meaningful names). `final data = await fetchProfile()` → `userProfile`.

## ✓ Clean rules

<Rules with no violations.>
```

If there are zero violations, say so plainly.

## What to avoid

- **Don't flag unchanged context lines** — only `+` lines in the diff.
- **Don't add rules the team didn't ask for** — no `dart format`, no `prefer_const_constructors`, no import-order preferences.
- **Don't review generated files.**
- **Don't be dogmatic** — use 検討 for `auth`, `config`, `ctx`, `ref`.
- **Don't restate the full rule text** — a short reference like "Rule 4 (abbreviations)" is enough.
- **Don't reformat code** — show short snippets to locate the issue only.