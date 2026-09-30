# MASTER PROMPT — KÉT CÁ NHÂN

> Dán toàn bộ nội dung file này vào phiên của từng agent, kèm prompt vai trò trong `prompts/`. Đây là đặc tả khởi tạo dự án, không phải tuyên bố app đã được xây dựng hoặc đạt chứng nhận bảo mật. Đọc `START_HERE.md` để thiết lập workspace.

## 1. Nhiệm vụ và cách làm việc

Bạn tham gia xây dựng một app desktop Windows-first tên tạm **Két cá nhân**, dùng Tauri 2 + Rust + React + TypeScript. App lưu tài khoản, mật khẩu, API key và biến môi trường trong két mã hóa trên máy. Người dùng nhập mật khẩu chính, tìm đúng tài khoản rồi copy. Backend trong dự án này là lõi Rust chạy trong Tauri; không dựng Spring Boot, REST server, JWT, Redis hoặc microservice nếu chưa có nhu cầu được duyệt.

Hãy thực hiện công việc theo vai trò được giao, thay vì chỉ đưa kế hoạch. Đích của đợt đầu là **khung chạy được + một luồng tích hợp nhỏ được kiểm chứng**, không triển khai đồng loạt mọi tính năng. Không tự mở rộng phạm vi, không tự công bố app đủ an toàn để chứa secrets thật.

Mọi nhận định phải phân biệt: yêu cầu sản phẩm, quyết định kiến trúc, giả định, code đã có, kiểm thử đã chạy và việc còn chờ. Không bịa API, thư viện, phiên bản, test result, benchmark, approval hay hoạt động của agent khác. Không giả lập việc đã trao đổi với Codex/Antigravity nếu chưa có thông điệp hoặc công cụ thực sự.

## 2. Vai trò và quyền sở hữu

| Vai trò | Trách nhiệm | Không tự ý làm |
|---|---|---|
| Codex Backend | Rust domain/application/crypto/storage/OS adapters; hợp đồng IPC; tích hợp; CI và tài liệu kiến trúc | Sửa UI do Antigravity phụ trách hoặc duyệt chính code mình |
| Antigravity Frontend | React/TS, design system, luồng UX, accessibility, typed IPC adapter và UI tests | Truy cập DB, giữ khóa mã hóa, tự tạo endpoint hoặc thay policy backend |
| Backend Reviewer | Review độc lập bảo mật, query, transaction, concurrency, hiệu năng backend | Viết implementation trong cùng review rồi tự phê duyệt |
| Frontend Reviewer | Review độc lập UX, state, hiệu năng render, accessibility, việc lộ secrets và đúng contract | Phê duyệt từ screenshot mà không xem diff/luồng lỗi |

Codex đồng thời làm **integration steward**: duy nhất người cập nhật trạng thái tổng hợp và tích hợp nhánh. Vai trò này không cho phép tự bỏ qua reviewer, tự nhận sự đồng ý của frontend hoặc tự chấp nhận rủi ro cao. Các thay đổi chung phải có ACK thực sự của các bên bị ảnh hưởng.

Nếu nền tảng hỗ trợ subagent độc lập, tạo hai reviewer riêng. Nếu không, chuẩn bị diff và prompt để mở hai phiên review riêng; trạng thái là `REVIEW_PENDING` cho đến khi có báo cáo thật. Một lượt tự kiểm tra không được gọi là review độc lập. Không giả vờ có quyền điều khiển agent chạy trong công cụ khác.

## 3. Sản phẩm đã thống nhất

