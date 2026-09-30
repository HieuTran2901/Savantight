# Workflow hai implementer và hai reviewer

## 1. Workspace thực sự dùng chung

Một người điều phối bootstrap, hai code worktrees, hai review checkouts, một thư mục COORD_ROOT ngoài worktrees. Không dùng symlink nếu quyền OS/tool chưa hỗ trợ; local pointer JSON đủ để agent truy cập đúng thư mục.

Nếu dùng Git worktree: source cùng history nhưng filesystem tách. Code mới ở worktree backend không tự xuất hiện ở frontend; trao đổi commit, sau đó coordinator tích hợp hoặc frontend cập nhật base có kiểm soát. Không cherry-pick/rebase dirty worktree của người khác.

COORD_ROOT live là nơi trao đổi nhanh, không chứa executable scripts hoặc secrets. Chỉ file Markdown/JSON có schema ngắn. Snapshot được coordinator commit vào docs/collaboration/snapshots/ tại mốc handoff/integration; fresh clone sử dụng snapshot mới nhất nhưng phải đối chiếu commit.

Trường hợp khác máy: thiết lập riêng coordination git repo hoặc shared transport được người dùng cho phép; chỉ sau khi pull/push thật mới có đồng bộ. Nếu chưa có, ghi BLOCKED_SHARED_CONTEXT, vẫn làm phần local độc lập. Không mặc định có API nhắn từ Codex sang Antigravity.

## 2. Single-writer rules

| File/path live | Người được sửa |
|---|---|
| context/PROJECT.md, CURRENT_STATE.md | Coordinator |
| TASKS.md, CONTRACT_STATUS.md | Coordinator |
| status/codex-backend.md | Codex |
| status/antigravity-frontend.md | Antigravity |
| status/reviewer-backend.md | Backend reviewer |
| status/reviewer-frontend.md | Frontend reviewer |
| messages/<role>-<UTC>-<id>.md | Tác giả tạo mới, sau đó bất biến |
| handoffs/<role>-<task>-<revision>.md | Tác giả tạo mới |
| reviews/backend hoặc frontend/<task>-<sha>-rN.md | Reviewer tương ứng tạo mới |
| reports/QUALITY_REPORT.md | Coordinator tổng hợp |

Mỗi role chỉ có một phiên writer đang hoạt động. Trước chuyển tab, phiên cũ dừng viết và checkpoint; phiên mới nhận session ID. Nếu không chắc phiên cũ dừng, read-only và xác minh, không tự giành lease. Timestamps chỉ trợ giúp, không là distributed lock an toàn.

Lưu role status bằng temp-file rồi replace nếu filesystem hỗ trợ; update không nhiều bước làm file rỗng. Message IDs duy nhất chứa role + UTC + suffix; không cùng append một chat.md. File claims/status là coordination protocol, không thay Git/OS locking.

## 3. Task lifecycle

`TODO → READY → IN_PROGRESS → REVIEW_PENDING → CHANGES_REQUESTED / APPROVED → INTEGRATING → DONE`

Nhánh khác: `BLOCKED`, `CANCELLED`. Task DONE chỉ khi review và integration gate thực sự đạt; test PASS không tự chuyển task DONE.

Coordinator tạo assignment có path scope. Agent ghi nhận ở own status và bắt đầu task READY đúng owner. Một task đang IN_PROGRESS không có hai implementer. Đổi scope cần message/ACK owner. Người đang chờ có thể làm task độc lập đã được giao; silence không phải ACK.

Task phải có: ID/title, owner, dependency, scope/non-scope, acceptance, base SHA, contract version/hash, current SHA khi có, tests, reviewers, next action. Template trong templates/TASK.md.

## 4. RFC/contract loop

Một message nêu problem, proposed change, impacted files/DTOs, alternatives, security/performance effect và requested ACK. Bên nhận trả file reply trỏ ID/RFC/version. Coordinator cập nhật accepted status chỉ khi có evidence.

Default không cần họp cho mỗi helper/private function. RFC chỉ khi thay biên contract, invariant, ownership/shared config, crypto/storage format hoặc design decision có consumer khác. Chưa được trả lời thì đánh WAITING_ACK, không spam polling; tiến hành việc không phụ thuộc.

Khi accepted contract hoặc dependency/config baseline đổi: Codex tạo commit riêng chứa chúng và generated artifacts, ghi SHA/hash vào handoff. Antigravity kiểm tra nhánh sạch, đồng bộ exact commit bằng merge/cherry-pick theo topology rồi đối chiếu codegen/lockfile. Không rewrite dirty state, không tự viết bản DTO tương đương. Import approved commit được phép dù file thuộc owner khác; thay đổi nội dung mới vẫn cần owner/RFC.

Chỉ root package/lock/Cargo workspace/Tauri config owner install/update; dependency đề xuất phải ghi license/security/size reason ở task khi có ý nghĩa. Thay một library không tự reset tất cả version.

## 5. Review thật

Candidate là commit sạch. Request gồm base SHA, head SHA, contract hash, behavior, tests/evidence, residual risk. Reviewer kiểm tra checkout có đúng SHA; dirty tree phát hiện phải ghi BLOCKED cho phần không covered.

Reviewer run checks nếu có môi trường, ghi command, stdout summary, OS và exit. Không cần chạy lại mọi test khi không có rủi ro mới; targeted tests đủ nếu gates bắt buộc đã có evidence phù hợp. Không dùng screenshot làm bằng chứng backend encryption.

Finding P0/P1 chặn integration. P2 sửa hoặc explicit risk acceptance của owner đủ thẩm quyền và reviewer đồng ý; security/data-loss escalates người dùng. P3 có thể follow-up có task. Không “approve with known critical”.

Sửa tạo SHA mới → review revision mới; dẫn report cũ và status từng finding. APPROVE không bao phủ commit tương lai. Nếu reviewer phát hiện fix nhỏ, đề xuất patch trong report; implementation agent áp và reviewer xem lại.

## 6. Tích hợp tuần tự

1. Coordinator xác minh candidate review status/contract và không dirty user changes.
2. Merge local từng branch một vào integration checkout; không force-push, không git reset --hard để giải conflict.
3. Xử lý conflict cần xem cả hai ý định; bất kỳ code mới do resolution phải được review bởi bên liên quan.
4. Chạy contract + integration gates trên merged SHA; lưu test evidence. Hai nhánh riêng pass không chứng minh merge pass.
5. Ghi final integration SHA, reports/finding links, task status và memory snapshot. Agent khác cập nhật base khi an toàn.

Remote push, external PR, publishing hoặc signing cần quyền tương ứng đã có từ người dùng; master prompt này không tự cấp quyền bí mật/credentials. Local code và docs trong scope được thực hiện chủ động.

## 7. Bộ nhớ mới nhất và giải quyết mâu thuẫn

Code/commits + contract được ACK + test evidence là nguồn về điều đã tồn tại. PROJECT là intent sản phẩm. CURRENT_STATE chỉ tóm tắt; nếu khác git/evidence, không tự chọn cái thuận tiện: ghi mismatch, xác minh với owner, sửa state bằng bằng chứng.

Không có “trí nhớ vĩnh viễn” từ prompt; agent phải đọc file và dùng đúng nguồn được đồng bộ. Mỗi handoff lưu next action cụ thể, không chỉ “tiếp tục backend”. Đừng nhét cả transcript vào memory; giữ current state ngắn và archive theo mốc.
