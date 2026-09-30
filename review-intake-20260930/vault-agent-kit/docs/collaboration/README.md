# Snapshot điều phối trong Git

Thư mục này không phải COORD_ROOT live. Coordinator lưu snapshot đã loại bỏ dữ liệu nhạy cảm vào `snapshots/<milestone>-<UTC>/` sau mỗi integration/handoff quan trọng.

Snapshot gồm PROJECT, CURRENT_STATE, TASKS, CONTRACT_STATUS, own-role statuses, handoff/review/report cần để phục hồi và một INDEX nêu integration SHA + nguồn live path loại thông tin cá nhân nếu cần.

Không commit secrets, private paths nhạy cảm, raw clipboard, test user thật hoặc toàn bộ logs không kiểm duyệt. Không copy snapshot lên live nếu live mới hơn. Clone mới thiếu shared directory phải báo đang dùng snapshot, đối chiếu git và tạo shared context dưới quyền coordinator.

Gói khởi tạo chưa có snapshot implementation vì chưa có source/commit của app.
