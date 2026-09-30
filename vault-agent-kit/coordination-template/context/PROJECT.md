# Bộ nhớ sản phẩm — Két cá nhân

Baseline 2026-09-30. Đây là intent đã tổng hợp từ cuộc trao đổi, không là trạng thái code.

- User muốn desktop password/API key/env vault, mở bằng master password và copy.
- Stack đề xuất Tauri 2/Rust + React/TS, Windows trước, offline mặc định.
- Codex backend, Antigravity frontend; mỗi bên có independent reviewer.
- UI thân thiện, kem/sage, hình mềm/lỗ khóa; không dashboard/thống kê/chú thích dài. Một platform nhiều accounts; label + email gần nút copy; search nhanh.
- Settings tách riêng, gồm Bảo mật/Giao diện/Sao lưu/Riêng tư/Thông tin. Không security drawer thường trực tại Tất cả.
- Muốn HIBP, clipboard timeout, rate limiting, duplicate warning, secure generator; thêm reauth/backup/lifecycle guards.
- Chấp nhận privacy hide+lock/hotkey; không random rename/move app.
- Recovery/Windows Hello/expiration backlog; không cloud/team/autofill/browser extension v1.
- Ưu tiên code dễ bảo trì, contract thật, evidence thật; không che lỗi/fake API/fake tests.
- Dùng docs/security.md cho threat model và giới hạn; không claim an toàn tuyệt đối.

Nguồn chuẩn dài hạn: MASTER_PROMPT, docs architecture/security/quality/contracts + ADR. Nếu thay intent cần ghi quyết định mới, không tự giả nhớ lại từ chat.
