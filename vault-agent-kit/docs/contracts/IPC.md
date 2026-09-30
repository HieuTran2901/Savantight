# IPC v0.1 — PROPOSED, chưa có ACK hoặc implementation

Đây là thiết kế API riêng của dự án. Không phải API sẵn có của Tauri. Các tên sau cần Codex + Antigravity xác nhận, sinh DTO và đăng ký handler trước khi dùng thật. Tất cả hiện `DRAFT / NOT_IMPLEMENTED`.

## Nguồn chuẩn khi code tồn tại

`crates/vault-contracts/` là nguồn DTO Rust, command declaration/registry cùng owner Codex. Công cụ codegen được chọn sau kiểm tra khả năng tương thích, pin phiên bản trong ADR. `packages/contracts/` là output được commit và có drift check; không sửa tay. Runtime schema phải sinh cùng nguồn hoặc kiểm tra tương đương, không hai bộ types tự viết.

Contract version độc lập app version. Gói này là seed `0.1-draft`, không được ghi ACCEPTED cho đến có ACK thật của hai bên.

## Command inventory ban đầu

| Command dự kiến | Đầu vào chính | Kết quả | Trạng thái cho phép |
|---|---|---|---|
| `vault_status` | Không | `VaultStatus` tối thiểu, không record metadata | Mọi state |
| `vault_create` | masterPassword, initial policy, expectedLifecycleEpoch | vault ID/status; không secret | NoVault; reservation độc quyền |
| `vault_unlock` | masterPassword, expectedLifecycleEpoch | session epoch/status | Locked, qua throttle |
| `vault_lock` | reason enum | resulting state + epoch | Idempotent; NoVault giữ NoVault |
| `platform_list` | cursor, limit | summary + account counts | Unlocked |
| `platform_create` | name, URL tùy chọn | summary + revision | Unlocked |
| `credential_list` | platformId, cursor, limit | metadata page | Unlocked |
| `credential_create` | platformId, typed payload | metadata + revision | Unlocked |
| `credential_get` | credentialId | detail không secret, revision | Unlocked |
| `credential_update` | ID, expectedRevision, explicit patch | metadata + newRevision | Unlocked |
| `credential_delete` | ID, expectedRevision | deletion receipt | Unlocked + policy |
| `credential_reveal` | ID, field ID, revision, grant khi cần | ephemeral revealed value | Unlocked + policy |
| `credential_copy` | ID, field ID, revision, grant khi cần | CopyReceipt, không value | Unlocked + policy |
| `vault_search` | metadata query, cursor, limit | account/platform summary hits | Unlocked |
| `settings_get` | section enum | typed settings snapshot | Theo public/locked/unlocked policy |
| `settings_update` | expectedRevision, typed patch | persisted snapshot | Unlocked; reauth với protected fields |

Reauth, generator, duplicates, HIBP, env, export/backup/restore/recovery bổ sung theo slice. Không tạo một `execute_action(any)`/`query_sql`/`read_file(path)` toàn quyền để né inventory. `platform_create` và account update phải xác định transaction semantics trước xây UI.

## DTO semantics cần freeze

- Envelope chung: requestId, contractVersion; command chỉ được phép khi Unlocked phải kèm expectedSessionEpoch. Response có requestId + actual epoch để frontend loại stale replies. Rust reject epoch mismatch tại admission và boundary đã tuyến tính hóa với lock. Epoch không thay authentication. Create/unlock dùng expectedLifecycleEpoch trong giai đoạn chưa có unlocked session.
- `VaultStatus`: state enum, contractVersion, lifecycleEpoch, sessionEpoch khi có, retryAfterMs tùy trường hợp; không đường dẫn vault/name/email khi locked. Epoch có phạm vi process instance để không trùng sau restart.
- `Page<T>`: items, nextCursor nullable; limit default50/max200. Cursor opaque phải validate, không chứa secret.
- `CredentialSummary`: ID, platformId, display label, username optional, kind, revision và cảnh báo cho revision hiện tại; không `password` kể cả masked string.
- `CredentialPatch`: từng field `unchanged/set/clear` theo spec; secret set chỉ trên đường explicit save. Unchanged default; không clear từ missing/empty text vô ý.
- `CopyReceipt`: operationId, credentialId, sessionEpoch, scheduledClearAfterMs; không hứa clipboard đã được xóa khi mới scheduled.
- `RevealedValue`: value, record/revision/epoch, expiry cần thiết; không được persist/cache. Quy định UI timeout không đồng nghĩa value bị thu hồi khỏi OS memory.
- `AppError`: stable code, messageKey, safe details và optional retryAfterMs; không stringify nguồn lỗi nhạy cảm.

`vault_create` phải reserve độc quyền, state Creating, publish atomic no-clobber; file corrupt/unreadable không được coi là absent. `vault_lock` NoVault giữ NoVault; Creating bị hủy nếu lock thắng publication, hoặc kết quả Locked nếu publish đã hoàn tất. Với vault hiện hữu, chỉ trả Locked sau barrier; không báo hoàn tất cleanup nếu nó chưa xong. Các state và lifecycleEpoch phải được sinh cùng contract.

Error codes đề xuất: LOCKED, INVALID_INPUT, INVALID_CREDENTIALS, RATE_LIMITED, NOT_FOUND, CONFLICT, INTEGRITY_ERROR, IO_ERROR, CLIPBOARD_BUSY, PERMISSION_DENIED, REAUTH_REQUIRED, STALE_SESSION, CANCELLED, UNSUPPORTED_VERSION, NOT_IMPLEMENTED. UI không suy diễn từ text English.

## Events dự kiến

`vault_state_changed`, `record_changed`, `clipboard_clear_result`, `settings_changed` chỉ state/ID/revision/generation. Không password, username hay query. Frontend phải cleanup listeners. Event bị mất không cấp thêm quyền: command vẫn kiểm tra session; có cơ chế refresh status khi focus/reconnect.

## Registry implementation cần tạo

Mỗi command: name, input/output schema IDs, introducedVersion, requiredState, callerWindow, reauthPolicy, inputBounds, errors, emits, implementationPath, source SHA, tests, status. Không đánh VERIFIED khi chỉ typecheck.

Custom commands phải được đánh giá quyền dựa trên Tauri docs hiện hành. Có capabilities plugin không chứng minh custom invoke đã bị giới hạn; app command manifest + runtime check đều cần.

## Quy trình thay đổi

RFC → BE proposal → FE ACK/feedback → sensitive review khi cần → exact version/hash ACCEPTED → generated bindings → BE/FE implementation → integration evidence → VERIFIED. ACK có tác giả, timestamp và target hash. Silence/timeouts không là đồng ý.

Codex bàn giao commit baseline chứa contract, generated output và dependency/config liên quan. Frontend nhập chính commit đó vào worktree sạch và xác minh hash trước consumer implementation. Shared coordination chỉ chứa pointer/evidence, không tự đồng bộ source; việc import commit được duyệt không thay quyền sở hữu file.

Breaking change sau implementation phải ghi migration/consumer impact, không âm thầm rename command. Nếu external API chưa xác minh, ghi UNVERIFIED, không dựa vào tên nhớ được.
