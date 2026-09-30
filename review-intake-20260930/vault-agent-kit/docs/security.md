# Security baseline và tiêu chí kiểm chứng

Đây là yêu cầu thiết kế, chưa phải kết quả audit implementation. Không có lời hứa “không thể bị hack”.

## Threat model

| Đối tượng/rủi ro | Bảo vệ mục tiêu | Giới hạn |
|---|---|---|
| File vault/backup bị lấy | KDF mạnh + AEAD + không lưu key trần | Master yếu vẫn có thể bị đoán offline |
| File bị sửa/import độc hại | Authentication, input bounds, schema checks | Không tự phát hiện rollback bản hợp lệ cũ |
| Người khác dùng app đang mở | Auto-lock, reauth, scope kiểm tra ở Rust | Máy đã bị admin/malware kiểm soát ngoài bảo đảm |
| Clipboard và shoulder surfing | Copy trực tiếp Rust, timeout, mask | Không thu hồi bản clipboard đã bị lưu bên ngoài |
| Frontend bị lỗi/XSS | CSP, không remote scripts, command allowlist, Rust authz | WebView được phép reveal vẫn có thể thấy dữ liệu đang reveal |
| Crash/disk full/concurrent edits | Transaction, revision CAS, backup nhất quán | Không hứa không mất dữ liệu nếu không có backup tốt |
| Dependency/update bị thay | Pin/check dependencies, release signatures | Không có bằng chứng độc lập thì chưa coi là release trusted |

## S-01 — Cryptographic format

- Có format version, suite ID, random vault ID, KDF parameters/salt, wrapped DEK key slot và record envelope version. Đặc tả byte encoding + test vectors trước freeze.
- Không lưu master verifier bằng fast hash nếu không cần; mở được authenticated key slot là bằng chứng unlock. Password không trim, casefold hoặc normalize âm thầm.
- KDF có floor/ceiling nguồn chính thức và policy được benchmark. Đối chiếu OWASP như floor, không coi thông số web login là cấu hình tối ưu đương nhiên cho vault.
- Nonce strategy phải bảo đảm không lặp cùng key, cả rollback/restore/retry/multi-instance. Dùng CSPRNG theo thư viện với collision/write limits được ghi hoặc construction chuẩn đã review; không counter không bền vững.
- AAD bao gồm cả quan hệ parent nếu nó điều khiển phân nhóm/quyền. Nếu chuyển parent/revision thì re-encrypt authenticated context trong transaction. Không thay AAD nhưng giữ ciphertext cũ.
- Baseline một aggregate revision: re-encrypt cả summary + secret envelope hiện có khi revision/context đổi, nonce mới mỗi envelope, commit atomic. Test metadata-only update → restart → reveal/copy. Key-slot AAD bind vault ID, suite/format, slot purpose, KDF params/salt theo byte spec; giới hạn header được kiểm tra trước KDF, integrity được xác thực sau dẫn xuất key.
- KEK không dùng lại cho fingerprint/password generation; DEK không đem làm log ID. Dùng purpose-separated keys nếu thuật toán cần thêm key, theo construction chuẩn.
- Khi integrity fail: từ chối và giữ nguyên file, không bỏ qua record lỗi rồi overwrite mất dữ liệu.

## S-02 — Dữ liệu ngoài vault

- App data ACL theo user OS; tránh thư mục world-readable. Không cần admin để lưu vault.
- Chỉ ciphertext đi qua SQLite nên journal/temp DB không chứa dữ liệu bí mật nguyên văn. Search cache plaintext chỉ RAM. Audit cả files ngoài SQLite, crash logs và support bundle.
- Không log request body/DTO chứa master, key, secret, recovery code, clipboard, full hash, HIBP prefix, username/search text nhạy cảm. Tránh Debug/Display tự động của secret structs.
- Báo cáo, screenshots, test fixtures và coordination chỉ dùng dữ liệu giả. Secret scanner chỉ một lớp; rà soát nội dung vẫn cần.
- Rust zeroize buffers khi khả thi, hạn chế clone, pin/locking memory chỉ nếu support/được đo; không hứa zeroize toàn bộ JVM/JS/OS/swap/history. JS string không được trình bày là zeroizable.

## S-03 — Authorization, lifecycle và reauth