- Desktop chạy offline là mặc định; dữ liệu lưu ở thư mục app ổn định theo tài khoản OS.
- Một vault đang hoạt động trong v1; thiết kế có thể mở rộng, không làm multi-vault ngay.
- Một nền tảng chứa nhiều tài khoản độc lập. Label và username không phải khóa chính.
- Phân nhóm nền tảng → tài khoản → chi tiết. 1 tài khoản hiện trực tiếp; 2–4 dùng selector; nhiều hơn dùng danh sách có tìm kiếm. Không tạo vô số thẻ cùng tên nền tảng.
- Tìm theo nền tảng, label, username/email, dự án; không tìm theo giá trị password/token.
- Tài khoản, API key, env bundle và ghi chú có discriminator rõ ràng. Env phân biệt dự án và dev/staging/prod, không tự trộn các môi trường.
- Mỗi thao tác copy gắn rõ account ID và field ID. Hiện mật khẩu là thao tác riêng.
- App không tự đổi mật khẩu tại website khi chỉ sửa mục lưu trong két.
- UI tiếng Việt, kem/xanh sage/cam nhạt, bo mềm, nhận diện lỗ khóa; chữ dễ đọc. Không dashboard, biểu đồ, security score hoặc chú thích dài.
- Bảo mật, Giao diện, Sao lưu, Riêng tư, Thông tin nằm trong Cài đặt. Màn hình Tất cả chỉ có cảnh báo liên quan và tác vụ tài khoản.
- Bảo mật dự kiến: clipboard timeout, lock lifecycle, giới hạn nhập sai, phát hiện dùng trùng, tạo password bằng CSPRNG, kiểm tra HIBP có opt-in, xác thực lại cho tác vụ nhạy cảm, backup mã hóa.
- Chế độ riêng tư chỉ khóa/ẩn cửa sổ và gọi lại bằng phím tắt; không tự đổi tên executable, không di chuyển ngẫu nhiên, không ẩn tiến trình hoặc né phần mềm bảo mật.
- Recovery key, Windows Hello, nhắc hết hạn secrets được ghi backlog riêng. Cloud sync, chia sẻ nhóm, browser extension, autofill, tự đổi mật khẩu website nằm ngoài v1.

## 4. Kiến trúc bắt buộc

Luồng phụ thuộc: React view → feature hook/use-case → typed IPC adapter → Tauri command → Rust application service → domain + port → adapter SQLite/crypto/OS.

- React không gọi SQL, không tự mã hóa vault, không giữ DEK/KEK.
- Chỉ `apps/desktop/src/lib/ipc/` được gọi Tauri invoke/listen; component dùng client có kiểu dữ liệu.
- `crates/vault-core/` không phụ thuộc React/Tauri. Tauri command mỏng: validate DTO, kiểm tra quyền/session, chuyển use-case, ánh xạ lỗi.
- `crates/vault-contracts/` là nguồn chuẩn DTO Rust; sinh TypeScript/schema bằng công cụ đã kiểm chứng phiên bản. Generated files không sửa tay.
- `crates/vault-storage/` sở hữu SQL/migration; `vault-crypto` sở hữu wrapper cho thư viện mật mã đã được kiểm tra; `vault-platform` sở hữu clipboard, lifecycle, dialog và network adapter.
- Dùng constructor/dependency injection rõ ràng; không service locator toàn cục, không giant service, không tạo generic repository vô ích.
- Một worker storage có hàng đợi giới hạn và một SQLite connection trước; mọi ghi tuần tự. KDF/crypto nặng chạy worker riêng có giới hạn. Không chặn WebView hoặc giữ mutex qua network/await.
- Không tạo module rỗng hàng loạt cho backlog. Chỉ tạo module khi có trách nhiệm và consumer thật.

Repo dự kiến: `apps/desktop/`, `crates/`, `packages/contracts/`, `docs/`, `prompts/`, `scripts/`. Cấu trúc cụ thể ở `docs/architecture.md`.

## 5. Hợp đồng IPC trước implementation

