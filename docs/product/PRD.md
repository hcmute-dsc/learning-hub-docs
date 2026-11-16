# PRD — HCMUTE DSC Learning Hub

> Trạng thái: bản nháp v0.1 (chờ Ban chủ nhiệm duyệt) · Nhãn phạm vi: **[MVP]**, **[P2]** = Phase 2, **[F]** = Future.
> Tên làm việc: "Learning Hub". Cảm hứng: Exercism (học bằng làm bài, có mentor review) nhưng dành cho **một** CLB, không phải SaaS.

## 1. Tầm nhìn

Một nơi duy nhất để thành viên HCMUTE DSC **học bằng cách làm**: chọn lộ trình (track), học bài ngắn, làm bài tập, nộp qua GitHub, nhận phản hồi tự động và nhận xét từ anh/chị mentor. Nội dung do chính thành viên đóng góp bằng Pull Request, và hệ thống đủ đơn giản để các thế hệ Ban chủ nhiệm sau tự bảo trì.

## 2. Vấn đề

| # | Vấn đề hiện tại | Bằng chứng (REPOSITORY-AUDIT) |
| --- | --- | --- |
| P1 | Nội dung học rải rác: VitePress (SQL), React data (HTML/CSS), bản đồ TS (mentoring), Study Jam | 4+ repo, 3 định dạng |
| P2 | Không có nơi theo dõi tiến độ; mentor không biết thành viên đang ở đâu | Study Jam chỉ có task/marking theo sự kiện |
| P3 | Học thụ động: đọc bài xong không có phản hồi | SQL Challenge có đáp án gập, không có chấm |
| P4 | Khó đóng góp: không có chuẩn nội dung/quy trình review | — |
| P5 | Hai stack song song, backend cũ ít bảo trì | study-jam-api, Course-DSC (Java) |
| P6 | Mentoring dựa vào Discord/Facebook, khó truy vết | — (cần xác nhận với CLB) |

## 3. Mục tiêu / Không phải mục tiêu

**Mục tiêu**
- G1: Một catalog track duy nhất, có thứ tự, có điều kiện tiên quyết. [MVP]
- G2: Mọi bài tập có tiêu chí hoàn thành rõ; ít nhất loại `code` có phản hồi tự động qua GitHub Actions. [MVP]
- G3: Mentor review được bài nộp, comment theo dòng, yêu cầu sửa/duyệt. [MVP]
- G4: Nội dung quản lý bằng Git, mỗi bản publish truy vết được tới commit SHA. [MVP]
- G5: Thành viên theo dõi được tiến độ cá nhân; mentor/admin thấy tiến độ nhóm. [MVP]
- G6: Dễ bảo trì: modular monolith, ≤ 3 service chạy (api, web, postgres), setup cục bộ ≤ 15 phút. [MVP]

**Không phải mục tiêu**
- Không phải SaaS/đa tổ chức, không bán khóa học, không thanh toán.
- Không xây online judge riêng ở MVP.
- Không thay Discord (chat, thảo luận) hay site CLB (tin tức).
- Không thay thế mentoring-program (bản đồ) hay Study Jam đang chạy.
- Không gamification nặng (điểm thưởng, leaderboard công khai).

## 4. Người dùng mục tiêu & persona

| Persona | Mô tả | Nhu cầu chính |
| --- | --- | --- |
| **Linh – Newbie (năm 1)** | Chưa biết Git, biết ít Python/C++. Tham gia CLB Gen 7. | Lộ trình rõ ràng, bài nhỏ, không bị ngợp, được giúp khi kẹt. |
| **Minh – Member trung cấp (năm 2–3)** | Biết lập trình, muốn học SQL/Docker/web. | Tự học theo tốc độ, bài tập thực tế, phản hồi nhanh. |
| **Hà – Mentor (năm 3–4)** | Cựu Ban/leader nhóm, ít thời gian. | Hàng đợi review gọn, biết ai cần giúp, không mất công chấm lặp. |
| **Tuấn – Content maintainer** | Thành viên viết bài/bài tập. | Viết Markdown, mở PR, thấy lỗi validate ngay, xem preview. |
| **Khoa – Admin (Ban chủ nhiệm)** | Quản trị CLB, thay đổi theo nhiệm kỳ. | Publish track, phân quyền, xem số liệu, bàn giao dễ. |

## 5. Vai trò & quyền

