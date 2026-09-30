# Bắt đầu với bộ điều phối Két cá nhân

Bộ này là prompt và tài liệu khởi tạo; chưa chứa implementation app, chưa có benchmark/native test của app. Hai báo cáo trong `planning-reviews/` chỉ review chất lượng đặc tả.

## Cách dùng

1. Giải nén vào một thư mục mới. Nếu dự án đã có code, để bộ này cạnh repo và yêu cầu Codex nhập tài liệu có kiểm tra xung đột; đừng ghi đè `AGENTS.md`/config hiện có.
2. Mở Codex ở thư mục dự án; gửi `MASTER_PROMPT.md` và `prompts/CODEX_BACKEND.md` (hoặc dán toàn bộ hai file). Codex làm bootstrap workspace trước.
3. Sau khi Codex báo COORD_ROOT và worktree frontend đã sẵn sàng, mở Antigravity tại worktree frontend; gửi cùng `MASTER_PROMPT.md` và `prompts/ANTIGRAVITY_FRONTEND.md`.
4. Dùng hai phiên/subagent reviewer riêng với `prompts/REVIEWER_BACKEND.md` và `prompts/REVIEWER_FRONTEND.md`. Không lấy implementer tự đóng vai reviewer rồi ghi independent approval.
5. Khi đổi tab, gửi `prompts/RESUME.md` và tên role. Agent đọc bộ nhớ chung, xác minh git state rồi tiếp tục.

Nếu hai công cụ chạy trên cùng máy, dùng một COORD_ROOT vật lý. Nếu khác máy, cần thiết lập đường truyền/file sync hoặc git coordination repo thực sự; không có đồng bộ tự động chỉ từ prompt này.

## Bố trí workspace sau bootstrap

```text
workspace/
  vault-main/             # repo tích hợp; chỉ coordinator ghi
  vault-be/               # worktree Codex
  vault-fe/               # worktree Antigravity
  vault-review-be/        # review commit backend
  vault-review-fe/        # review commit frontend
  vault-coordination/     # một thư mục live chung, ngoài worktrees
```

Trong mỗi worktree có `.agent-local.json` gitignored, ví dụ trên Windows:

```json
{
  "role": "codex-backend",
  "repo_root": "D:/Projects/VaultWorkspace/vault-be",
  "coord_root": "D:/Projects/VaultWorkspace/vault-coordination"
}
```

Đây là ví dụ đường dẫn, không phải thư mục đã được tạo. Không dùng đường dẫn sandbox trong gói này làm đường dẫn Windows. Codex xác định đường dẫn thật, không tự ghi đè thư mục đang có.

## Trật tự bootstrap

Codex inspect → setup repo và dependency manifest chung → seed `coordination-template/` ra COORD_ROOT → tạo worktree/role configs → mở task frontend → thống nhất IPC → implementation song song → review riêng → tích hợp → kiểm tra → checkpoint.

Có thể tạo task design frontend trước contract; task kết nối backend phải chờ contract được ACK. Tên command trong draft chưa phải API đã chạy.

Sau ACK, Codex tạo commit baseline riêng cho contract/generated bindings/shared config và ghi SHA trong handoff. Antigravity đồng bộ đúng commit vào worktree sạch của mình, xác minh contract hash/lockfiles rồi mới kết nối API. File điều phối không tự đưa source code sang worktree khác.

Checkpoint đầu là shell + Settings navigation + vault_status IPC thật; review scaffold trước encrypted vertical slice. Không có secret demo lưu nguyên văn trong checkpoint đầu.

## Điểm cần giữ nguyên khi giao việc

- Rust backend trong Tauri, không thêm web backend không cần thiết.
- Contract trước implementation; một owner cho mỗi file/task.
- Thư mục live chung không phải bản copy `docs/` trong các worktree.
- Reviewer độc lập; báo cáo gắn đúng commit.
- Bảo mật nằm trong Settings, cảnh báo tại tài khoản; không dashboard.
- Chỉ dùng secrets giả đến khi vượt các gate bảo mật và nền tảng.

`MASTER_PROMPT.md` là bản tổng hợp đủ để đưa cho agent. Các file `docs/` là đặc tả chi tiết; `coordination-template/` là bộ nhớ ban đầu, chưa có sự đồng ý/implementation của hai agent bên ngoài.