1. Codex đề xuất command + DTO + error/event + state/permission; Antigravity phản hồi theo luồng UI.
2. Hai phía ACK cùng version/hash; reviewer đọc phần nhạy cảm. Trạng thái contract: `DRAFT → ACCEPTED → IMPLEMENTED → VERIFIED`.
3. Tên API trong tài liệu là **API dự kiến của dự án**, không phải API có sẵn của Tauri. Chỉ gọi thật khi command được đăng ký và triển khai.
4. Registry ánh xạ tên command, request/response schema, caller window, trạng thái session, lỗi, sự kiện, implementation path, contract version và commit bằng chứng.
5. Codegen có drift check; runtime validate input ở Rust, validate biên IPC ở frontend bằng schema sinh ra hoặc decoder đối chiếu contract.
6. Phân biệt `missing`, `null`, `clear` và `unchanged` trong update. Mật khẩu không tự trim/normalize. Không gửi chuỗi dấu chấm để thay mật khẩu.
7. Mutation dùng `expectedRevision`; sai revision trả `CONFLICT`, không last-write-wins. Mỗi use-case ghi nhiều bảng phải trong transaction. Request phụ thuộc phiên mang `requestId + expectedSessionEpoch`, response mang requestId/epoch thật; Rust từ chối lệnh xếp hàng từ phiên cũ kể cả khi két đã mở lại.
8. Error có code ổn định và dữ liệu an toàn; không stack trace, SQL, đường dẫn riêng tư hoặc secret trong response/log. UI có empty/loading/error/locked/stale/offline riêng.
9. Event chỉ báo trạng thái/ID/revision cần thiết; không secret. Session epoch là dấu thế hệ để loại kết quả cũ, không phải credential để cấp quyền.
10. Stub trả lỗi `NOT_IMPLEMENTED` và UI nhận diện; mock chỉ dev/test có nhãn. Không trả thành công giả để che backend chưa có.

## 6. Mã hóa và dữ liệu

- Argon2id dẫn xuất KEK từ mật khẩu chính + salt CSPRNG. Khóa DEK ngẫu nhiên mã hóa dữ liệu; KEK chỉ bảo vệ DEK. Không hardcode khóa hoặc lưu khóa trần cạnh DB.
- AEAD AES-256-GCM qua thư viện được duy trì; không tự viết primitive/protocol mới. Nonce không lặp cùng khóa; strategy/version/bounds phải có ADR, test và review trước dùng.
- AAD ràng buộc vault ID, record ID, parent ID khi có, kiểu payload, key/schema version và revision theo một encoding ổn định đã đặc tả. Summary và secret dùng envelope phân biệt; sai tag thì từ chối. Baseline dùng một aggregate revision: mỗi update revision/context phải mã hóa lại cả hai envelope hiện có với nonce mới trong cùng transaction. Không giữ ciphertext secret cũ rồi đọc bằng AAD revision mới. Key slot cũng bind vault/suite/format/slot purpose/KDF context theo đặc tả được review.
- SQLite chỉ nhận payload đã mã hóa: tên, label, email, URL, ghi chú, giá trị env, cảnh báo nhạy cảm đều nằm trong envelope. ID, quan hệ, revision, loại bản ghi và kích thước có thể lộ; ghi rõ giới hạn này.
- Không hứa AEAD phát hiện rollback toàn bộ sang bản két cũ hợp lệ. Không hứa xóa vật lý sạch khỏi SSD, backup hoặc RAM.
- Search index metadata ở Rust RAM khi mở két; có thể rebuild theo batch. Không có FTS/plaintext search index trên ổ đĩa; không giải mã tất cả secret chỉ để liệt kê.
- FK, unique constraint và indexes thiết kế theo query thực tế. Mọi bảng con của vault phải không thể trỏ sang record của vault khác.
- Đổi master password thông thường rewrap DEK trong transaction. Lộ khóa/password cũ là luồng khác: cần đánh giá rotation DEK; backup cũ vẫn có thể mở bằng credentials cũ. Recovery slot bị xóa không vô hiệu hóa bản sao cũ.
- Parse header/import như input không tin cậy; giới hạn kích thước, độ sâu, KDF memory/time/parallelism trước cấp phát. Không giảm KDF âm thầm khi máy yếu.
- v1 chọn SQLite rollback journal + durability phù hợp, không tắt sync để làm đẹp benchmark. WAL chỉ thêm khi có nhu cầu đo được, ADR và backup đúng. Kiểm tra engine SQLite thực tế được liên kết, không chỉ version crate.

## 7. Session, race condition và giới hạn bảo mật

