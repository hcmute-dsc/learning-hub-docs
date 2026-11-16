# USER-FLOWS — Hành trình người dùng

> Thuật ngữ khớp DATABASE.md: `track` > `module` > `lesson` / `exercise`; `submission`, `test_run`, `review`, `mentor_assignment`, `content_version`.
> Nhãn: **[MVP]**, **[P2]**.

## 1. Đăng ký & đăng nhập [MVP]

Không có form đăng ký riêng: lần đăng nhập GitHub đầu tiên tạo tài khoản nếu hợp lệ. Duyệt bằng **danh sách email** của CLB (đã chốt 16/11/2026): API đọc mọi email đã xác minh trên GitHub (`GET /user/emails`, cả email phụ) và so với `approved_emails`. Kiểm tra org `hcmute-dsc` chỉ là tùy chọn bổ sung, không nằm trong luồng mặc định.

```mermaid
flowchart TD
  A[Mở hub] --> B[Bấm Đăng nhập với GitHub]
  B --> C[GitHub OAuth consent]
  C --> D{Callback hợp lệ?<br/>state + PKCE}
  D -- Không --> E[Trang lỗi: thử lại]
  D -- Có --> V[Đọc mọi email đã xác minh<br/>GET /user/emails]
  V --> H{User đã tồn tại?}
  H -- Không --> P{Có email khớp approved_emails<br/>hoặc là BOOTSTRAP_ADMIN_GITHUB_LOGIN?}
  P -- Có --> I[Tạo user status=active + role member + audit log]
  P -- Không --> G[Tạo user status=pending<br/>trang Đang chờ duyệt, admin/mentor duyệt tay]
  H -- Có --> J[Cập nhật avatar/tên<br/>nếu đang pending và email khớp thì active]
  I --> K[Tạo session cookie httpOnly]
  J --> K
  K --> L[Onboarding 3 bước lần đầu<br/>nhập tên hiển thị, chọn mục tiêu, gợi ý track]
  L --> M[Dashboard]
```

## 2. Khám phá learning path [MVP]

```mermaid
flowchart LR
  A[Dashboard/Tracks] --> B[Lọc: level, chủ đề, thời lượng]
  B --> C[Chọn track]
  C --> D[Trang track: mô tả, kết quả học,<br/>danh sách module, prerequisite, người đóng góp]
  D --> E{Đã enroll?}
  E -- Chưa --> F[Nút Bắt đầu học]
  E -- Rồi --> G[Nút Tiếp tục: lesson/exercise kế tiếp]
```

## 3. Enroll vào track [MVP]

```mermaid
sequenceDiagram
  actor M as Member
  participant W as Web
  participant A as API
  participant D as DB
  M->>W: Bấm "Bắt đầu học"
  W->>A: POST /api/v1/tracks/{slug}/enrollments
  A->>D: INSERT enrollment (unique user_id+track_id)
  alt đã tồn tại
    D-->>A: conflict
    A-->>W: 200 enrollment hiện có (idempotent)
  else mới
    D-->>A: enrollment
    A-->>W: 201
  end
  W-->>M: Chuyển tới lesson đầu tiên
```

## 4. Học lesson [MVP]

```mermaid
flowchart TD
  A[Mở lesson] --> B[Đọc Markdown: mục tiêu, nội dung, ví dụ]
  B --> C{Có exercise liên kết?}
  C -- Có --> D[CTA: Làm bài tập]
  C -- Không --> E[Đánh dấu đã học]
  D --> E
  E --> F[lesson_progress = completed]
  F --> G[Lesson kế tiếp / module kế tiếp]
```

Lesson không bắt buộc hoàn thành để làm exercise, trừ khi `prerequisites` khai báo.

## 5. Làm bài tập [MVP]

```mermaid
flowchart TD
  A[Mở exercise] --> B{Prerequisite đã đạt?}
  B -- Chưa --> C[Hiển thị khóa + liên kết tới bài cần làm trước]
  B -- Rồi --> D{Loại exercise}
  D -- code --> E[Đọc đề + repo template/hướng dẫn<br/>làm trên GitHub của bạn]
  D -- self_check --> S[Làm theo checklist, tự đối chiếu đáp án gập<br/>bấm Tôi đã hoàn thành]
  D -- quiz --> F[Làm quiz trong app]
  D -- project --> G[Đọc yêu cầu + tiêu chí review]
  E --> H[Nộp: dán URL PR/commit]
  F --> I[Chấm tức thì, hiện đáp án đúng]
  G --> H
```

