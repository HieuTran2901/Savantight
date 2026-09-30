# Quality gates, hiệu năng và quy tắc chống lỗi

Mọi kết quả ban đầu: NOT_RUN. Budget trong MASTER_PROMPT là mục tiêu, không benchmark thực tế.

## Danh sách rủi ro người dùng yêu cầu

| Rủi ro | Ràng buộc thiết kế | Bằng chứng reviewer cần |
|---|---|---|
| N+1 query | Batch/aggregate list/count, không query/IPC theo từng row | Query count với 1/50/200 records, không tăng theo N |
| Memory leak | Resource có owner + cleanup; bounded caches/jobs | Repeat navigate/unlock/lock, active handles, memory trend |
| Race condition | Backend lock barrier, epoch, clipboard generation, revision CAS | Deterministic interleaving tests, stale response bị bỏ |
| Security vulnerability | Threat model, authz Rust, KDF/AEAD, CSP, input bounds | Tamper/locked direct IPC/adversarial import + review |
| Duplicate code | Owner domain rule và primitive chung | Diff/call-site review, không hai bản DTO/policy trôi nhau |
| API design kém | Versioned contract, errors, pagination, typed adapter | Contract tests/codegen diff; schema behavior rõ |
| Thiếu index | Index theo where/join/order, không index plaintext secret | Migration + EXPLAIN QUERY PLAN trên dữ liệu đại diện |
| Component quá lớn | Presentational vs hook/service, LOC review budget | Cohesion, rerender profile, exception được reviewer duyệt |
| Business logic sai | Invariants platform/account/env/revision rõ | Test nhiều account, trùng username, update partial, env scope |
| AI bịa API | Kiểm chứng official docs/source, contract registry | Link/version/symbol tồn tại + compile/integration thực |
| AI che lỗi | RCA và regression; không fake success/empty catch | Repro trước/sau, root cause, failing case giờ pass |

## Ngân sách đo lường

Thiết lập `docs/benchmarks/environment.md` khi có implementation: CPU/RAM/SSD, Windows/WebView2/build profile, engine version, commit, dataset seed, sample count và workload. Không ghi dữ liệu user thật. Dữ liệu 1k/10k record giả; batch count 50/200.

- Cold start và warm start đo riêng. KDF latency/peak RAM tách khỏi thông thường.
- Search p95 backend warm index ≤100ms, UI feedback ≤100ms; debounce 150–250ms ghi riêng.
- List/detail metadata p95 ≤150ms; không include reveal/network nếu không ghi rõ.
- Interactive main thread không long task >50ms trong trace target; animation tránh layout thrash.
- Memory đo cả Rust và WebView processes sau warmup/GC hợp lý; 100 vòng tương tác, không yêu cầu RSS trở về đúng byte. Tăng tuyến tính hoặc task/handle còn sống là blocker để điều tra.
- Khi không đạt: profile → nguyên nhân → sửa → đo lại; không giảm KDF hoặc tắt checks. Budget override cần rationale/owner/reviewer và dữ liệu, không thay số để xanh.

## Tổ chức source

Ngưỡng LOC ở master là review triggers cho code viết tay. Ngưỡng “trên 250 component/450 Rust module” cần refactor hoặc exception được reviewer phê duyệt, không là giấy phép tạo hàng trăm file bé khó đọc.

Mỗi feature chứa UI/hooks/schema/view-model tests cùng nhau; lib chỉ code thật sự dùng chung. Domain không import UI; contracts không phụ thuộc DB; adapter không tự quyết định business policy. Một hàm một trách nhiệm, tên thể hiện intent, error paths rõ.

Không `utils.ts`/`service.rs` gom mọi thứ, component root hàng nghìn dòng, mega context rerender cả app, literal invoke tràn lan, copied error translation. Không build abstraction chỉ để có clean architecture diagram.

## Test cases có ý nghĩa

1. Create→unlock→save→restart→unlock→copy bằng dữ liệu giả; DB/raw files không chứa marker secret.
2. Wrong password, tag altered, record swapped, truncated file, KDF/header ngoài bounds; giữ file cũ.
3. Direct sensitive command khi locked; reauth sai/action khác/hết TTL/reused; không bypass qua UI.
4. Lock trong unlock/decrypt/copy/HIBP/search; late response không resurrect session.
5. Hai mutation cùng revision; chỉ một thành công, một CONFLICT, không partial commit.
6. Clipboard A/B/new same text/OS busy/old timeout; không xóa newer clipboard.
7. Multi-account grouping, same username hợp lệ, selector >4, search deep-select, correct copy ID.
8. Settings autosave race: sequence/revision, không báo Đã lưu trước ack; failed save hiển thị đúng.
9. Env duplicate/quote/newline/no-shell-execution và scope production; không silent overwrite.
10. Snapshot/restore với disk-full/interruption/corrupt/old version; không mất vault tốt.
11. HIBP opt-out không gửi request; failed≠notFound; stale revision discarded; no secret logs.
12. Generated binding drift, registered command/capabilities match; NOT_IMPLEMENTED không thành success.
13. UI keyboard/focus/reduced motion/1280×800/150% DPI; loading/error/locked không lộ nội dung.
14. Queued command từ session A sau lock/reunlock session B phải STALE_SESSION, không dùng quyền phiên B.
15. Concurrent vault_create, lock trong create, file tồn tại nhưng corrupt, NoVault lock no-op; không overwrite vault.
16. Metadata-only update với aggregate revision rồi restart/reveal/copy vẫn xác thực được cả hai envelope.

Các feature chưa xây chưa phải test pass; matrix task ghi rõ NOT_IMPLEMENTED. Test CLI names phải tồn tại trước khi trích dẫn. Nền tảng không có thì NOT_RUN/BLOCKED.

## Gate CI dự kiến

Codex tạo scripts có thực cho fmt/lint/typecheck/test/contract-check/secret-scan/dependency-audit/build. Tên script không phải kết quả đã chạy. Không bổ sung toàn bộ tooling nặng chỉ để tick checklist; chọn tool thật tương thích và pin.

Rust fmt/clippy/tests; frontend typecheck/lint/targeted tests; codegen diff; build production sans mocks; dependency/secret scan. Native Windows acceptance chạy ở môi trường Windows; công cụ automation browser chỉ kiểm tra WebView UI riêng.

Security boundaries cần targeted regression; không viết test snapshot lặp implementation cho mỗi class CSS. Không mở rộng suite tùy tiện sau khi rủi ro cụ thể đã được giải quyết.
