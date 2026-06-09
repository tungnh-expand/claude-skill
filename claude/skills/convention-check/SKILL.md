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

**Scope**

| Zone | Directories | Treatment |
|---|---|---|
| Shared | `lib/services/`, `lib/utils/`, `lib/extensions/`, `lib/widgets/`, `lib/core/`, `lib/foundation/`, `lib/providers/`, `lib/navigator/`, `lib/config/`, `lib/constants/`, `lib/event_bus/`, `lib/ttp/` | Full rule applies |
| Feature | `lib/screens/` | Lighter touch — only flag if genuinely non-obvious |

**Format:** Use `///` (not `//`, not `/** */`). Reference symbols with `[SymbolName]`.

**Step 1 — Always skip (never flag):**
- Private members (leading `_`)
- Test files (`test/**`, `*_test.dart`, `integration_test/**`)
- Widget state classes (`_FooState`) and `*State` plain data holders (e.g., `LoginState`, `SettingState`)
- Trivial overrides: `build`, `dispose`, `toString`, `==`, `hashCode`, `createState`
- Generated files (`*.g.dart`, `*.freezed.dart`, etc.)

**Step 2 — Skip if method is obviously self-explanatory (all must be true):**
- Body is ≤ 3 lines, no `async`/`await`
- No dependency calls (`_repository`, `_service`, `_dio`, `_storage`, `http`, `SharedPreferences`, etc.)
- No `try/catch`, no `throw`
- Body is a simple field access or single delegation — regardless of return type

```dart
void clear() => _cache.clear();      // obvious — skip
bool get isEmpty => _items.isEmpty;  // obvious — skip
User get currentUser => _user;       // obvious — skip
```

If **any** signal is missing → not obvious → proceed to Step 3.

**Step 3 — Flag:**

| Severity | When |
|---|---|
| **要修正** | Public class / mixin / extension / enum / typedef in shared zone with no `///` |
| **要修正** | Non-obvious public method in shared zone with no `///` |
| **要修正** | Top-level Riverpod provider in `lib/providers/` with no `///` (providers inside `lib/screens/` → skip) |
| **検討** | Non-obvious public method in `lib/screens/` with no `///` |
| **検討** | Doc comment that just restates the name (`/// Gets the user.` on `getUser()`) |

**Example:**

```dart
// 検討 — restates the name, no useful info
/// Fetches the user.
Future<User> fetchUser(String id) async { /* ... */ }

// Good — behavior, edge cases, throws
/// Fetches a [User] by [id] from the remote API.
/// Throws [NotFoundException] if the user does not exist.
/// Throws [NetworkException] on connectivity failure.
Future<User> fetchUser(String id) async { /* ... */ }

// Good — Riverpod provider in lib/providers/
/// Provides the current authentication state.
/// Automatically invalidated when the session token changes.
final authStateProvider = StateNotifierProvider<AuthController, AuthState>(
  (ref) => AuthController(ref),
);
```

---

## Severity

Two levels — consistent with `pre-pr-review`:

| Level | Examples |
|---|---|
| **要修正 (Must fix)** | Wrong casing, PascalCase file name, dead commented-out code, `HTTPClient`-style acronym, undocumented public class/service/utility in `lib/services/` or `lib/utils/` |
| **検討 (Consider)** | Borderline abbreviations (`auth`, `config`, `ctx`), names that could be improved, missing doc on a screen-specific non-obvious method, doc comment that restates the name |

When in doubt, prefer **検討** over **要修正**.

---

## Output format

```markdown
# Convention audit

**Scope:** <e.g. "5 Dart files (2 generated files skipped)">
**Base branch:** <branch>
**Result:** N 要修正 · N 検討

## 要修正 (Must fix)

- **`lib/services/AuthService.dart`** — Rule 1 (file naming). Rename to `auth_service.dart`.
- **`lib/services/auth_service.dart:12`** — Rule 1 (class naming). `class authService` → `class AuthService`.

## 検討 (Consider)

- **`lib/screens/login/login_controller.dart:8`** — Rule 3 (meaningful names). `final data = await fetchProfile()` → `userProfile`.

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