| Hành động | member | mentor | content_maintainer | admin |
| --- | :-: | :-: | :-: | :-: |
| Xem track/lesson đã publish | ✔ | ✔ | ✔ | ✔ |
| Enroll, đánh dấu tiến độ, nộp bài | ✔ | ✔ | ✔ | ✔ |
| Xem bài nộp của mình | ✔ | ✔ | ✔ | ✔ |
| Xem bài nộp của mentee được giao | — | ✔ | — | ✔ |
| Viết review / comment | — | ✔ | — | ✔ |
| Nhận mentee (self-claim) / được gán | — | ✔ | — | ✔ |
| Gán mentor cho member | — | — | — | ✔ |
| Đồng bộ/publish content version | — | — | ✔ (yêu cầu) | ✔ (duyệt) |
| Duyệt tài khoản `pending` | — | ✔ | — | ✔ |
| Quản lý danh sách email được duyệt (`approved_emails`) | — | — | — | ✔ |
| Quản lý user & vai trò | — | — | — | ✔ |
| Xem audit log | — | — | — | ✔ |

Một người có thể có nhiều vai trò. Chi tiết RBAC: SECURITY.md.

## 6. Tính năng cốt lõi

| # | Tính năng | Phase |
| --- | --- | --- |
| F1 | Đăng nhập GitHub OAuth, hồ sơ cơ bản | MVP |
| F2 | Catalog track + trang chi tiết (module, lesson, exercise) | MVP |
| F3 | Enroll track; tiến độ bài học (lesson progress) | MVP |
| F4 | Exercise loại `code` (nộp PR/commit GitHub công khai, chấm bằng workflow của CLB trong `learning-hub-checker`), `quiz` (chấm trong app), `self_check` (tự đối chiếu), `project` (mentor chấm) | MVP |
| F5 | Submission + TestRun; phản hồi tự động | MVP |
| F6 | Mentor: hàng đợi review, comment, approve / request changes | MVP |
| F7 | Mentor assignment (admin gán hoặc mentor nhận) + yêu cầu mentoring | MVP |
| F8 | Content sync từ Git, validate, publish version theo commit SHA | MVP |
| F9 | Dashboard thành viên / mentor / admin cơ bản | MVP |
| F10 | Thông báo Discord (webhook) khi: có review, có bài mới cần review | MVP (một chiều) |
| F11 | Hint & solution mở khóa sau N lần thử / sau khi nộp | P2 |
| F12 | Huy hiệu & streak nhẹ, có thể tắt | P2 |
| F13 | Notification trong app, email digest | P2 |
| F14 | Đóng góp nội dung qua UI "Edit on GitHub" + preview | P2 |
| F15 | Liên kết Google/email trường | P2 |
| F16 | Bản đồ mentoring đọc từ content repo | P2 |
| F17 | Runner sandbox riêng (Judge0/Piston hoặc runner tự dựng) | F |
| F18 | Cohort/Sprint đầy đủ (lịch, deadline), API công khai (nhãn `enrollments.cohort` có từ MVP) | P2/F |
| F19 | Chứng nhận hoàn thành track do hub tự cấp: trang/link xác minh được, gồm tên thành viên, track, ngày hoàn thành, phiên bản nội dung (`commit_sha`). Không quy đổi điểm rèn luyện | P2 |

## 7. Yêu cầu chức năng (rút gọn)

| ID | Yêu cầu | Phase |
| --- | --- | --- |
| FR-01 | Hệ thống cho phép đăng nhập bằng GitHub; tạo `user` ở lần đầu; đọc mọi email đã xác minh của tài khoản GitHub (`GET /user/emails`); nếu có email khớp danh sách `approved_emails` của CLB (hoặc là admin bootstrap) thì tự `active`, ngược lại ở trạng thái `pending` chờ admin/mentor duyệt; user `pending` khớp danh sách ở lần đăng nhập sau cũng thành `active`. Membership org GitHub chỉ là kiểm tra tùy chọn. | MVP |
| FR-02 | Liệt kê track đã publish, lọc theo level/tag, sắp xếp theo `position`. | MVP |
| FR-03 | Enroll/unenroll track; một user có thể enroll nhiều track. | MVP |
| FR-04 | Exercise có `prerequisites`; exercise bị khóa cho tới khi prerequisite đã hoàn thành. | MVP |
| FR-05 | Lesson hiển thị Markdown, đánh dấu hoàn thành thủ công. | MVP |
| FR-06 | Nộp bài `code` bằng URL PR/commit GitHub; hệ thống nhận kết quả CI qua webhook. | MVP |
| FR-07 | Bài `quiz` chấm tự động trong app; số lần thử không giới hạn. | MVP |
| FR-08 | Một submission có tối đa 1 review "đang mở"; mentor comment theo file/dòng. | MVP |
| FR-09 | Member xem lịch sử mọi lần nộp và kết quả mỗi lần chạy test. | MVP |
| FR-10 | Admin gán/hủy mentor; mentor xem danh sách mentee. | MVP |
| FR-11 | Content sync đọc repo qua `ContentSource`, validate schema, tạo `content_version` ở trạng thái `draft`, admin publish. | MVP |
| FR-12 | Mọi hành động quản trị ghi `audit_logs`. | MVP |
| FR-13 | Xóa mềm user/nội dung; giữ bài nộp (xem DATABASE). | MVP |
| FR-14 | Xuất tiến độ (CSV) cho admin. | P2 |
| FR-15 | Admin nhập/thay danh sách email được duyệt; chỉ admin xem được; mọi thay đổi ghi `audit_logs`; không xuất cho member. | MVP |
| FR-16 | Hoàn thành track → cấp chứng nhận có link xác minh công khai (F19). | P2 |

