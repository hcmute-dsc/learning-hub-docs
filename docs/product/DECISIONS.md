# DECISIONS — Các quyết định kiến trúc

> Mỗi quyết định có: Options · Ưu · Nhược · Đề xuất · Lý do. Trạng thái tất cả: **Đề xuất, cần Ban chủ nhiệm duyệt**. Nguyên tắc chung: không viết lại chỉ để đồng nhất công nghệ; ưu tiên chi phí bảo trì và dễ onboard thành viên mới.

## ADR-01 — Mở rộng Study Jam hay xây ứng dụng mới?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. Mở rộng study-jam-api/ui | Tận dụng user/role/CI/deploy sẵn; không phải dựng lại | Domain khác (event/task/marking); Java + Ant Design khác stack định hướng; repo không cập nhật từ 2025; thành viên mới ít người biết Spring; phải gỡ nhiều thứ trước khi thêm lesson/submission |
| B. Ứng dụng mới (NestJS + Next.js) | Domain sạch; một ngôn ngữ (TypeScript) cho cả FE/BE/content tooling; khớp stack yêu cầu | Phải dựng auth, CI/CD lại; hai hệ thống cùng tồn tại một thời gian |
| C. Mới nhưng đọc dữ liệu cũ qua API | Giữ dữ liệu Study Jam | Phụ thuộc hệ thống ít được bảo trì |

**Đề xuất: B**, giữ Study Jam chạy nguyên trạng (KEEP) đến hết khóa hiện tại rồi DEPRECATE.
**Lý do:** study-jam-api chỉ có `User/Event/Task`, không có phần nào của domain học tập; chi phí "tận dụng" thấp hơn chi phí gỡ. Không migrate dữ liệu ở MVP; nếu cần, nhập danh sách thành viên một lần bằng script.

## ADR-02 — mentoring-program độc lập hay thành module của Learning Hub?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. Giữ độc lập | Đang hoạt động tốt, deploy Pages đơn giản, không rủi ro | Hai nơi cho thông tin mentoring |
| B. Gộp vào hub (route `/mentoring`) | Một cổng duy nhất | Phải port React/Vite canvas sang Next; không đem lại giá trị MVP |
| C. Tách đôi: bản đồ độc lập (nội dung), tác vụ mentoring (ghép cặp, review) trong hub | Mỗi bên làm đúng việc | Cần đồng bộ nội dung roadmap |

**Đề xuất: C** — mentoring-program giữ vai trò "bản đồ chương trình" (KEEP); module `mentorship` của hub xử lý mentor assignment và review. Phase 2: roadmap mentoring là YAML trong content repo, bản đồ đọc từ đó.

## ADR-03 — SQL Challenge và Course-DSC có import vào content repo trung tâm?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. Import cả hai | Một nguồn | Course-DSC không có nội dung → vô nghĩa |
| B. Import SQL Challenge; HTML/CSS (dữ liệu); Course-DSC deprecate | Nội dung thật được tái dùng; chi phí thấp | Cần script convert front-matter |
| C. Không import, hub trỏ link ra ngoài | Không tốn công | Không theo dõi tiến độ/bài nộp được |

**Đề xuất: B.** SQL Challenge là lộ trình mẫu (30 bài, "cần biết trước", bài tập, đáp án) — gần như đã đúng mô hình Track → Lesson → Exercise. Import bằng script một lần, giữ lịch sử bằng subtree/ghi `source:` trong front-matter, site VitePress vẫn chạy song song cho đến khi hub đủ dùng.

## ADR-04 — Monorepo?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. Monorepo pnpm (`apps/api`, `apps/web`, `packages/shared`) | Chia sẻ Zod schema/type API; 1 PR cho thay đổi xuyên FE/BE; 1 lần setup lint/CI | CI lâu hơn khi lớn (dùng path filter) |
| B. Nhiều repo (api, web) | Phân quyền tách biệt | Đồng bộ type thủ công; nhiều chi phí cho nhóm nhỏ |

**Đề xuất: A** cho code; **content ở repo riêng** (ADR-05) vì vòng đời, người sở hữu và quy trình review khác code.

## ADR-05 — Nội dung lưu ở PostgreSQL, Git, hay cả hai?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. Chỉ DB (CMS) | Sửa qua UI | Không review bằng PR; khó đóng góp; mất truy vết |
| B. Chỉ Git, đọc trực tiếp lúc request | Đơn giản | Chậm, rate limit GitHub, khó query/tìm kiếm/ràng buộc khóa ngoại |
| C. **Git là nguồn sự thật + sync sang DB** | PR review, lịch sử, mỗi bản publish gắn commit SHA; DB phục vụ truy vấn nhanh, tiến độ, FK | Cần pipeline sync + validation |

