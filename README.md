# HCMUTE DSC Learning Hub — bộ tài liệu thiết kế

Nền tảng học tập nội bộ cho thành viên **HCMUTE Developer Student Club**, lấy cảm hứng từ [Exercism](https://exercism.org/): học bằng cách làm, có bài tập, có mentor review. **Không phải SaaS** — một nền tảng cho một CLB, thiết kế để các thế hệ sinh viên kế tiếp bảo trì được.

> Trạng thái: **tài liệu thiết kế v0.1** (ADR-01…ADR-09 đã chốt 16/11/2026) · Mã ứng dụng đang dựng trong monorepo `learning-hub` (E3 đang làm) · Ngày audit repo: 2026-11-16.
> Ngôn ngữ: tiếng Việt (thuật ngữ kỹ thuật giữ tiếng Anh).

## Đọc theo thứ tự nào?

| Bạn là… | Đọc |
| --- | --- |
| Ban chủ nhiệm | README này → [CRITIQUE](docs/product/CRITIQUE.md) → [DECISIONS](docs/product/DECISIONS.md) → [PRD](docs/product/PRD.md) |
| Mentor / người viết nội dung | [USER-FLOWS](docs/product/USER-FLOWS.md) → [CONTENT-SPECIFICATION](docs/product/CONTENT-SPECIFICATION.md) |
| Tech lead / người trực web | [GH-CLI](docs/ops/GH-CLI.md) → [DEPLOYMENT](docs/ops/DEPLOYMENT.md) → [WORKFLOW](docs/ops/WORKFLOW.md) |
| Người muốn đóng góp | [WORKFLOW](docs/ops/WORKFLOW.md) → [GitHub Project](https://github.com/orgs/hcmute-dsc/projects/1) |
| Dev | [ARCHITECTURE](docs/product/ARCHITECTURE.md) → [DATABASE](docs/product/DATABASE.md) → [BACKEND-DESIGN](docs/product/BACKEND-DESIGN.md) → [FRONTEND-DESIGN](docs/product/FRONTEND-DESIGN.md) → [SECURITY](docs/product/SECURITY.md) → [MVP-BACKLOG](docs/product/MVP-BACKLOG.md) |

## Mục lục tài liệu (`docs/product/`)

| Tài liệu | Nội dung |
| --- | --- |
| [REPOSITORY-AUDIT.md](docs/product/REPOSITORY-AUDIT.md) | Kiểm kê repo org `hcmute-dsc`, mức chắc chắn, đề xuất KEEP/INTEGRATE/… |
| [DECISIONS.md](docs/product/DECISIONS.md) | 9 quyết định kiến trúc: options, ưu/nhược, quyết định (đã chốt 16/11/2026) |
| [CRITIQUE.md](docs/product/CRITIQUE.md) | **Phản biện** thiết kế cho bối cảnh CLB học thuật; lỗi đã phát hiện; Phase 0 |
| [PRD.md](docs/product/PRD.md) | Tầm nhìn, persona, vai trò, yêu cầu, user stories, MVP/P2/Future, chỉ số, rủi ro |
| [USER-FLOWS.md](docs/product/USER-FLOWS.md) | Hành trình người dùng (Mermaid) |
| [INFORMATION-ARCHITECTURE.md](docs/product/INFORMATION-ARCHITECTURE.md) | Điều hướng, sitemap, dashboard, cấu trúc học |
| [DATABASE.md](docs/product/DATABASE.md) | Schema PostgreSQL, ERD, ràng buộc, index, migration, xóa mềm, lưu giữ |
| [ARCHITECTURE.md](docs/product/ARCHITECTURE.md) | Kiến trúc hệ thống, module, GitHub/Discord, job, triển khai |
| [BACKEND-DESIGN.md](docs/product/BACKEND-DESIGN.md) | NestJS: module, quy ước API, endpoint, event |
| [FRONTEND-DESIGN.md](docs/product/FRONTEND-DESIGN.md) | Next.js: route, component, state, a11y, responsive |
| [CONTENT-SPECIFICATION.md](docs/product/CONTENT-SPECIFICATION.md) | Nội dung bằng Git: schema YAML/Markdown, validate, version theo SHA, import |
| [SECURITY.md](docs/product/SECURITY.md) | Xác thực, RBAC, webhook, sandbox, secret, audit |
| [MVP-BACKLOG.md](docs/product/MVP-BACKLOG.md) | Epic → story → task, MoSCoW, lịch thực hiện |

## Vận hành (`docs/ops/`)

| Tài liệu | Nội dung |
| --- | --- |
| [DEPLOYMENT.md](docs/ops/DEPLOYMENT.md) | Hiện trạng production, auto deploy bằng GitHub Actions, cài lại từ đầu (user `learnhub`, SSH key, secret, sao lưu, Discord), lỗi đã gặp, bàn giao |
| [WORKFLOW.md](docs/ops/WORKFLOW.md) | Quy trình làm việc với [GitHub Project](https://github.com/orgs/hcmute-dsc/projects/1): field, vòng đời issue/PR, họp sprint |
| [GH-CLI.md](docs/ops/GH-CLI.md) | Cài và cấu hình GitHub CLI: quyền (scope) cần cấp, lệnh vận hành hay dùng, lỗi đã gặp, bàn giao |

## Tóm tắt (11 mục theo yêu cầu)

1. **Hệ sinh thái hiện có.** 19 repo public + 4 private. Nội dung học rải rác (sql-challenge/VitePress, html-css/React data, mentoring-program/bản đồ TS); hai stack song song (Java/Spring cũ, TS/React mới); chưa có theo dõi tiến độ, bài nộp, review. Hoạt động gần đây tập trung vào site tĩnh. *(REPOSITORY-AUDIT §2)*
2. **Audit & tái sử dụng.** study-jam-api → INTEGRATE/DEPRECATE; study-jam-ui → KEEP; mentoring-program → KEEP+INTEGRATE; Course-DSC → DEPRECATE; **sql-challenge → INTEGRATE (nội dung đầu tiên)**; html-css → INTEGRATE dữ liệu; .github → REFACTOR. Repo private: Unknown/cần xác nhận. *(REPOSITORY-AUDIT §3–4)*
3. **Tầm nhìn & MVP.** Tracks → Modules → Lessons/Exercises; 4 loại bài (`code`, `quiz`, `self_check`, `project`); nộp qua GitHub; mentor review; nội dung từ Git. MVP chia Lite/Full, **bắt đầu bằng Phase 0** (đã chốt: content repo + import SQL Challenge + một đợt thử bằng công cụ sẵn có; khung app E1–E3 đã có). *(PRD §10, CRITIQUE)*
4. **Kiến trúc.** Modular monolith NestJS + Next.js + PostgreSQL; 3 tiến trình ở MVP-Lite (MVP-Full thêm `sql-checker` + PostgreSQL riêng); không Redis/MinIO/microservice; adapter GitHub/Discord cô lập sau các port. *(ARCHITECTURE)*
5. **Database.** 18 bảng domain (gồm `approved_emails`) + 3 bảng hạ tầng; MVP-Full thêm `push_subscriptions` + `users.leaderboard_opt_in`; ràng buộc idempotency, 1 version published, 1 review mở/submission. *(DATABASE)*
6. **API.** `/api/v1`, cursor pagination, lỗi `problem+json`, `Idempotency-Key`, OpenAPI → client sinh tự động. *(BACKEND-DESIGN §3–4)*
7. **Nội dung.** Mỗi track một repo `learning-hub-track-<slug>`, `learning-hub-content` là registry ghim commit của từng track (ADR-10); `uid` ổn định (UUID v7); schema Zod + validator `@hub/content` chạy trong CI; sync tạo version `draft`, publish mới ghi catalog (1 version = 1 commit SHA); trừu tượng `ContentSource`. *(CONTENT-SPECIFICATION)*
8. **Chiến lược repo.** `learning-hub` (monorepo), `learning-hub-content` (công khai, registry), `learning-hub-track-<slug>` (mỗi track một repo, ADR-10), `learning-hub-checker` (riêng tư, test ẩn), `learning-hub-docs`; `learning-hub-runner` (Judge0/Piston) chỉ tương lai và trên VPS riêng. Đã tạo (công khai, 16/11/2026): `learning-hub`, `learning-hub-docs`, `learning-hub-content` (track `sql-30-days`). Chưa tạo: `learning-hub-checker`. *(DECISIONS ADR-09)*
9. **Bảo mật.** GitHub OAuth + duyệt thành viên bằng danh sách email của CLB, session DB, RBAC + policy tài nguyên, HMAC webhook, **CI chấm do CLB sở hữu** (không tin workflow của thành viên), `sql-checker` cô lập (DB riêng, read-only, timeout) ở MVP-Full, sandbox nhiều ngôn ngữ chỉ tương lai trên VPS riêng. *(SECURITY)*
10. **Lộ trình.** Phase 0 (4–6 tuần) → MVP-Lite (~8 tuần) → MVP-Full (~4–6 tuần: `sql-checker` ngay sau MVP-Lite, bảng xếp hạng opt-in, Web Push, …), nhóm 3–6 sinh viên; Phase 2 có chứng nhận hoàn thành track. *(MVP-BACKLOG §4)*
11. **Quyết định & câu hỏi mở.** Xem bên dưới.

## Quyết định đã chốt (16/11/2026)

Do Chủ nhiệm CLB quyết định; chi tiết ở CRITIQUE §8 và các tài liệu được dẫn.

1. **Phase 0: có.** Khung app E1–E3 đã dựng; Phase 0 = content repo + import SQL Challenge + một đợt thử, chưa thêm tính năng app cho tới quyết định đi/không đi (tiêu chí đề xuất ở CRITIQUE §6). *(MVP-BACKLOG §0, PRD §10)*
2. **Chủ sở hữu tài khoản/hạ tầng:** Chủ nhiệm CLB đương nhiệm (GitHub `TienNHM`) giữ GitHub App/OAuth app, domain, VPS, secret và tài trợ VPS. Hosting = 1 VPS + Docker Compose. Runbook bàn giao liệt kê mọi tài khoản/secret theo chức danh để chuyển cho Chủ nhiệm kế tiếp. *(ARCHITECTURE §14, MVP-BACKLOG E11)*
3. **Ai được vào & duyệt thành viên:** duyệt bằng **danh sách email** do CLB duy trì. Khi đăng nhập GitHub, API đọc mọi email đã xác minh (`GET /user/emails`, cả email phụ); khớp danh sách → `active`, còn lại `pending` để admin/mentor duyệt tay. Email trường có thể thêm vào GitHub làm email phụ đã xác minh. Admin quản lý danh sách trong app (bảng `approved_emails`); dữ liệu cá nhân → chỉ admin, thay đổi ghi audit, không xuất cho member. Org GitHub chỉ là kiểm tra tùy chọn; giữ `BOOTSTRAP_ADMIN_GITHUB_LOGIN` cho admin đầu tiên. *(SECURITY §1, USER-FLOWS §1, DATABASE §3.1b, MVP-BACKLOG E2)*
4. **Mentor:** số mentor, giới hạn mentee/mentor, thời gian review mục tiêu và cách ghi nhận được chốt **theo từng học kỳ**; giữ ở cấu hình (`MENTOR_MAX_MENTEES`, `REVIEW_TARGET_HOURS`), 72 giờ chỉ là mặc định. *(BACKEND-DESIGN §2 mentorship, PRD §11)*
5. **Giấy phép nội dung** (giao tech lead): nội dung import giữ giấy phép gốc (SQL Challenge MIT → track `sql-30-days` dùng MIT, giữ tác giả + link nguồn); nội dung mới CC BY-SA 4.0 (văn bản) + MIT (code mẫu); người import xác nhận nguồn/tác giả trong mô tả PR. *(CONTENT-SPECIFICATION §10)*
6. **Chứng nhận:** hub tự cấp chứng nhận hoàn thành mỗi track (trang/link xác minh được: tên, track, ngày hoàn thành, phiên bản nội dung) ở **P2**. Quy đổi điểm rèn luyện: ngoài phạm vi, chưa xác nhận. *(PRD F19)*
7. **Review:** kết luận (`approved`/`changes_requested`) ghi ở hub; thảo luận chi tiết ở cả hub và GitHub. *(USER-FLOWS §8)*
8. **ADR-01…ADR-09 đã chốt:** ứng dụng mới NestJS + Next.js; bản đồ mentoring độc lập, tác vụ mentoring trong hub; import SQL Challenge (đã xong), Course-DSC deprecate; monorepo pnpm + content repo riêng; Git là nguồn sự thật; GitHub là chính (Google P2). *(DECISIONS)*
9. **Chấm bài (ADR-07):** A + D ở MVP; ngay sau MVP-Lite thêm `sql-checker` chỉ cho SQL trên VPS (PostgreSQL riêng, read-only, timeout); Judge0/Piston chỉ Future, không bao giờ chung VPS với hub. *(ARCHITECTURE §7, SECURITY §7, MVP-BACKLOG E13)*
10. **Bảng xếp hạng & push (ADR-08):** giữ ở MVP-Full — bảng xếp hạng opt-in theo học kỳ (top 10 + hạng của mình, không theo tốc độ, không toàn thời gian) và Web Push (Discord là kênh thứ hai). *(CRITIQUE §3.4, MVP-BACKLOG E14–E15)*

### Chốt thêm (16/11/2026)

- **Tech lead:** TienNHM. Ban chủ nhiệm CLB thay **mỗi năm một lần** → bàn giao (runbook, tài khoản, secret theo vai trò) diễn ra hằng năm, cùng kỳ đổi Ban chủ nhiệm. *(CRITIQUE §2.2, MVP-BACKLOG E11)*
- **Mentor:** onboarding có điều khoản "mentor góp ý, **không** chịu trách nhiệm học thuật chính thức". *(CRITIQUE §4.2)*
- **Dữ liệu thành viên** (kể cả danh sách email): **không** cần đăng ký với nhà trường; lưu **riêng tư** (chỉ admin truy cập, không công khai, không chia sẻ). *(CRITIQUE §5, SECURITY §13)*

## Câu hỏi còn mở

1. study-jam-api / study-jam-ui còn chạy production không? Chi phí AWS hiện tại? *(AUDIT)*
2. Ai sở hữu `management-service`, `HCMUTE-DSC-be`? Có dữ liệu thành viên cần dùng lại?
3. Course-DSC và các repo legacy: có đồng ý archive?
4. Ngoại lệ tạm `CONTENT_GITHUB_TOKEN` (PAT fine-grained, read-only, tuỳ chọn — SECURITY §10): có dùng ở MVP-Lite không, và tài khoản nào tạo/giữ token cho tới khi có GitHub App?
5. Người kế nhiệm tech lead: chưa chỉ định (nên chọn trước kỳ bàn giao Ban chủ nhiệm hằng năm).

## Quy ước

- Nhãn phạm vi: **MVP**, **P2** (Phase 2), **F** (Future).
- Nhãn bằng chứng trong audit: Verified / Partially verified / Unknown / Requires human confirmation.
- Mermaid hiển thị trực tiếp trên GitHub.

## Cách đóng góp vào tài liệu

Sửa bằng PR; mọi thay đổi ảnh hưởng nhiều tài liệu (tên bảng, vai trò, endpoint, trạng thái) phải cập nhật **đồng thời** ở tất cả tài liệu liên quan (xem checklist nhất quán ở cuối file này).

### Checklist nhất quán
Tên bảng/cột ↔ DATABASE; vai trò ↔ PRD §5 & SECURITY §2; trạng thái `submission` ↔ USER-FLOWS §6 & DATABASE; endpoint ↔ BACKEND-DESIGN §4; loại bài ↔ PRD F4, DATABASE, CONTENT-SPECIFICATION; luồng chấm ↔ DECISIONS ADR-07, ARCHITECTURE §7, SECURITY §7.
