# Release Note — v1.2.0

**Ngày phát hành:** 2026-06-09

## Tổng quan

Sửa lỗi git command trong `pre-pr-review` và viết lại Rule 5 của `convention-check` để phù hợp với cấu trúc thực của Flutter project.

---

## Thay đổi

### `/pre-pr-review` — Sửa git diff command và chuẩn hóa

- **Fix:** `git diff $(git merge-base HEAD <branch>)...HEAD` → `git diff $(git merge-base HEAD <branch>)` — bỏ `...HEAD` thừa, nhất quán với `convention-check`
- Cập nhật numbering rules: "Rules 2–6" → "Rules 1–5"
- Category labels phần Vietnamese đổi sang tiếng Việt: `Đặt tên` `Viết tắt` `Định danh` `Tài liệu`
- Bỏ "test convention violation" khỏi ví dụ 要修正 (không có rule tương ứng)

### `/convention-check` — Viết lại Rule 5 (public API documentation)

- **Scope table** theo thư mục thực của project: shared zone (`lib/services/`, `lib/utils/`, `lib/providers/`, v.v.) áp dụng full rule; `lib/screens/` áp dụng nhẹ hơn
- **Logic 3 bước** thay thế danh sách phẳng: Step 1 skip always → Step 2 skip-if-obvious → Step 3 flag
- **Heuristic "obvious"**: body ≤ 3 dòng, không `async`, không dependency call, không `try/catch` → bỏ qua không flag
- **Rule Riverpod provider**: top-level provider trong `lib/providers/` = 要修正 nếu thiếu `///`; provider trong `lib/screens/` = skip
- Tách "doc comment restating name" từ 要修正 sang 検討 (đây là doc kém, không phải undocumented)
- Thêm ví dụ cụ thể: obvious methods, bad/good `fetchUser`, Riverpod provider
- Cập nhật Output format examples theo thư mục thực (`lib/services/`, `lib/screens/`)

---

# Release Note — v1.1.0

**Ngày phát hành:** 2026-06-03

## Tổng quan

Cập nhật `/pre-pr-review` để đồng bộ format báo cáo với bên BE.

---

## Thay đổi

### `/pre-pr-review` — Đồng bộ format output với BE

- Báo cáo xuất ra **hai phần: Japanese trước, Vietnamese sau** (thay vì một phần tiếng Anh)
- Header mới: `# コードレビュー結果` / `# Kết quả review code`
- Bảng tổng hợp (`要修正` / `検討`) hiển thị ngay đầu mỗi phần
- Category labels đồng nhất với BE:
  - JP: `命名` `コメント` `略語` `識別子` `コミット` `テスト` `ドキュメント`
  - VN: `Naming` `Comments` `Abbreviation` `Identifier` `Commit` `Test` `Documentation`
- Thêm rule **no AI attribution** — report không được đề cập AI/Claude/LLM
- Chỉ flag các dòng được thêm hoặc sửa (`+` lines), không review code bị xóa
- Hiển thị danh sách commit cho user trước khi bắt đầu review
- Giữ nguyên Flutter verification commands (không thay bằng npm như BE)

---

# Release Note — v1.0.0

**Ngày phát hành:** 2026-05-27

## Tổng quan

Phát hành bộ Claude Code Skills tích hợp quy trình kiểm tra chất lượng code và commit message cho dự án Flutter/Dart.

---

## Tính năng mới

### `/commit-check`
Kiểm tra commit message theo chuẩn **Conventional Commits**. Tự động lấy lịch sử git, không yêu cầu người dùng paste thủ công.

### `/convention-check`
Audit code Dart/Flutter theo coding convention của team (naming, dead code, meaningful identifiers, abbreviations, public API docs). Tự động lấy diff từ git.

### `/pre-pr-review`
Unified pre-PR quality gate — orchestrate cả `commit-check` + `convention-check` + Flutter verification, xuất báo cáo Markdown. Chạy một lệnh duy nhất trước khi tạo PR.

---

## Thay đổi

- Chuyển đổi nội dung skill từ tiếng Việt sang tiếng Anh (`convention-check`, `pre-pr-review`) để phù hợp với team quốc tế.

---

## Cách sử dụng

```
/commit-check       — Kiểm tra commit message
/convention-check   — Kiểm tra convention code
/pre-pr-review      — Chạy toàn bộ quy trình trước PR
```