## 8. Yêu cầu phi chức năng

| Nhóm | Yêu cầu |
| --- | --- |
| Hiệu năng | Trang catalog/lesson p95 < 500 ms với 500 người dùng đồng thời (dữ liệu từ DB, không gọi GitHub lúc render). |
| Quy mô | Thiết kế cho ≤ 1.000 user, ≤ 20 submission/phút đỉnh. Một VM nhỏ đủ. |
| Sẵn sàng | Mục tiêu 99% giờ học (không SLA cứng); sao lưu DB hằng ngày. |
| Bảo mật | Xem SECURITY.md; không chạy code thành viên trong API. |
| Bảo trì | Modular monolith, ranh giới module có lint rule; coverage service domain ≥ 70%; migration tự động. |
| Onboard | `pnpm install && docker compose up` chạy được; README ≤ 1 trang; seed data. |
| Truy cập | WCAG 2.1 AA cho luồng chính; dùng được bằng bàn phím; đáp ứng mobile. |
| i18n | Giao diện tiếng Việt trước, cấu trúc cho i18n (vi/en); nội dung có thể có `lang`. |
| Quan sát | Log JSON (Pino), request id, health check; cảnh báo lỗi qua Discord. |
| Quyền riêng tư | Chỉ lưu dữ liệu cần thiết (GitHub id, tên, avatar, email nếu có); xóa theo yêu cầu. |

## 9. User stories & acceptance criteria (đại diện)

| ID | Story | Acceptance criteria |
| --- | --- | --- |
| US-01 | Là member, tôi muốn đăng nhập bằng GitHub để không phải nhớ mật khẩu. | Given tài khoản GitHub của tôi có một email đã xác minh (chính hoặc phụ) nằm trong danh sách email của CLB, When bấm "Đăng nhập GitHub", Then vào dashboard; tài khoản không có email khớp thấy trang "Đang chờ duyệt" (status `pending`), chưa xem được catalog, và được thông báo khi admin/mentor duyệt. |
| US-02 | Là member, tôi muốn xem danh sách track để chọn lộ trình. | Hiển thị tiêu đề, level, số bài, thời lượng ước tính; chỉ track `published`. |
| US-03 | Là member, tôi muốn enroll vào track để theo dõi tiến độ. | Enroll lần 2 là idempotent (không tạo trùng); dashboard hiện track với % hoàn thành. |
| US-04 | Là member, tôi muốn đọc lesson rồi làm bài tập liên quan. | Lesson có nút "Đánh dấu đã học"; bài tập bị khóa hiển thị lý do (prerequisite nào chưa xong). |
| US-05 | Là member, tôi muốn nộp bài bằng PR để nhận phản hồi tự động. | Dán URL PR → submission `submitted`; trong ≤ 3 phút xuất hiện TestRun (`passed`/`failed`) kèm log tóm tắt; nộp lại tạo submission mới (attempt +1). |
| US-06 | Là member, tôi muốn xin mentor để được hướng dẫn. | Gửi yêu cầu kèm lời nhắn; thấy trạng thái `pending/accepted`; chỉ 1 yêu cầu `pending` cùng lúc. |
| US-07 | Là mentor, tôi muốn xem hàng đợi bài cần review. | Sắp xếp cũ nhất trước; lọc theo track/mentee; bài đã có người review hiển thị người đó. |
| US-08 | Là mentor, tôi muốn comment theo dòng và chọn kết quả. | Chọn `approved` hoặc `changes_requested`; member nhận thông báo Discord; không sửa được review của mentor khác. |
| US-09 | Là content maintainer, tôi muốn thêm lesson bằng PR. | CI content repo fail nếu thiếu `uid`, trùng slug, prerequisite không tồn tại; PR merged → có `content_version` `draft`. |
| US-10 | Là admin, tôi muốn publish version. | Xem diff (thêm/sửa/xóa mục) trước khi publish; publish lưu `commit_sha`, người publish, thời điểm; audit log được ghi. |
| US-11 | Là admin, tôi muốn gán mentor cho member. | Một member tối đa 1 mentor `active` mỗi track; đổi mentor giữ lịch sử. |