- Mỗi sensitive command kiểm tra backend state, quyền record/scope, revision, caller window và action policy. Capabilities không thay domain checks.
- Session-bound IPC có expectedSessionEpoch, response có requestId/epoch. Chặn request cũ bị queue qua lock/reunlock ngay tại admission và serialized commit/copy boundary; epoch không phải authorization token. Lock và side effects không dùng check-then-act rời nhau.
- Create chỉ trên absence thực sự, reservation độc quyền trước KDF, no-clobber atomic publish. Corrupt/unreadable không biến thành NoVault. Lock thắng trước publish thì hủy tạo, publish thắng thì kết quả Locked. Create/unlock có lifecycle generation riêng khi chưa có unlocked session.
- Bounded jobs; single unlock in flight; lock cấp quyền không phụ thuộc UI nhận event thành công.
- Reauth grant dùng một lần, gắn action/vault/session/revision, TTL ngắn được đặc tả. UI boolean không có quyền.
- Rate limiting luôn có mức nền; policy thiếu/hỏng phải fallback an toàn có giới hạn, không fail-open. Local persisted counters có thể bị owner sửa/rollback; không coi là chống offline cracking.
- Khi hide/close-to-tray: lock rồi hide; hotkey chỉ hiện Locked. Khi exit: cleanup best effort. Không mặc định chạy ẩn cùng máy khi chưa có lựa chọn người dùng.
- Blur che password; chuyển app để paste không bắt unlock liên tục. OS session lock/sleep mới khóa cả vault theo baseline.

## S-04 — Clipboard

- Return `CopyReceipt`, không password, cho action copy. App không đọc clipboard thường xuyên để theo dõi người dùng.
- Windows adapter dùng generation + clipboard sequence/ownership và đồng bộ OS phù hợp; verify việc check/clear dưới bảo vệ đúng, xử lý clipboard busy bằng retry hữu hạn.
- Test: copy A rồi B; copy A rồi user copy C; user copy lại cùng A; timer A sau B; lock trong copy; quit/disk independent; OS clipboard busy. Không dùng compare-string làm bảo đảm chống ABA.
- Nếu cleanup thất bại, không báo đã xóa. UI “tự xóa sau…” là policy, receipt event mới có thể báo thành công thực tế.

## S-05 — HIBP và outbound

- Chỉ opt-in, fixed HTTPS origin, certificate validation bình thường, không bypass TLS hoặc tự proxy secrets.
- Range 5 hex của SHA-1, header padding, bỏ zero count; SHA-1 là lookup chứ không storage KDF. Không email/raw/full hash/query-per-keystroke.
- Connection metadata/prefix/timing vẫn tồn tại, không hứa hoàn toàn ẩn danh. Persist kết quả chỉ trong envelope, gắn revision và thời gian, không chứa fingerprint plaintext.
- Timeouts, cap bytes, bounded concurrency, retry hữu hạn/transient; honor backoff. API lỗi giữ `error/unknown`, không chuyển `safe`.
- Quét chỉ khi unlocked, cancel/ignore stale; không giữ DEK để quét sau lock.

## S-06 — Backup, restore và key changes

- Snapshot bằng API SQLite phù hợp hoặc DB đã đóng nhất quán; không copy một file đang có pending journal/WAL.
- Backup không giải mã dữ liệu; nếu chứa metadata công khai, giới hạn như vault. Người dùng chọn đích; bản backup mới ghi tạm rồi finalize an toàn.
- Restore không trust path/ZIP filenames/header; giới hạn kích thước/KDF/schema/record count, chống path traversal/symlink theo adapter và permissions. Verify authentication trước thay vault tốt.
- Rewrap master là thay credential truy cập hiện tại, không revoke key đã bị lấy. Rotation DEK là use-case riêng. Giữ rollback-safe copy cho lỗi ghi nhưng bảo vệ theo policy backup.
- Recovery là key slot khác bảo vệ cùng DEK; thêm/xóa cần reauth. Removing slot không xóa các bản backup cũ. Không hardcode recovery secret universal.
- Deletion thông thường không bảo đảm erase SSD/backups. Không tự wipe khi unlock sai.

## S-07 — Tauri và phát hành

- Pin Tauri/Rust/JS dependencies có lockfiles; kiểm tra advisories và bundled SQLite version. Không ghi “latest secure” không bằng chứng.
- Kiểm tra chính sách custom commands riêng; tài liệu Tauri ghi command đã đăng ký có default exposure khác plugin permission. Explicit app command manifest/allowlist và runtime authz đều cần.
- CSP production hạn chế script/resource/network, không remote web content có native access, không wildcard fs/shell/http grants. File picker/backend narrow action thay quyền đọc mọi file cho JS.
- Code signing installer và chữ ký updater là hai lớp khác nhau. Khi phát hành phải cấu hình thực, bảo vệ signing key ngoài repo và kiểm tra update signature. Chưa có signing infra thì đánh dấu dev build, không giả đã ký.
- Không tự tải/chạy executable từ URL người dùng, không hidden persistence/rename/random folder.

## Gate trước secrets thật

Threat model + format được review; encrypted persistence/tamper tests; locked IPC/race tests; clipboard native; backup restore/recovery policy; dependency review; release config; hai independent code reviews đúng SHA. Chưa đủ thì prototype dùng secrets giả. AI review không thay kiểm thử/audit độc lập phù hợp trước phân phối rộng.