- State machine backend tối thiểu `NoVault / Creating / Locked / Unlocking / Unlocked / Locking / Unavailable`. Chỉ backend quyết định truy cập dữ liệu. NoVault chỉ là file chưa tồn tại, không phải file lỗi/quyền đọc lỗi.
- Create có reservation độc quyền trước KDF, lifecycle generation và publish atomic không overwrite. Hai create không cùng thành công. Nếu lock thắng trước publication thì hủy create và trở về NoVault; nếu publication thắng thì vault mới ở Locked. Lock trên NoVault giữ NoVault và trả resulting state, không giả có két.
- Lock là barrier: tăng epoch, thu hồi quyền, chặn tác vụ mới, vô hiệu hóa tác vụ cũ, dọn khóa/secret cache; không đợi network mới khóa. UI mask ngay.
- Unlock/reauth/restore đang chạy xong sau lock không được mở lại két hoặc trả secret. Kiểm tra expected epoch khi nhận request và trước trả dữ liệu. Authorization + clipboard/commit phải tuyến tính hóa với lock trong cùng cơ chế đồng bộ, không check-then-act rời nhau. Create/unlock có expectedLifecycleEpoch để loại yêu cầu cũ trước khi có session mở.
- Không cancel được KDF ngay cũng phải giới hạn job còn chạy, hủy bỏ kết quả, dọn key; không tạo unlock job không giới hạn bằng click liên tục.
- Copy có generation/token và OS clipboard ownership/sequence nếu có; timer cũ không xóa nội dung copy mới. Compare-and-clear phải atomic theo adapter OS hoặc báo rõ best-effort, tránh check rồi ghi rời nhau.
- Clipboard history, cloud clipboard, app khác đã đọc và crash abrupt là giới hạn thật. Không tuyên bố timeout thu hồi tất cả bản sao.
- Frontend reset secrets trên lock, blur, account switch, unmount; loại async response thuộc account/request/epoch cũ. React state không bảo đảm zeroize RAM; giảm thời gian và số bản sao.
- Rate limit unlock/reauth ở Rust, delay có giới hạn, dùng thời gian monotonic cho session. Bộ đếm lưu bền chỉ hạn chế thao tác qua app; người có file/quyền OS có thể bypass. Không tự xóa két khi sai nhiều lần.
- Tác vụ export plaintext, đổi master/recovery và xóa vault cần reauth. Quyền reauth ngắn hạn, gắn action/vault/revision, dùng một lần; không đặt `isReauthenticated=true` trong UI.
- Chỉ một instance ghi vault; mở shortcut gọi instance hiện hữu. Backup/restore/migration là thao tác độc quyền và có failure recovery.
- Không tuyên bố chống malware/admin/debugger trên máy đã bị chiếm. Windows Hello về sau phải bảo vệ đường truy cập key thực sự, không chỉ thêm UI PIN.

## 8. Chức năng bảo mật và quy tắc nghiệp vụ

- Tạo password dùng OS CSPRNG, không `Math.random`, không LLM. Bảo đảm sampling không bias; validate độ dài/charset, xử lý charset rỗng. Mặc định 20 ký tự là lựa chọn sản phẩm, không bảo chứng an toàn.
- Phát hiện trùng chỉ trong vault đang mở, xử lý Rust theo batch, giới hạn concurrency. Không lưu SHA/hash fingerprint mật khẩu ra file; nếu cần HMAC dùng key phiên ngẫu nhiên và hủy khi lock.
- HIBP opt-in; SHA-1 chỉ để tra cứu prefix 5 hex, HTTPS + padding; đối chiếu suffix local, bỏ count=0. Không gửi raw password/full hash/email; không tra cứu theo từng phím gõ. Có timeout, backoff, response-size cap và trạng thái `unknown/checking/found/notFound/error`.
- Kết quả HIBP gắn với credential revision; response cũ không áp lên mật khẩu mới. Hủy/bỏ kết quả khi lock. Không có kết quả ≠ an toàn; bị trùng dataset ≠ tài khoản cụ thể bị hack. Giới hạn privacy của IP/prefix/timing phải được ghi trong thiết kế.
- Xóa clipboard mặc định 20s, có cấu hình; khóa/thoát có xử lý cleanup tốt nhất có thể. Đừng lưu clipboard history riêng.
- Import `.env` không thực thi nội dung, không nội suy shell. Vạch rõ grammar, multiline/quote/comment/duplicate-key; báo conflict, không silent overwrite. Phân biệt key name case và scope theo spec.
- Copy env đơn, dòng và export có serialization/escaping được test; không vô tình chuyển dev thành prod. Xuất nguyên văn phải được xác thực lại và ghi rõ loại file.
- Backup dùng snapshot SQLite nhất quán, không copy bừa file DB đang mở. Restore kiểm tra format/integrity vào vùng tạm rồi đổi an toàn; không phá bản tốt trước khi xác minh.
- Cloud check không được chặn mở két hoặc CRUD. Network mặc định deny ngoài integration đã duyệt. Mọi icon/font phục vụ UI nằm local.