## 10. Phạm vi

### Phase 0 — xác thực nhu cầu (trước MVP, xem CRITIQUE §1.2, §6)
Dựng content repo, import SQL Challenge, chạy một đợt với ≥ 20 thành viên bằng công cụ có sẵn; chỉ xây app khi đạt tiêu chí đi/không đi.
**Đã chốt (16/11/2026): có chạy Phase 0.** Khung app (E1–E3) đã dựng từ trước; trong Phase 0 chưa thêm tính năng app mới.

### MVP (8–12 tuần, nhóm 3–5 sinh viên bán thời gian; chia **MVP-Lite** trước, **MVP-Full** sau — xem MVP-BACKLOG)
F1–F10, FR-01…FR-13. Nội dung đầu tiên: track **SQL 30 ngày** (import) + một track **Git & GitHub** (viết mới từ Study Jam) làm bài `code` mẫu. Mục tiêu hạ rào cản: track "làm quen GitHub" dùng quiz + `self_check` trước khi nộp PR.

### Phase 2
F11–F16, F19 (chứng nhận hoàn thành track), xuất CSV, thông báo in-app/email, Redis + BullMQ nếu cần retry/queue, MinIO nếu cho upload file.

### Future
F17–F18, runner sandbox, cohort, tích hợp dữ liệu từ `HCMUTE-DSC-be`/`management-service` nếu được xác nhận. Quy đổi chứng nhận sang điểm rèn luyện của trường: **ngoài phạm vi** (chưa xác nhận).

## 11. Chỉ số thành công

| Chỉ số | Mục tiêu sau 1 học kỳ |
| --- | --- |
| Thành viên active hằng tuần / tổng thành viên | 15–25% (đặt lại sau khi đo baseline ở Phase 0) |
| Tỷ lệ enroll → hoàn thành ≥ 1 exercise | ≥ 50% |
| Tỷ lệ hoàn thành track SQL 30 ngày | 10–15% (học tự nguyện thường dưới 15%) |
| Thời gian trung vị bài nộp → có review | ≤ 72 giờ mặc định (theo dõi, không cam kết; mục tiêu chốt theo học kỳ qua cấu hình `REVIEW_TARGET_HOURS`) |
| Số bài/lesson do thành viên (không phải Ban) đóng góp | ≥ 10 |
| Thời gian onboard dev mới (clone → chạy) | ≤ 30 phút |
| Số mentor active | ≥ 5 |

## 12. Rủi ro & giả định

| ID | Rủi ro / giả định | Giảm thiểu |
| --- | --- | --- |
| R1 | Nhóm dev sinh viên rời đi cuối nhiệm kỳ | Tài liệu, ADR, tránh công nghệ lạ, README onboarding |
| R2 | Thiếu mentor → bài chờ review lâu | Hàng đợi, mentor luân phiên, review ngang hàng (P2); cam kết mentor (số mentor, giới hạn mentee, thời gian review) chốt mỗi học kỳ và cấu hình được |
| R3 | Phụ thuộc GitHub (rate limit/Actions/outage) | Cache nội dung vào DB; nộp bài chịu được trễ; retry |
| R4 | Nội dung không đủ → nền tảng trống | Import SQL Challenge ngay MVP; ưu tiên 2 track |
| R5 | Thành viên chưa có GitHub / chưa quen PR | Track Git & GitHub là bài đầu; hỗ trợ bằng mentor |
| R6 | Chi phí hosting | 1 VPS do Chủ nhiệm CLB tài trợ + Docker Compose (Postgres trong compose); tận dụng Actions free tier |
| R7 | Bản quyền nội dung import (it-quiz, html-css) | Giữ giấy phép gốc; người import xác nhận nguồn/tác giả trong mô tả PR (CONTENT-SPECIFICATION §10) |
| R8 | Tài khoản/hạ tầng do một người giữ (Chủ nhiệm đương nhiệm) | Runbook bàn giao liệt kê mọi tài khoản/secret theo chức danh (MVP-BACKLOG E11) |
| R9 | Danh sách email được duyệt là dữ liệu cá nhân | Chỉ admin xem/sửa; thay đổi ghi audit; không xuất cho member (SECURITY §1) |
| A1 | Giả định: thành viên có email nằm trong danh sách email của CLB và thêm được email đó (đã xác minh) vào GitHub | Người không khớp vẫn vào được qua duyệt tay (`pending` → `active`) |
| A2 | Giả định: ≤ 1.000 user | Nếu lớn hơn, xem lại ADR-07 |
