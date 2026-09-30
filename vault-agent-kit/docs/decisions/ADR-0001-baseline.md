# ADR-0001 — Desktop offline, typed IPC và encrypted payloads

Date: 2026-09-30. Status: PROPOSED baseline; cần xác nhận kỹ thuật khi bootstrap, không phải chữ ký của Codex/Antigravity.

## Bối cảnh

Một cá nhân quản lý nhiều tài khoản theo nền tảng, secrets dự án, dùng nhanh bằng copy. Windows-first, không cloud sync trong v1. Cần review độc lập và phối hợp hai công cụ coding.

## Quyết định đề xuất

Tauri 2/Rust + React/strict TS; IPC typed; domain tách framework; SQLite encrypted summary/secret payloads; metadata search Rust RAM; một writer trước, rollback journal thay vì bật WAL mặc định. Argon2id/KEK/DEK/AEAD qua libraries đã kiểm chứng, không tự viết primitive. Shared coordination directory ngoài worktrees, snapshot về repo.

## Lý do

Giảm số biên mạng và thành phần vận hành, phân chia công việc rõ, kiểm tra authorization ở Rust và không lưu index nhạy cảm plaintext. Một writer phù hợp workload cá nhân và đơn giản hóa transaction/race trước khi có bằng chứng cần pool.

## Trade-offs

Structural metadata của SQLite vẫn lộ; search phải warm khi unlock, có plaintext metadata trong RAM. Crypto format/nonce/key lifecycle cần review cẩn thận, không tự đạt an toàn chỉ từ tên thuật toán. Windows native integration cần test Windows. Không sync tự động giữa công cụ ở hai máy.

## Quyết định tiếp theo

Exact dependencies/codegen, crypto envelope encoding/nonce limits, master password policy/KDF bounds, backup format, UI state semantics và command manifest phải có evidence/RFC. Chưa chọn version từ trí nhớ. Thay lựa chọn storage/crypto cần ADR mới, không âm thầm thay dưới lớp adapter.