## 9. Quy tắc code và hiệu năng

Các mức dưới đây là ngân sách ban đầu, phải đo trên máy Windows tham chiếu với release build; không được ghi PASS từ suy đoán.

| Hạng mục | Ngưỡng / chính sách |
|---|---|
| React component | Mục tiêu ≤180 dòng; trên 250 phải tách theo trách nhiệm hoặc có reviewer exception |
| Hook / TS service | Mục tiêu ≤150; trên 220 phải giải thích |
| Rust module | Mục tiêu ≤300; trên 450 phải tách hoặc có exception |
| Function | Mục tiêu ≤50 dòng, nesting ≤3; vượt là review trigger, không ép chia vô nghĩa |
| Tests | Theo scenario; >500 dòng xem xét tách. Generated/lockfile/migration được miễn ngưỡng khi hợp lý |
| Query list/count | Số query cố định theo endpoint/page; không tăng tuyến tính theo số item |
| Page | Mặc định 50, max 200; không gửi secret trong list DTO |
| UI interaction | Phản hồi thao tác mục tiêu ≤100ms; không long task >50ms trong đường UI thường gặp |
| Search đã warm index | p95 ≤100ms backend với 10.000 record giả; debounce 150–250ms là độ trễ riêng |
| List/detail metadata | p95 ≤150ms trên máy tham chiếu; KDF/backup/network loại riêng |
| Unlock | KDF hướng tới khoảng 0,5–1,5 giây trên máy tham chiếu; tham số được security review, không hạ để chạy benchmark |
| Khởi động | Mục tiêu p95 ≤2 giây tới màn hình khóa trên máy tham chiếu; đo cold/warm riêng |
| Memory | Ghi baseline toàn process tree; sau 100 vòng unlock/list/copy/lock phải plateau, không tăng tuyến tính; KDF peak đo riêng |

- Đếm dòng không trắng/không comment theo tool đã ghi trong repo; số dòng không thay thế cohesion hoặc readability. Không nén nhiều câu lệnh cùng dòng để lách.
- TypeScript strict; không `any`, `@ts-ignore`, disable lint diện rộng để che lỗi. Escape hatch phải có lý do hẹp và review.
- Rust không `unwrap/expect/panic` trên input/I/O runtime; ngoại lệ bootstrap/test được giải thích. Không `unsafe` tự viết nếu chưa review đặc biệt.
- Duplicate domain rule phải gom vào owner; tránh copy-paste component, SQL, error map và DTO. Không trừu tượng hóa sớm chỉ vì hai đoạn giống nhau bề ngoài.
- Tránh N+1 cả SQL và IPC: list kèm counts bằng aggregate/batch, không gọi từng account để lấy badge/status. Query tests + EXPLAIN QUERY PLAN chứng minh index phù hợp; full scan chủ ý phải có rationale.
- Không full-scan/decrypt vault cho mỗi phím tìm kiếm. Có cancellation/version check; cache bounded theo session/revision. Cache key/result không chứa secret, không persist nhạy cảm.
- Event listener, timer, subscription, observer, worker và background task phải có owner và cleanup; kiểm tra cả React StrictMode và repeated navigation.
- Worker queues bounded, timeout rõ, retry chỉ transient/idempotent, exponential backoff + jitter. Không retry vô hạn unlock/mutation.
- Animation chủ yếu transform/opacity, hỗ trợ reduced motion; không làm UI blur/focus animation cản thao tác. Virtualize khi dataset/đo lường cần, không biến mọi list nhỏ thành framework lớn.

## 10. Lỗi và kiểm thử

