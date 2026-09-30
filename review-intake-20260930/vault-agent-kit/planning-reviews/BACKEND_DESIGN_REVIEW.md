# Backend/security design review

- Reviewer: independent agent `/root/security_reviewer`.
- Type: DESIGN_REVIEW.
- Scope: MASTER_PROMPT, docs/architecture, docs/security, docs/contracts/IPC, docs/quality.
- Snapshot manifest SHA-256: `2ba9c3d2477e4a34ee7653d928edadd080c09949365bdf61eb728647101780ba`.
- Initial verdict: CHANGES_REQUESTED.
- Final verdict: DESIGN APPROVE.

## Findings và kết quả review lại

| ID | Severity | Finding ban đầu | Resolution reviewer xác nhận |
|---|---|---|---|
| BE-D01 | P1 | IPC thường chưa buộc vào phiên khởi tạo request | CLOSED: expectedSessionEpoch/requestId, kiểm tra admission và linearized boundary, thêm queued-request qua re-unlock test |
| BE-D02 | P1 | Create thiếu reservation/no-clobber/lifecycle rõ | CLOSED: exclusive reservation, lifecycle epoch, atomic publication, lock/create semantics, NoVault khác unreadable/corrupt |
| BE-D03 | P2 | Hai envelope nhưng một revision trong AAD chưa có strategy | CLOSED: aggregate revision, re-encrypt cả hai envelope hiện có với nonce mới trong transaction, metadata-update/restart test |
| BE-D04 | Suggestion | Header/key-slot authenticated context chưa explicit | CLOSED: vault/suite/format/slot/KDF binding trong spec requirement |
| BE-D05 | Suggestion | Master cần nhắc linearization ở gần epoch checks | CLOSED: cấm check-then-act rời nhau với commit/copy |

## Kết luận reviewer

Các finding trong phạm vi review đã đóng. Bộ thiết kế đủ để bắt đầu scaffold và luồng tích hợp bằng dữ liệu giả. Contract vẫn là draft cần ACK; crypto byte format cụ thể vẫn cần đặc tả và review trước triển khai.

Reviewer đã xác minh checksum của 33 file trong snapshot; checksum xác định phiên bản, không có nghĩa đã review nội dung mọi file ngoài scope.

## Giới hạn

Chưa có source app. Không chạy test chức năng, benchmark, Windows native hoặc audit implementation. Không chứng nhận app an toàn chứa secrets thật hay sẵn sàng phát hành.
