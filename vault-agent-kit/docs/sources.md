# Tài liệu chính thức để agents kiểm chứng

Tra cứu ngày 2026-09-30. Đây là nguồn tham khảo, không phải API đã được implement trong repo. Trước chọn version/tool, đọc tài liệu đúng version thực tế và ghi bằng chứng ở task/ADR. Các budgets và workflow trong gói là đề xuất dự án, không phải chuẩn bắt buộc của các nguồn này.

| Nguồn | Dùng để kiểm chứng |
|---|---|
| [Tauri IPC](https://v2.tauri.app/concept/inter-process-communication/) | Commands/events và biên frontend/native |
| [Tauri capabilities](https://v2.tauri.app/security/capabilities/) | Plugin/window permissions và custom app command exposure |
| [Tauri CSP](https://v2.tauri.app/security/csp/) | Production content policy |
| [Tauri updater](https://v2.tauri.app/plugin/updater/) | Chữ ký và cấu hình updates khi đến release |
| [OWASP Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) | KDF floor và phân biệt password validation/encryption |
| [OWASP Cryptographic Storage](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html) | Threat model, authenticated encryption và key storage |
| [OWASP Key Management](https://cheatsheetseries.owasp.org/cheatsheets/Key_Management_Cheat_Sheet.html) | Key lifecycle và backup |
| [HIBP API](https://haveibeenpwned.com/API/v3) | Password range lookup, padding và tránh incremental search |
| [SQLite query planning](https://www.sqlite.org/queryplanner.html) | Index theo query patterns |
| [SQLite EXPLAIN QUERY PLAN](https://sqlite.org/eqp.html) | Quan sát execution plan, không tự suy luận từ tên index |
| [SQLite backup](https://www.sqlite.org/backup.html) | Snapshot DB nhất quán |
| [SQLite WAL](https://www.sqlite.org/wal.html) | Trade-offs, sidecar files, concurrency và release caveats |

SQLite documentation có lưu ý lỗi WAL theo version engine; agent cần kiểm tra bản engine thực sự linked và changelog hiện hành nếu bật WAL. Không dùng claim “crate mới nhất nên engine an toàn” thay kiểm chứng.
