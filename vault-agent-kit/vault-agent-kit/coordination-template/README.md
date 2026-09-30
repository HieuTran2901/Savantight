# Seed của thư mục live dùng chung

Coordinator copy thư mục này ra một vị trí ngoài worktrees, ví dụ `D:/Projects/VaultWorkspace/vault-coordination`, rồi ghi absolute path vào .agent-local.json từng role. Đừng ghi live updates vào các bản template riêng trong từng worktree.

Folder live cần thêm khi có file thật: messages/, handoffs/, reviews/backend/, reviews/frontend/, evidence/. Reports và state tổng hợp chỉ coordinator ghi. Role status chỉ owner ghi. Chi tiết protocol tại docs/workflow.md trong repo.

Đây là seed chưa có app code, chưa ACK contract và chưa có review implementation. Không đưa dữ liệu người dùng thật vào bất cứ file nào ở đây.
