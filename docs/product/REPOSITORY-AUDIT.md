# REPOSITORY-AUDIT — Kiểm kê hệ sinh thái HCMUTE DSC

> Trạng thái: bản nháp v0.1 · Ngày audit: 2026-11-16 · Phương pháp: GitHub API (`gh api`) trên các repo **public** của org `hcmute-dsc`, cộng với trang https://hcmute-dsc.io.vn/.
> Chưa clone, chưa chạy code, chưa đọc repo private.

## 1. Quy ước mức độ chắc chắn

| Nhãn | Ý nghĩa |
| --- | --- |
| **Verified** | Đã đọc trực tiếp file/metadata trong repo. |
| **Partially verified** | Đã thấy cấu trúc/tên file nhưng chưa đọc nội dung đủ sâu. |
| **Unknown** | Không truy cập được hoặc chưa kiểm tra. |
| **Requires human confirmation** | Cần người trong CLB xác nhận (vận hành thật, chủ sở hữu, kế hoạch). |

## 2. Tóm tắt hệ sinh thái

- Org có 19 repo public + 4 repo private (1 đã archive). Mô tả org: "HCMUTE Developer Student Club (formerly GDSC - HCMUTE)". **Verified**
- Website CLB https://hcmute-dsc.io.vn/ liệt kê 5 dự án: Memoryland, SQL Challenge, Mentoring, IT Quiz, Club Day; dòng thời gian: Study Jam Git & GitHub (2024, 2025), SQL Challenge (2025, 2026), InterviewPrep101, HCMUTE CTF, tuyển Gen 7. **Verified** (đọc trang chủ)
- Có **hai nền tảng công nghệ song song**: Java/Spring Boot (study-jam-api, Course-DSC) và TypeScript/React/Next (hầu hết còn lại). Nội dung học đang nằm **rải rác** (VitePress, React data file, bản đồ TS) và **chưa có hệ thống theo dõi tiến độ/bài nộp** dùng chung.
- Hoạt động gần đây tập trung vào các site tĩnh (mentoring-program, sql-challenge, memoryland, ClubDay-2026, it-quiz); các backend Java không được cập nhật từ đầu 2025.

## 3. Bảng audit repo (7 repo bắt buộc)