## 6. Nộp bài & phản hồi tự động [MVP]

```mermaid
sequenceDiagram
  actor M as Member
  participant W as Web
  participant A as API (submissions)
  participant G as GitHub (learning-hub-checker)
  participant D as DB
  participant X as Discord
  M->>W: Dán URL PR/commit + (tuỳ chọn) ghi chú
  W->>A: POST /api/v1/exercises/{id}/submissions (Idempotency-Key)
  A->>A: Kiểm tra URL thuộc tài khoản GitHub của member, repo công khai
  A->>D: INSERT submission(status=submitted, attempt=n+1, content_version_id)
  A->>G: workflow_dispatch → learning-hub-checker {repo, sha, exercise, submissionId}
  A-->>W: 202 + submission id
  Note over G: Workflow của CLB checkout bài tại sha,<br/>chạy test ẩn (không secret)
  G-->>A: Webhook workflow_run.completed (HMAC, chỉ từ checker repo)
  A->>A: Xác thực chữ ký + repo nguồn, map run → submission theo submissionId
  A->>D: INSERT test_run(status, summary, run_url) + cập nhật submission
  alt passed và exercise type=code (không cần review)
    A->>D: submission.status = passed
  else cần review (type=project hoặc exercise.requires_review)
    A->>D: submission.status = awaiting_review
    A->>X: Thông báo mentor (hàng đợi)
  else failed
    A->>D: submission.status = failed
  end
  W-->>M: Poll/SSE: hiển thị kết quả + link log GitHub
```

Trạng thái submission: `submitted → testing → passed | failed → awaiting_review → approved | changes_requested`. Nộp lại = submission mới, không sửa submission cũ.

## 7. Xin mentor [MVP]

```mermaid
flowchart TD
  A[Member: Nhờ mentor] --> B[Chọn track + viết lời nhắn]
  B --> C[mentor_assignment: status=requested]
  C --> D[Thông báo Discord kênh mentor]
  D --> E{Ai xử lý?}
  E -- Mentor tự nhận --> F[status=active, mentor_id]
  E -- Admin gán --> F
  E -- Từ chối/hết chỗ --> G[status=declined + lý do]
  F --> H[Member thấy tên mentor + bài nộp tự vào hàng đợi mentor]
  H --> I[Kết thúc: status=ended hoặc đổi mentor, giữ lịch sử]
```

## 8. Mentor review [MVP]

```mermaid
flowchart TD
  A[Mentor mở Hàng đợi review] --> B[Lọc: mentee của tôi / toàn track / cũ nhất]
  B --> C[Mở submission: diff PR + kết quả test + lịch sử lần nộp]
  C --> D[Viết comment theo file/dòng<br/>review_comments]
  D --> E{Kết luận}
  E -- Đạt --> F[review.status=approved → exercise hoàn thành]
  E -- Cần sửa --> G[review.status=changes_requested]
  F --> H[Thông báo Discord cho member]
  G --> H
  H --> I[Member sửa, nộp lại → submission mới quay về bước test]
```

**Đã chốt (16/11/2026):** kết luận review (`approved` / `changes_requested`) luôn ghi ở hub (`reviews`); thảo luận chi tiết làm được ở cả hub (`review_comments`) và GitHub (comment PR), hub lưu liên kết tới PR. Thời gian review mục tiêu cấu hình theo học kỳ (`REVIEW_TARGET_HOURS`, mặc định 72 giờ).

## 9. Đóng góp nội dung [MVP]

```mermaid
flowchart TD
  A[Thành viên muốn thêm/sửa bài] --> B[Fork/branch learning-hub-content]
  B --> C[Viết lesson.md/exercise.yaml theo CONTENT-SPECIFICATION]
  C --> D[Chạy hub-content-validate cục bộ]
  D --> E[Mở Pull Request]
  E --> F[CI: hub-content-validate - schema, tham chiếu, prerequisite, link/gitleaks dự kiến]
  F --> G{CODEOWNERS review}
  G -- Cần sửa --> C
  G -- Duyệt --> H[Merge vào main]
  H --> I[Webhook/Action/CLI sync → hub tạo content_version draft]
  I --> J[Admin xem diff và Publish]
```