**Đề xuất: C.** Tiến độ, bài nộp, review (dữ liệu người dùng) chỉ ở DB. Nội dung (track/module/lesson/exercise) chỉ chỉnh ở Git; DB là bản sao có chỉ mục, lưu `content_version_id` → `commit_sha`. Truy cập repo thông qua interface `ContentSource` (xem CONTENT-SPECIFICATION).

## ADR-06 — Đăng nhập bằng Google, GitHub, hay cả hai?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. GitHub only | Bài nộp qua GitHub cần identity này; thành viên CNTT đều có | Thành viên mới chưa có GitHub phải tạo |
| B. Google only | Quen thuộc, email trường | Phải liên kết GitHub riêng để nộp bài |
| C. **GitHub là chính, liên kết Google/email trường tùy chọn** | Một identity cho bài nộp + review; có thể xác thực thành viên qua org membership | Hai luồng OAuth |

**Đề xuất: C** — MVP chỉ cần GitHub OAuth (hoặc GitHub App user-to-server). Google để Phase 2 cho xác thực `@student.hcmute.edu.vn` nếu CLB muốn. Không lưu mật khẩu. Quyền vào hệ thống = tài khoản GitHub thuộc **org hcmute-dsc** hoặc nằm trong allowlist (xem Open Questions).

## ADR-07 — Chấm bài tự động: GitHub Actions, Judge0, Piston, hay khác?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. **GitHub Actions** — workflow **do CLB sở hữu** (repo `learning-hub-checker`) checkout bài làm tại `head_sha` và chạy test ẩn (**không** chạy workflow nằm trong repo thành viên: họ sửa được để luôn pass — xem CRITIQUE §1.1) | Không tự dựng sandbox; chạy mọi ngôn ngữ trên VM GitHub-hosted tạm; thành viên học PR; an toàn cho server hub; kết quả đáng tin | Chậm hơn (30–120 s); phụ thuộc GitHub; cần GitHub App + webhook; chỉ nhận repo công khai |
| B. Judge0 / Piston (tự host) | Phản hồi nhanh, tích hợp UI | Phải vận hành sandbox (rủi ro bảo mật), tốn tài nguyên, đội sinh viên khó duy trì |
| C. Chạy trong API | — | **Cấm** (không an toàn) |
| D. Quiz/self-check không cần chạy code | Rẻ, cho khái niệm | Không kiểm tra kỹ năng viết code |

**Đề xuất: A + D ở MVP.** B chỉ khi có nhu cầu thật (>~200 bài nộp/tuần hoặc yêu cầu phản hồi tức thời) và khi có người sở hữu `learning-hub-runner` (cách ly hoàn toàn, xem SECURITY).

## ADR-08 — Loại khỏi MVP

Loại: online judge tự dựng; code editor chạy code trong trình duyệt phía server; gamification ngoài streak/huy hiệu đơn giản (Phase 2); bảng xếp hạng công khai; chat realtime (dùng Discord); thông báo push; thanh toán; multi-tenant; Redis/BullMQ (chỉ thêm khi webhook/Discord cần retry – xem ARCHITECTURE); MinIO/upload file (bài nộp là link GitHub); tìm kiếm toàn văn nâng cao (dùng Postgres `tsvector` khi cần); cổng Google OAuth; tích hợp dữ liệu từ repo private.

## ADR-09 — Chiến lược repo GitHub (tóm tắt, chưa tạo)

| Repo | Vai trò | Khi nào tạo |
| --- | --- | --- |
| `learning-hub` | Monorepo code (`apps/api`, `apps/web`, `packages/shared`, `infra/`) | Sau khi duyệt tài liệu |
| `learning-hub-content` | Track/module/lesson/exercise (YAML + Markdown), CI validate | Cùng lúc, import sql-challenge đầu tiên |
| `learning-hub-docs` | Bộ tài liệu này; có thể chuyển vào `learning-hub/docs` nếu muốn một repo | Đã có local |
| `learning-hub-checker` | Workflow + test ẩn chấm bài `code` trên GitHub Actions (ADR-07 A); chỉ Ban kỹ thuật có quyền ghi | Cùng lúc với bài `code` đầu tiên |
| `learning-hub-runner` | Sandbox chạy code riêng (**tương lai**) | Chỉ khi ADR-07 chuyển sang B |
| `learning-hub-exercises-*` (tùy chọn) | Template repo cho exercise cần starter code | Khi có bài `code` |

Chuyển đổi: (1) tạo content repo, import sql-challenge, giữ site cũ; (2) dựng hub; (3) ngừng cập nhật Course-DSC/study-jam-*; (4) cập nhật `.github` profile và website CLB. Không có repo nào bị xóa; chỉ archive sau khi owner đồng ý.