| Repo | Stack / nội dung (đã thấy) | Hoạt động cuối | Mức chắc chắn | Đề xuất |
| --- | --- | --- | --- | --- |
| [study-jam-api](https://github.com/hcmute-dsc/study-jam-api) | Spring Boot 3.3.1, Java 17, JPA, Spring Security + JWT (jjwt), Liquibase (4 changelog), PostgreSQL, springdoc-openapi, MapStruct, JUnit 5 (4 file test service). CI Maven trên PR; deploy AWS ECS/ECR khi push `main`. Entity: `User` (SUPER_ADMIN/ADMIN/PARTICIPANT, email+password), `Event`, `Task`. API: `/auth`, `/users`, `/tasks`. Có `docs/` về AWS, Liquibase, PostgreSQL EC2. | 2025-02-21 | Verified (cấu trúc, entity, pom, workflow); Partially verified (logic service, chất lượng test) | **INTEGRATE (tham chiếu) → DEPRECATE dần.** Không mở rộng: domain là "event + task" cho Study Jam, không có lesson/submission/progress. Giữ lại ý tưởng: Liquibase kỷ luật migration, vai trò, tài liệu deploy. Cần xác nhận còn chạy production không. |
| [study-jam-ui](https://github.com/hcmute-dsc/study-jam-ui) | Next.js 14 App Router, Ant Design 5, react-hook-form + Zod, Tailwind, Yarn. Trang: sign-in, users, marking, ranking. Gọi backend qua `BASE_BACKEND_URL`; session util + middleware. Không CI. | 2024-09-17 | Verified (cấu trúc, package.json); Partially verified (code) | **KEEP** nguyên trạng như công cụ sự kiện cũ (không đầu tư thêm). Tái sử dụng mẫu: Zod + server actions + middleware auth. Không dùng lại Ant Design cho Learning Hub (xem FRONTEND-DESIGN). |
| [mentoring-program](https://github.com/hcmute-dsc/mentoring-program) | React + TypeScript + Vite; bản đồ tương tác có 2 sơ đồ (Study Jam: track Git/GitHub, Docker; Mentoring Program), vi/en i18n, dark/light, changelog, deploy GitHub Pages khi push `main`. Nội dung nằm trong `src/diagrams/*.ts`. README rõ ràng. | 2026-11-01 | Verified | **KEEP + INTEGRATE.** Giữ độc lập ở MVP; Learning Hub liên kết sang. Phase 2: lộ trình mentoring chuyển thành YAML trong content repo để bản đồ và hub dùng chung. Quy trình mentoring (ghép cặp, review) trở thành module `mentorship` trong hub. |
| [Course-DSC](https://github.com/hcmute-dsc/Course-DSC) | Spring Boot (Java 17), OAuth2 client + resource server, mail, Liquibase, PostgreSQL. Chỉ 1 bảng `users` (migration `01-create-users-table`), `AuthController`, `UserController`, 3 file test. Không README (404). | 2025-04-30 | Verified (cây file, pom); Partially verified (code) | **DEPRECATE.** Mới là khung (auth + user), chưa có domain khóa học. Không có gì cần migrate. Cần xác nhận chủ repo trước khi archive. |
| [sql-challenge](https://github.com/hcmute-dsc/sql-challenge) | VitePress; 30 bài Markdown tiếng Việt + 3 bài ôn tập + tổng kết; mỗi bài có mục tiêu, "cần biết trước", ví dụ, bài tập, đáp án gập; PostgreSQL 16 qua `docker-compose` (+ Adminer), dataset "DSC Shop" (`setup/init/01_schema.sql`, `02_seed.sql`), sơ đồ SVG, MIT, deploy GitHub Pages, có `review/2026-10-17-course-review.md`. | 2026-10-17 | Verified | **INTEGRATE — nguồn nội dung đầu tiên.** Import thành `tracks/sql-30-days` trong content repo (giữ nguyên Markdown, bổ sung front-matter). Giữ site VitePress tới khi hub thay thế; không viết lại. |
| [html-css](https://github.com/hcmute-dsc/html-css) | React 19 + Vite + Tailwind 4 + react-router 7; `src/data/exercises.json`, component `ExerciseList`/`ExerciseDetail`. README là template mặc định. | 2025-04-16 | Verified (cấu trúc, package.json); **Partially verified** (chưa đọc `exercises.json`) | **INTEGRATE dữ liệu**, DEPRECATE app sau khi import. Cần đọc `exercises.json` để ánh xạ sang schema Exercise. |
| [.github](https://github.com/hcmute-dsc/.github) | Profile README còn nhận diện GDSC cũ (icons8, GDSC-HCMUTE counter, link gdsc.community.dev). | 2024-10-07 | Verified | **REFACTOR:** cập nhật README profile theo thương hiệu HCMUTE DSC, trỏ về hub + site. Việc nhỏ, độc lập. |

## 4. Repo khác (kiểm tra mức nông)

| Repo | Ghi chú | Mức chắc chắn | Đề xuất |
| --- | --- | --- | --- |
| hcmute-dsc.github.io | Site CLB (Docusaurus, blog/news, i18n). Là website https://hcmute-dsc.io.vn. | Partially verified | **KEEP**; thêm link "Học" → hub. Kênh tin tức vẫn ở đây. |
| it-quiz | TypeScript, ôn IT trắc nghiệm, chạy offline (theo trang chủ). | Unknown (chưa đọc repo) | **INTEGRATE** (ứng viên): ngân hàng câu hỏi → loại exercise `quiz`. Cần xác nhận định dạng dữ liệu. |
| memoryland, ClubDay-2026, ClubDay2026, oggy-adventure, minigame-samsung, old.hcmute-dsc.github.io | Sự kiện/kỷ niệm/minigame/bản site cũ. | Unknown | **KEEP** độc lập; ngoài phạm vi hub. |
| Web-CV-boots-AI-fe, AI-CheckFace-Gemini-apiFPT, Tool-merge-mail, dsc-website-template | Dự án phụ/legacy. | Unknown | Không liên quan; có thể là tài liệu học "dự án mẫu". |
| management-service (private, Python) | Mô tả trống. | **Unknown / Requires human confirmation** | Có thể chứa dữ liệu thành viên; hỏi chủ repo trước khi thiết kế identity. |
| HCMUTE-DSC-be (private, Python) | Backend site CLB: sự kiện, dự án, hồ sơ thành viên, tài liệu. | **Unknown / Requires human confirmation** | Có thể là nguồn "member profile"; chưa tích hợp ở MVP. |
| central-management (private, archived) | Cổng quản lý thành viên/sự kiện (TypeScript). | Unknown | Tham khảo ý tưởng; không dùng lại. |
| Web-CV-boots-AI-be, Assets (private) | — | Unknown | Ngoài phạm vi. |

## 5. Phân tích theo 13 tiêu chí (tổng hợp)

| Tiêu chí | Kết quả |
| --- | --- |
| README/tài liệu | Tốt: mentoring-program, sql-challenge, study-jam-api. Thiếu: Course-DSC, html-css. |
| Ngôn ngữ/framework | Java 17 + Spring Boot (2 repo), React/Next/Vite/VitePress (5 repo). Không có NestJS ở đâu → stack mới không "lệch" thêm một hệ ngoài TS. |
| Kiến trúc | study-jam-api: layered (controller → service → repository, DTO + MapStruct). Site tĩnh: data-in-code. |
| DB schema/migration | Liquibase (YAML/XML) ở 2 repo Java; sql-challenge có schema mẫu DSC Shop (dùng để dạy, **không phải** schema hub). |
| Business features hiện có | Event/Task/Participant (Study Jam), user + role. **Chưa có**: lesson, exercise, submission, progress, review. |
| Auth | Email+password + JWT (study-jam-api), OAuth2 client (Course-DSC, **chưa kiểm tra có hoạt động**). Không có GitHub OAuth. |
| API contract | OpenAPI qua springdoc (study-jam-api). |
| Testing | JUnit service tests (Java). Frontend: **không thấy test**. Partially verified. |
| CI/CD | Maven CI + AWS ECS (study-jam-api); GitHub Pages (mentoring-program, sql-challenge). Đây là mẫu CI để hub tái dùng. |
| Chất lượng code | Chưa review sâu. Unknown. |
| Thành phần tái dùng | Nội dung Markdown SQL (rất giá trị), exercise.json, mô hình role, mẫu deploy Pages, UI bản đồ mentoring. |
| Thiếu / nợ kỹ thuật | Hai stack song song; backend Java không bảo trì từ 2025; không có định dạng nội dung chuẩn; không có tracking tiến độ. |

## 6. Việc cần người xác nhận (đầu vào cho Open Questions)

1. study-jam-api / study-jam-ui còn chạy production (AWS) không? Chi phí hiện tại?
2. Ai sở hữu `management-service`, `HCMUTE-DSC-be`? Có dữ liệu thành viên cần tái sử dụng?
3. Course-DSC có kế hoạch tiếp tục không?
4. `exercises.json` (html-css) và dữ liệu it-quiz có bản quyền/nguồn gốc rõ ràng để import? (Quy trình đã chốt: người import xác nhận nguồn/tác giả trong mô tả PR; nguồn cụ thể vẫn cần xác nhận.)
5. Giấy phép nội dung chung cho content repo (sql-challenge đang MIT). → **Đã chốt 16/11/2026** (CONTENT-SPECIFICATION §10).
