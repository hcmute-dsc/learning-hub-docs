# PRD — HCMUTE DSC Learning Hub

> Trạng thái: bản nháp v0.1 (chờ Ban chủ nhiệm duyệt) · Nhãn phạm vi: **[MVP]** (MVP-Lite trước, **MVP-Full** sau), **[P2]** = Phase 2, **[F]** = Future. ADR đã chốt 16/11/2026 (DECISIONS).
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
- G6: Dễ bảo trì: modular monolith, ≤ 3 service chạy ở MVP-Lite (api, web, postgres; MVP-Full thêm `sql-checker` + PostgreSQL riêng, ADR-07), setup cục bộ ≤ 15 phút. [MVP]

**Không phải mục tiêu**
- Không phải SaaS/đa tổ chức, không bán khóa học, không thanh toán.
- Không xây online judge nhiều ngôn ngữ (Judge0/Piston) ở MVP; chỉ có checker riêng cho SQL ở MVP-Full (ADR-07).
- Không thay Discord (chat, thảo luận) hay site CLB (tin tức).
- Không thay thế mentoring-program (bản đồ) hay Study Jam đang chạy.
- Không gamification nặng (điểm thưởng, xếp hạng toàn thời gian hoặc theo tốc độ). Bảng xếp hạng chỉ có dạng opt-in theo học kỳ (F21, ADR-08).

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
| F12 | Huy hiệu & streak nhẹ, có thể tắt (quy tắc ở §6.1) | P2 |
| F13 | Notification trong app, email digest | P2 |
| F14 | Đóng góp nội dung qua UI "Edit on GitHub" + preview | P2 |
| F15 | Liên kết Google/email trường | P2 |
| F16 | Bản đồ mentoring đọc từ content repo | P2 |
| F17 | Runner sandbox nhiều ngôn ngữ (Judge0/Piston hoặc runner tự dựng), trên VPS riêng — không chung VPS với hub | F |
| F18 | Cohort/Sprint đầy đủ (lịch, deadline), API công khai (nhãn `enrollments.cohort` có từ MVP) | P2/F |
| F19 | Chứng nhận hoàn thành track do hub tự cấp: trang/link xác minh được, gồm tên thành viên, track, ngày hoàn thành, phiên bản nội dung (`commit_sha`). Không quy đổi điểm rèn luyện | P2 |
| F20 | Chấm SQL tự động bằng `sql-checker` trên VPS (PostgreSQL riêng, dataset DSC Shop): nộp câu truy vấn trong app, so với tập kết quả kỳ vọng; chuyển dần 31 bài `self_check` của `sql-30-days` sang chấm tự động (ADR-07) | MVP-Full |
| F21 | Bảng xếp hạng opt-in theo học kỳ/đợt: reset mỗi kỳ, xếp theo số bài hoàn thành (`self_check` trọng số thấp hơn), không theo tốc độ, top 10 + hạng của mình, không xếp hạng toàn thời gian (ADR-08) | MVP-Full |
| F22 | Thông báo Web Push trên trình duyệt (bài được review, yêu cầu review mới cho mentor, track mới publish); Discord là kênh thứ hai; iOS cần "Thêm vào Màn hình chính" | MVP-Full |

### 6.1 F12 — Huy hiệu và streak (chốt 18/11/2026, learning-hub#48)

Động lực nhẹ, không điểm, không xếp hạng (§3 "Không gamification nặng").

**Bộ huy hiệu đầu tiên** (định nghĩa trong code: `packages/shared/src/achievements.ts`, quy tắc ở `apps/api/src/modules/learning/badges.ts`):

| Mã | Tên | Điều kiện |
| --- | --- | --- |
| `first_lesson` | Bài học đầu tiên | hoàn thành ≥ 1 bài học |
| `first_exercise` | Bài tập đầu tiên | hoàn thành ≥ 1 bài tập (`completed`: qua chấm tự động hoặc mentor duyệt) |
| `first_module` | Xong một chặng | mọi bài học và bài tập của một module đều xong |
| `first_track` | Về đích | hoàn thành trọn một track |
| `exercises_10` | Mười bài tập | hoàn thành ≥ 10 bài tập |
| `streak_7` | Bền bỉ 7 ngày | streak dài nhất ≥ 7 |

**Cấp huy hiệu**
- Cấp sau các sự kiện học tập có sẵn: hoàn thành bài học, bài tập `passed`, project được duyệt. Hoàn thành module/track luôn đi kèm một trong các sự kiện đó.
- Idempotent: khóa `(user_id, badge)` trong `user_badges`; cấp lại không tạo trùng, sự kiện `learning.badge_awarded` chỉ phát một lần mỗi huy hiệu (để F13 thông báo sau này).
- Lưới an toàn: khi đọc dashboard/hồ sơ, hệ thống cấp nốt huy hiệu còn thiếu, nên thành viên học trước khi có tính năng vẫn nhận đủ.
- Đã cấp thì không thu hồi, kể cả khi nội dung thay đổi hoặc mở lại bài học.
- Thêm huy hiệu: thêm vào hai file trên kèm test; không đổi tên mã đã có (mã được lưu trong DB).