Phase 2: nút "Sửa trên GitHub" trên mỗi lesson, preview PR.

## 10. Admin publish learning path [MVP]

Hai bước tách biệt: **sync** chỉ đọc + validate + ghi một `content_version` (không đụng catalog); **publish** mới ghi catalog.

```mermaid
sequenceDiagram
  actor Mt as Content maintainer
  actor Ad as Admin
  participant Hub as API (content)
  participant CS as ContentSource (GitHub / LocalDir)
  participant DB as DB
  Mt->>Hub: POST /api/v1/admin/content/sync (commit_sha tùy chọn)
  Hub->>CS: getHead() nếu không có SHA, đọc cây file tại commit SHA
  Hub->>DB: Đã có version (source_repo, commit_sha)? → trả lại version đó (idempotent)
  Hub->>Hub: Validate (@hub/content) + khóa tự nhiên so với DB
  alt lỗi
    Hub->>DB: INSERT content_version(status=rejected, summary.issues)
    Hub-->>Mt: 422 danh sách lỗi theo file
  else hợp lệ
    Hub->>DB: INSERT content_version(status=draft, commit_sha, summary.diff)
    Hub-->>Mt: Diff so với catalog hiện tại (thêm/sửa/xóa/không đổi theo loại)
  end
  Note over Hub,DB: Sync KHÔNG ghi tracks/modules/lessons/exercises
  Ad->>Hub: POST /api/v1/admin/content/versions/{id}/publish
  Hub->>CS: Đọc lại cây file tại đúng commit SHA đã ghim
  Hub->>Hub: Validate lại (+ khóa tự nhiên)
  alt lỗi / nguồn đã đổi
    Hub-->>Ad: 422 (version giữ nguyên draft)
  else hợp lệ
    Hub->>DB: 1 giao dịch: bản published trước=superseded, upsert item mới/đổi (theo uid, content_hash) gắn content_version_id, item biến mất → archived, thay exercise_prerequisites, version=published
    Hub->>DB: audit_logs
    Hub-->>Ad: 200 version published
  end
```

- **Sync** idempotent theo `(source_repo, commit_sha)`: sync lại cùng commit trả về version đã có, không tạo bản ghi mới.
- **`rejected`** = sync gặp lỗi validate; `summary` giữ danh sách lỗi để maintainer sửa trong PR mới (commit mới ⇒ version mới).
- **Publish** chỉ upsert item mới/đổi (so `content_hash`), không bao giờ xóa cứng: item không còn trong Git chuyển `archived`. Kiểm tra khóa tự nhiên chặn trường hợp `uid` bị đổi (item "mới" trùng cha + slug, hoặc track slug, của bản ghi có `id` khác).
- MVP-Lite chưa có UI admin: maintainer chạy `pnpm --filter @hub/api content:sync [--sha <sha>] [--publish]` (cùng service với endpoint). UI xem diff/publish ở MVP-Full.

Bài đã nộp giữ `content_version_id` lúc nộp, nên đổi đề sau đó không làm hỏng lịch sử.

## 11. Luồng lỗi & biên (tóm tắt)

| Tình huống | Hành vi |
| --- | --- |
| Webhook GitHub không tới | Job đối soát mỗi 5 phút (cron trong API) gọi GitHub API đọc workflow run của `learning-hub-checker` theo `submissionId`; sau 15 phút quá hạn → `test_run.status = errored`, member có nút "Chạy lại". |
| Repo của member là private hoặc đã bị xóa | Báo lỗi rõ khi nộp: bài `code` yêu cầu repo công khai (hướng dẫn đổi visibility). |
| Tài khoản GitHub mới đăng nhập, chưa được duyệt | Trạng thái `pending`: thấy trang "đang chờ duyệt", không vào catalog; Ban/mentor duyệt trong trang admin. |
| Content publish làm bài đang làm dở bị xóa | Exercise bị gỡ giữ lại ở trạng thái `archived` đối với người đã nộp; không xóa cứng. |
| Mentor nghỉ | Admin chuyển mentee sang mentor khác; assignment cũ `ended`. |
