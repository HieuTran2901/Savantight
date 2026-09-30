# Task board — coordinator là writer duy nhất

Các tasks seed dưới đây chưa được claim; source SHA sẽ được điền sau bootstrap. Tham khảo templates/TASK.md cho task chi tiết khi bắt đầu.

| ID | Owner | Scope | Dependencies | Acceptance | Status |
|---|---|---|---|---|---|
| B-001 | Codex | Workspace/toolchain/governance/manifests | Không | Paths/version thực, một COORD_ROOT, app shell build hoặc blocker rõ | TODO |
| C-001 | Codex + FE ACK | IPC nhỏ + codegen + command policy | B-001 | vault_status contract agreed, handler registered, drift gate | TODO |
| F-001 | Antigravity | Tokens/shell/settings navigation | B-001 | UI scaffold có empty/locked, browser mock dev-only nếu cần | TODO |
| R-BOOT-001 | Hai reviewer riêng | Review scaffold/status IPC | C-001,F-001 | Scope review thật, không claim đã review crypto | TODO |
| C-002 | Codex + FE ACK | Contract create/unlock/account/copy/lock | R-BOOT-001 | DTO/errors/epochs/revisions được ACK và chuyển đúng commit | TODO |
| B-002 | Codex | Crypto/storage/session slice | C-002 | Vault giả encrypted, wrong-password/tamper/lock tests | TODO |
| F-002 | Antigravity | Unlock/multi-account/copy adapter | C-002,F-001 | Dùng contract thật; correct ID; stale/lock cleanup | TODO |
| B-003 | Codex | Clipboard timeout + race | B-002 | Copy Rust, generation/native check, error honest | TODO |
| R-BE-001 | BE Reviewer | Review BE candidate | B-002,B-003 | Report đúng SHA, invariant/query/crypto findings | TODO |
| R-FE-001 | FE Reviewer | Review FE candidate | F-002 | Report đúng SHA, lifecycle/UX/API findings | TODO |
| I-001 | Codex | Integration + checkpoint | R-BE-001,R-FE-001 | Approved scope, integration gates, report/snapshot | TODO |

## Backlog sau core slice

- Generator/duplicates settings + streaming scan.
- HIBP opt-in + honest statuses + network limits.
- Settings persisted policies + versioned saves.
- Backup/restore consistent + reauth export/master change.
- Env grammar/import/export with conflict handling.
- Privacy/tray/global shortcut + OS lifecycle.
- Recovery key; expiry reminders; Windows Hello evaluation.
- Release/signing/dependency review và gate Windows hoàn chỉnh.

Backlog không có nghĩa skeleton được phép lưu plaintext tạm thời. S-01/S-02/S-03 nền phải có trước secrets slice.