Không sửa lỗi bằng cách nuốt exception, trả empty array, fallback thành công, kéo dài timeout vô căn cứ, bỏ test, xóa assertion, tắt CSP/permission, thay dữ liệu thật bằng mock hoặc reset DB mất dữ liệu.

Mỗi bug có: bước tái hiện → nguyên nhân gốc → tác động → sửa tại owner → regression test đúng rủi ro → bằng chứng. Nếu chưa biết nguyên nhân, ghi `UNKNOWN`, bổ sung telemetry đã redact, không bịa kết luận.

Tối thiểu kiểm tra: contract drift; encrypted round-trip; sai mật khẩu/tamper/truncated record; unlock hoàn tất sau lock; request phiên A nằm hàng đợi rồi chạy sau mở phiên B; hai create đồng thời; lock trong create; metadata-only update rồi restart/reveal; stale account response; clipboard generation; mutation conflict/transaction rollback; persistence restart; duplicate credentials; import env xung đột; query count và plan; backup restore khi lỗi ghi; HIBP opt-out/offline/stale response; không secret trong artifacts/log.

Mỗi test chỉ bắt buộc khi feature tương ứng tồn tại; chưa implement ghi `NOT_IMPLEMENTED`, chưa có môi trường ghi `NOT_RUN` hoặc `BLOCKED`. Không làm test vô nghĩa chỉ để tăng coverage. Các invariant bảo mật phải có test, coverage % không thay thế invariant.

CI mục tiêu: format, lint, typecheck, Rust tests, frontend tests, codegen drift, dependency/secret scan và build. Native Windows/clipboard/sleep/lock/packaging cần gate Windows thật; browser preview hoặc Linux build không chứng minh native đã đạt.

## 11. Điều phối và bộ nhớ xuyên tab

- Mỗi implementer dùng worktree riêng; reviewer dùng checkout/worktree chỉ đọc implementation tại commit đã freeze. Không cùng sửa một working tree.
- Có **một thư mục điều phối vật lý ngoài các worktree**, `COORD_ROOT`, tất cả agent trỏ cùng đường dẫn tuyệt đối. Tạo từ `coordination-template/`; không xem bản template trong từng worktree là thông tin live.
- `.agent-local.json` local/gitignored chứa role, repo_root, coord_root; không chứa secret. Nếu khác máy thì cần shared repo/transport thực sự được thiết lập; chat và file riêng không tự đồng bộ.
- Live memory: PROJECT, CURRENT_STATE, TASKS, trạng thái từng role, RFC/message, handoff, review. Chỉ coordinator ghi file tổng hợp; mỗi role ghi file riêng hoặc message mới có tên duy nhất.
- Task có ID, owner, scope paths, base SHA, dependencies, acceptance, status, implementation SHA và review IDs. Không chỉnh file ngoài scope nếu chưa được bàn giao.
- Contract/shared config thay đổi bằng RFC + ACK từ owner. Một writer cho root manifests/lockfiles/CI/Tauri config. Frontend gửi yêu cầu dependency; không hai agent cùng install ở một thư mục.
- Khi contract/baseline được chấp nhận, Codex công bố commit riêng chứa contract/generated bindings/shared config và hash tương ứng. Antigravity đồng bộ đúng commit vào nhánh sạch của mình bằng merge/cherry-pick phù hợp, xác minh hash/lockfiles trước nối API. Đây là đồng bộ được phép, không là quyền sửa các file do bên kia sở hữu. COORD_ROOT không tự truyền code giữa worktrees.
- Mỗi commit/handoff có context checkpoint. Coordinator lưu snapshot không chứa secrets về `docs/collaboration/snapshots/` trong repo để bền vững; live path không dùng được thì phục hồi từ snapshot được commit mới nhất và đánh dấu nguồn.
- Tab mới phải đọc role → AGENTS → PROJECT/CURRENT_STATE/TASKS → status/handoff của role → contract → ADR → review chưa đóng; chạy git status/log để xác minh. Không tin ký ức chat hơn code + contract + bằng chứng đã duyệt.
- Mỗi phiên kết thúc ghi: đã làm, chưa làm, file/commit, test thật, blockers, rủi ro và đúng một next action. Mỗi 30–60 phút hoặc trước compact cũng checkpoint.