**Streak (chuỗi ngày học)**
- Ngày tính theo giờ Việt Nam (UTC+7); một ngày "có học" khi có ít nhất một bài học hoàn thành hoặc một lần nộp bài.
- Không phạt nặng: **được nghỉ xen kẽ 1 ngày**. Hai ngày có học cách nhau tối đa 2 ngày vẫn chung một chuỗi; nghỉ **2 ngày liền** thì chuỗi kết thúc.
- Độ dài chuỗi = số ngày có học trong chuỗi (không tính ngày nghỉ).
- Chuỗi hiện tại còn sống khi ngày học gần nhất là hôm nay, hôm qua hoặc hôm kia (hôm qua là ngày được nghỉ, học hôm nay là nối tiếp). Hồ sơ hiện thêm chuỗi dài nhất từng có.
- Không lưu streak trong DB: tính lại từ `lesson_progress.completed_at` và `submissions.submitted_at`.

**Tắt được**
- `/settings` có công tắc "Hiện huy hiệu và chuỗi ngày học" (`users.show_achievements`, mặc định bật).
- Tắt thì ẩn ở **mọi nơi**: thẻ "Thành tích" trên dashboard, ô streak và mục huy hiệu trên `/u/[login]`; API trả `achievements: null`, kể cả khi người khác xem hồ sơ.
- Khi tắt, huy hiệu vẫn được cấp ngầm; bật lại là thấy đủ.

### 6.2 Quy tắc chứng nhận hoàn thành track (F19, learning-hub#47)

- **Điều kiện hoàn thành:** mọi lesson và exercise **đang active** của track đã xong — lesson có `lesson_progress.status = completed`, exercise có submission `approved`, hoặc `passed` khi `requires_review = false` (DATABASE §3.11, §6). Track không có mục nào thì không bao giờ "hoàn thành". Quy tắc nằm ở **một chỗ** trong code (`apps/api/src/modules/learning/progress.ts`, `isAllDone`); lần đầu đủ điều kiện, enrollment được lưu `completed` + `completed_at` và phát sự kiện `learning.track_completed` đúng một lần. Chứng nhận, huy hiệu… chỉ **nghe** sự kiện này, không tự tính lại.
- **Cấp một lần:** mỗi (thành viên, track) tối đa một chứng nhận. Nội dung track đổi sau đó không cấp lại; chứng nhận đã thu hồi cũng không cấp lại khi thành viên "hoàn thành lại".
- **Nội dung chứng nhận:** tên hiển thị **lúc cấp**, tên track, ngày hoàn thành (`completed_at`), ngày cấp, repo + commit nội dung members đang đọc lúc cấp. Không có email, không có id tài khoản.
- **Mã:** 24 ký tự base32 chữ thường (120 bit ngẫu nhiên), là phần không đoán được của link `/certificates/[code]`.
- **Trang công khai** `/certificates/[code]`: ai có link cũng xem được, không cần đăng nhập; `noindex`; in hoặc lưu PDF bằng hộp thoại in của trình duyệt (CSS print, A4 ngang). Chứng nhận bị thu hồi vẫn mở được nhưng ghi rõ "đã bị thu hồi" (không ghi lý do).
- **Hồ sơ:** `/u/[login]` (chỉ thành viên đã đăng nhập) hiện các chứng nhận còn hiệu lực mà chủ nhân không ẩn. Thành viên ẩn/hiện từng chứng nhận ở `/settings`; ẩn khỏi hồ sơ **không** khóa link đã chia sẻ.
- **Thu hồi:** chỉ admin, ở tab Quản trị → Chứng nhận, bắt buộc ghi lý do (3–280 ký tự); ghi `audit_logs` (`certificate.revoke`). Không có thao tác "khôi phục" trong giao diện.
- **Hoàn thành trước khi có tính năng:** enrollment đã `completed` mà chưa có chứng nhận được cấp bù khi chủ nhân mở `/settings` hoặc hồ sơ của mình (commit nội dung là bản đang publish lúc cấp bù).

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
| FR-16 | Hoàn thành track → cấp chứng nhận một lần, có link xác minh công khai không cần đăng nhập; thành viên ẩn khỏi hồ sơ, admin thu hồi có audit (F19, quy tắc §6.2). | P2 |
| FR-17 | Bài SQL chấm bằng `sql-checker`: truy vấn chạy read-only với timeout, kết quả ghi `test_runs`; so có thứ tự khi đề yêu cầu `ORDER BY` (F20). | MVP-Full |
| FR-18 | Bảng xếp hạng chỉ gồm thành viên đã opt-in (`users.leaderboard_opt_in`), tính bằng truy vấn/view theo kỳ, không lưu điểm (F21). | MVP-Full |
| FR-19 | Thành viên bật/tắt Web Push theo từng thiết bị; subscription hết hạn (404/410) tự xóa (F22). | MVP-Full |

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
F1–F10, FR-01…FR-13; **MVP-Full** thêm F20–F22 (checker SQL ngay sau MVP-Lite, bảng xếp hạng opt-in, Web Push — quyết định 16/11/2026, DECISIONS ADR-07/ADR-08). Nội dung đầu tiên: track **SQL 30 ngày** (import) + một track **Git & GitHub** (viết mới từ Study Jam) làm bài `code` mẫu. Mục tiêu hạ rào cản: track "làm quen GitHub" dùng quiz + `self_check` trước khi nộp PR.

### Phase 2
F11–F16, F19 (chứng nhận hoàn thành track), xuất CSV, thông báo in-app/email, Redis + BullMQ nếu cần retry/queue, MinIO nếu cho upload file.

### Future
F17–F18, runner sandbox nhiều ngôn ngữ (VPS riêng), cohort, tích hợp dữ liệu từ `HCMUTE-DSC-be`/`management-service` nếu được xác nhận. Quy đổi chứng nhận sang điểm rèn luyện của trường: **ngoài phạm vi** (chưa xác nhận).

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
