# HCMUTE DSC Learning Hub — bộ tài liệu thiết kế

Nền tảng học tập nội bộ cho thành viên **HCMUTE Developer Student Club**, lấy cảm hứng từ [Exercism](https://exercism.org/): học bằng cách làm, có bài tập, có mentor review. **Không phải SaaS** — một nền tảng cho một CLB, thiết kế để các thế hệ sinh viên kế tiếp bảo trì được.

> Trạng thái: **tài liệu thiết kế v0.1 (nháp, chờ Ban chủ nhiệm duyệt)** · Mã ứng dụng đang dựng trong monorepo `learning-hub` (E3 đang làm) · Ngày audit repo: 2026-11-16.
> Ngôn ngữ: tiếng Việt (thuật ngữ kỹ thuật giữ tiếng Anh).

## Đọc theo thứ tự nào?

| Bạn là… | Đọc |
| --- | --- |
| Ban chủ nhiệm | README này → [CRITIQUE](docs/product/CRITIQUE.md) → [DECISIONS](docs/product/DECISIONS.md) → [PRD](docs/product/PRD.md) |
| Mentor / người viết nội dung | [USER-FLOWS](docs/product/USER-FLOWS.md) → [CONTENT-SPECIFICATION](docs/product/CONTENT-SPECIFICATION.md) |
| Dev | [ARCHITECTURE](docs/product/ARCHITECTURE.md) → [DATABASE](docs/product/DATABASE.md) → [BACKEND-DESIGN](docs/product/BACKEND-DESIGN.md) → [FRONTEND-DESIGN](docs/product/FRONTEND-DESIGN.md) → [SECURITY](docs/product/SECURITY.md) → [MVP-BACKLOG](docs/product/MVP-BACKLOG.md) |

## Mục lục tài liệu (`docs/product/`)

| Tài liệu | Nội dung |
| --- | --- |
| [REPOSITORY-AUDIT.md](docs/product/REPOSITORY-AUDIT.md) | Kiểm kê repo org `hcmute-dsc`, mức chắc chắn, đề xuất KEEP/INTEGRATE/… |
| [DECISIONS.md](docs/product/DECISIONS.md) | 9 quyết định kiến trúc: options, ưu/nhược, đề xuất |
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

## Tóm tắt (11 mục theo yêu cầu)

1. **Hệ sinh thái hiện có.** 19 repo public + 4 private. Nội dung học rải rác (sql-challenge/VitePress, html-css/React data, mentoring-program/bản đồ TS); hai stack song song (Java/Spring cũ, TS/React mới); chưa có theo dõi tiến độ, bài nộp, review. Hoạt động gần đây tập trung vào site tĩnh. *(REPOSITORY-AUDIT §2)*
2. **Audit & tái sử dụng.** study-jam-api → INTEGRATE/DEPRECATE; study-jam-ui → KEEP; mentoring-program → KEEP+INTEGRATE; Course-DSC → DEPRECATE; **sql-challenge → INTEGRATE (nội dung đầu tiên)**; html-css → INTEGRATE dữ liệu; .github → REFACTOR. Repo private: Unknown/cần xác nhận. *(REPOSITORY-AUDIT §3–4)*
3. **Tầm nhìn & MVP.** Tracks → Modules → Lessons/Exercises; 4 loại bài (`code`, `quiz`, `self_check`, `project`); nộp qua GitHub; mentor review; nội dung từ Git. MVP chia Lite/Full, **bắt đầu bằng Phase 0** (thử nhu cầu bằng công cụ sẵn có). *(PRD §10, CRITIQUE)*
4. **Kiến trúc.** Modular monolith NestJS + Next.js + PostgreSQL; 3 tiến trình ở MVP; không Redis/MinIO/microservice; adapter GitHub/Discord cô lập sau các port. *(ARCHITECTURE)*
5. **Database.** 17 bảng domain + 3 bảng hạ tầng; ràng buộc idempotency, 1 version published, 1 review mở/submission. *(DATABASE)*
6. **API.** `/api/v1`, cursor pagination, lỗi `problem+json`, `Idempotency-Key`, OpenAPI → client sinh tự động. *(BACKEND-DESIGN §3–4)*
7. **Nội dung.** Repo `learning-hub-content`; `uid` ổn định (UUID v7); schema Zod + validator `@hub/content` chạy trong CI; sync tạo version `draft`, publish mới ghi catalog (1 version = 1 commit SHA); trừu tượng `ContentSource`. *(CONTENT-SPECIFICATION)*
8. **Chiến lược repo.** `learning-hub` (monorepo), `learning-hub-content` (công khai), `learning-hub-checker` (riêng tư, test ẩn), `learning-hub-docs`; `learning-hub-runner` chỉ tương lai. Đã tạo: `learning-hub` và `learning-hub-docs` (công khai, 16/11/2026). Chưa tạo: `learning-hub-content`, `learning-hub-checker`. *(DECISIONS ADR-09)*
9. **Bảo mật.** GitHub OAuth, session DB, RBAC + policy tài nguyên, HMAC webhook, **CI chấm do CLB sở hữu** (không tin workflow của thành viên), sandbox riêng cho tương lai. *(SECURITY)*
10. **Lộ trình.** Phase 0 (4–6 tuần) → MVP-Lite (~8 tuần) → MVP-Full (~4 tuần), nhóm 3–6 sinh viên. *(MVP-BACKLOG §4)*
11. **Câu hỏi mở.** Xem bên dưới.

## Câu hỏi mở cần CLB quyết định

1. Có chạy **Phase 0** trước khi xây app không? Tiêu chí đi/không đi có chấp nhận được không? *(CRITIQUE §6)*
2. study-jam-api / study-jam-ui còn chạy production không? Chi phí AWS hiện tại? *(AUDIT)*
3. Ai sở hữu `management-service`, `HCMUTE-DSC-be`? Có dữ liệu thành viên cần dùng lại?
4. Mở cho **toàn trường** hay chỉ thành viên? Quy trình duyệt thành viên mới (GitHub org, allowlist, email trường)?
5. Số mentor cam kết, giới hạn mentee/mentor, mục tiêu thời gian review, cách ghi nhận mentor?
6. Hosting: nhà cung cấp, ngân sách, ai giữ tài khoản/domain/GitHub App/secret?
7. Giấy phép nội dung (đề xuất CC BY-SA 4.0 + MIT) và quy trình xác nhận bản quyền khi import (it-quiz, html-css)?
8. Review ghi ở hub hay ở GitHub (đề xuất: kết luận ở hub, thảo luận ở cả hai)?
9. Có liên kết điểm rèn luyện/chứng nhận hoạt động với nhà trường không? *(chưa kiểm chứng)*
10. Ai là tech lead và người kế nhiệm? Chu kỳ bàn giao mỗi nhiệm kỳ?
11. Course-DSC và các repo legacy: có đồng ý archive?

## Quy ước

- Nhãn phạm vi: **MVP**, **P2** (Phase 2), **F** (Future).
- Nhãn bằng chứng trong audit: Verified / Partially verified / Unknown / Requires human confirmation.
- Mermaid hiển thị trực tiếp trên GitHub.

## Cách đóng góp vào tài liệu

Sửa bằng PR; mọi thay đổi ảnh hưởng nhiều tài liệu (tên bảng, vai trò, endpoint, trạng thái) phải cập nhật **đồng thời** ở tất cả tài liệu liên quan (xem checklist nhất quán ở cuối file này).

### Checklist nhất quán
Tên bảng/cột ↔ DATABASE; vai trò ↔ PRD §5 & SECURITY §2; trạng thái `submission` ↔ USER-FLOWS §6 & DATABASE; endpoint ↔ BACKEND-DESIGN §4; loại bài ↔ PRD F4, DATABASE, CONTENT-SPECIFICATION; luồng chấm ↔ DECISIONS ADR-07, ARCHITECTURE §7, SECURITY §7.