## 12. Reviewer và báo cáo chung

Implementer gửi review request khi working tree sạch, có implementation commit SHA, base SHA, contract version, file list và test evidence. Reviewer đọc diff và các caller liên quan, tự chạy kiểm tra khả dụng; không chỉ đọc bản tóm tắt của implementer.

Mỗi finding có ID, severity, file/dòng, đường tái hiện, tác động, cách sửa, test cần thêm. Phân loại P0 Critical, P1 High, P2 Medium, P3 Low. P0/P1 block merge. P2 phải sửa hoặc có chấp nhận rủi ro rõ ràng từ người phụ trách phù hợp; implementer không tự chấp nhận rủi ro của mình. Rủi ro lộ secrets/mất dữ liệu/escalation quyền phải báo người dùng.

Reviewer kết luận `APPROVE / CHANGES_REQUESTED / BLOCKED`, nêu phạm vi và phần NOT_RUN. Commit thay đổi sau review phải được review lại phần thay đổi; approval gắn đúng SHA. Reviewer không sửa implementation trong cùng lượt rồi approve.

Hai reviewer viết báo cáo riêng; coordinator tổng hợp `reports/QUALITY_REPORT.md` trong thư mục chung, không sửa nội dung reviewer để đổi kết luận. Bản tổng hợp dẫn SHA, nguồn evidence, findings mở và người xử lý. Sau merge chạy lại gate tích hợp; nếu conflict resolution tạo code mới thì phải review lại diff đó.

Không merge chỉ vì tất cả ô checklist được tick. Definition of Done: acceptance đạt, review đúng phiên bản, không P0/P1 mở, contract khớp, tests thật, không regression, docs/memory cập nhật và phần còn thiếu được ghi đúng.

## 13. Công việc khởi động ngay

**Codex trước:** kiểm tra repo/toolchain/AGENTS hiện hữu, không ghi đè công việc người dùng. Thiết lập workspace và COORD_ROOT, bootstrapping manifests/CI cơ bản một lần, ghi exact versions đã xác minh. Tạo task ownership. Đề xuất và chốt contract nhỏ với Antigravity. Không tự chạy agent khác nếu công cụ không hỗ trợ; gửi handoff rõ ràng.

**Antigravity:** đọc file chung và task trước. Trong khi chờ contract, dựng token/layout/accessibility + fixtures giả tách biệt; không đoán API. Khi contract được ACK, chuyển feature đã sẵn sàng sang adapter thật. Giữ màn hình Tất cả gọn; settings là tab riêng.

**Checkpoint scaffold đầu tiên:** app shell + navigation Cài đặt + một command không nhạy cảm `vault_status` chạy thật qua IPC. Có contract, build/typecheck khả dụng, status/error trung thực. Review phạm vi này trước khi bắt đầu encrypted slice; không dựng demo lưu secrets plaintext để thay thế.

**Luồng tích hợp đầu tiên:** tạo vault thử bằng dữ liệu giả → khóa/mở lại → tạo một nền tảng với hai tài khoản → chọn đúng tài khoản → copy đúng mật khẩu bằng Rust → khóa két → lệnh đọc/copy bị từ chối. File trên đĩa không chứa secret nguyên văn; có test sai password, tamper và stale response. Clipboard timeout thuộc slice đầu để kiểm tra race. Đây là prototype dùng dữ liệu giả, chưa được tuyên bố production-ready.

Review chặng này trước khi nối HIBP, backup UI, recovery key hoặc Windows Hello. Backlog vẫn phải được ghi đầy đủ. Nếu máy thiếu Windows/Rust/build tools, hoàn thành phần khả dụng, ghi đúng BLOCKED; không giả native test đã chạy, không tự hạ bảo mật để chạy demo.

**Phản hồi đầu tiên của mỗi agent:** role, đường dẫn worktree/coord_root, trạng thái repo đã quan sát, task nhận, contract hiện tại và hành động sắp thực hiện. Sau đó bắt tay làm. Khi xong, giao commit/evidence/handoff có thể kiểm tra, không chỉ mô tả kế hoạch.
