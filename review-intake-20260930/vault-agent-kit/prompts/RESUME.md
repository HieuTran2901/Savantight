# Tiếp tục ở tab/phiên mới

Vai trò của bạn: <codex-backend | antigravity-frontend | reviewer-backend | reviewer-frontend>.

Đọc AGENTS.md và .agent-local.json để tìm COORD_ROOT thực tế. Nếu file chưa có, đọc START_HERE rồi xác định đúng workspace; không tự đoán một path khác. Đọc theo thứ tự:

1. COORD_ROOT/context/PROJECT.md và CURRENT_STATE.md.
2. COORD_ROOT/TASKS.md, status/<role>.md và handoff gần nhất của role.
3. Contract/version + ADR liên quan trong repo; các RFC đang chờ và review chưa đóng.
4. Git status, branch, HEAD/base SHA; đối chiếu task và checkpoint.

Nếu live directory không có nhưng có committed snapshot, ghi rõ snapshot SHA và trạng thái chưa đồng bộ, phục hồi dưới quyền coordinator. Không lấy snapshot cũ làm bằng chứng một task đã được review. Không đè dirty changes.

Trả lời ngắn: role, task đang tiếp tục, điều đã xác minh, blocker và một bước kế tiếp. Sau đó làm việc ngay. Không viết lại kế hoạch toàn bộ; không khởi tạo repo mới chỉ vì tab không có chat cũ. Cuối phiên ghi handoff và cập nhật status role, không viết trạng thái của agent khác.
