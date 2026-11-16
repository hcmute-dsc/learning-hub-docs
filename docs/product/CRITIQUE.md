# CRITIQUE — Phản biện thiết kế cho bối cảnh CLB học thuật

> Mục đích: tự kiểm tra bộ tài liệu xem có **thực sự phù hợp một CLB sinh viên** hay chỉ là một thiết kế "đẹp trên giấy". Mỗi mục có: Nhận định → Mức độ → Hành động (đã áp dụng vào tài liệu nào / còn là câu hỏi mở).
> Mức độ: 🔴 có thể làm dự án thất bại hoặc sai về an toàn · 🟠 rủi ro đáng kể · 🟡 cải thiện.

## 1. Lỗi thiết kế nghiêm trọng đã tìm ra

### 1.1 🔴 CI chạy trong repo của thành viên có thể bị làm giả kết quả
**Vấn đề:** bản nháp đầu cho thành viên tạo repo từ template, workflow test nằm trong repo đó, hub tin kết quả `workflow_run`. Thành viên sở hữu repo nên **sửa workflow thành luôn pass** (hoặc xóa test) được.
**Hành động (đã sửa):** mô hình **repo chấm bài do CLB sở hữu** (`learning-hub-checker`): GitHub App của hub kích hoạt workflow trong repo của CLB, workflow checkout bài làm của thành viên tại đúng `head_sha`, chạy **test ẩn do CLB giữ**, rồi báo kết quả. Kết quả chỉ tin từ repo của CLB. Xem DECISIONS ADR-07, ARCHITECTURE §7, USER-FLOWS §6, SECURITY §7. Chạy code thành viên trên runner GitHub-hosted (VM tạm, không secret) — chấp nhận được cho MVP, **không** thay thế sandbox riêng nếu sau này cần quyền cao hơn.
Hệ quả: bài `code` bắt buộc **repo công khai** (tránh tốn quota, tránh cấp quyền đọc repo private).

### 1.2 🔴 Rủi ro lớn nhất không phải kỹ thuật mà là nội dung + mentor
Exercism sống nhờ hàng nghìn tình nguyện viên; CLB có thể chỉ có 3–8 mentor và vài người viết bài. Nếu hub ra mắt mà (a) ít hơn ~2 track chất lượng, (b) review chờ > 3 ngày, thì thành viên bỏ.
**Hành động:** thêm **Phase 0 (xác thực nhu cầu)** ở MVP-BACKLOG: trước khi viết dòng code nào, dựng `learning-hub-content` + import SQL Challenge + chạy thử một đợt (cohort) bằng công cụ có sẵn (site VitePress + GitHub PR/Issue + kênh Discord). Chỉ xây app khi đạt tiêu chí "đi/không đi" (xem §6). Mục tiêu SLA review hạ từ 48 giờ → **72 giờ** làm chỉ số theo dõi, không cam kết.

## 2. Độ phức tạp so với năng lực đội sinh viên

### 2.1 🟠 Quá nhiều thứ cho nhóm 3–5 sinh viên bán thời gian
Bản nháp có 11 module Nest, 20 bảng, GitHub App, Discord, content sync, checker repo. Một học kỳ khó làm hết.
**Hành động:** MVP chia hai mức trong MVP-BACKLOG:
- **MVP-Lite (≈ 6–8 tuần):** đăng nhập GitHub, catalog đọc từ Git (sync thủ công bằng lệnh/endpoint admin), enroll + lesson progress, nộp bài `quiz` và `self_check`, **một** loại bài `code` qua checker repo, review của mentor. Không Discord bot, không outbox (gửi webhook Discord đơn giản, lỗi chỉ log).
- **MVP-Full:** phần còn lại (mentorship UI đầy đủ, dashboard admin, reconcile job, audit UI).
Gộp module khi cài đặt để giảm ma sát: `catalog + content + learning` có thể ở cùng một Nest module "learning" ban đầu (giữ **thư mục con** theo ranh giới để tách sau). Quy tắc ranh giới vẫn giữ, chỉ giảm số `*.module.ts`.

### 2.2 🟠 Bus factor và kế nhiệm
Sinh viên tốt nghiệp sau 2–3 năm; stack gồm NestJS, Next.js, Prisma, Docker, GitHub App, Discord, VM. Ít người nắm hết.
**Hành động:**
- Thêm tài liệu bàn giao bắt buộc (SECURITY/ARCHITECTURE đã nhắc; MVP-BACKLOG thêm epic "Vận hành & bàn giao": runbook, danh sách tài khoản/secret, checklist đổi người giữ).
- Tài khoản cloud, domain, GitHub App, Discord webhook thuộc **chức danh** (Chủ nhiệm/Trưởng ban kỹ thuật), không thuộc cá nhân; secret lưu trong trình quản lý chung (Bitwarden của CLB) — Open Question.
- Chọn công nghệ **phổ biến, dễ tìm tài liệu**; từ chối thêm thư viện hiếm. Mỗi quý "dependency day" cập nhật gói.
- Cân nhắc hosting **managed** (Render/Fly/Railway + Postgres managed) nếu chi phí chấp nhận, thay vì tự quản VM — Open Question chi phí.

### 2.3 🟠 Có nên tự xây? So sánh với công cụ sẵn có
| Lựa chọn | Hợp lý khi | Vì sao chưa chọn |
| --- | --- | --- |
| **Không xây gì mới**, dùng GitHub Classroom + site tĩnh + Discord | Muốn chạy ngay, ngân sách 0 | Không có tiến độ chung, không có hàng đợi review; vẫn là bước tốt cho Phase 0 |
| **Exercism tự host** (mã nguồn mở) | Học lập trình theo ngôn ngữ chuẩn | Rails + nhiều dịch vụ, nặng vận hành, nội dung tiếng Anh, không hợp SQL/Docker/Git tiếng Việt |
| **Moodle/Open edX** | Có giáo viên, chấm điểm, chứng nhận | Giao diện "LMS công ty", nặng, ít hợp văn hóa PR/GitHub |
| **CTFd** | Cuộc thi kiểu CTF | Không có lộ trình/mentor review |
| **Tự xây (Learning Hub)** | Cần: GitHub-first + mentor review + nội dung Git + tiếng Việt | Cái giá: bảo trì — nên giảm phạm vi (§2.1) |
Kết luận: tự xây **hợp lý nếu** đi qua Phase 0 và giữ MVP-Lite; không hợp lý nếu mục tiêu chỉ "có chỗ đọc bài".

## 3. Phù hợp với người dùng sinh viên

### 3.1 🟠 Rào cản GitHub với người mới
Newbie (Linh) chưa biết Git, nhưng bài đầu của track Git cần nộp bằng PR → nghịch lý "gà và trứng".
**Hành động:**
- Thêm loại bài **`self_check`** (tự kiểm tra có checklist/đáp án gập, không chấm) và quiz làm lối vào; **Track "Làm quen GitHub"** dùng quiz + `self_check` + một bài `code` rất nhỏ có hướng dẫn từng bước (có video/ảnh).
- Cho phép **"nộp dạng liên kết"** cho bài `project`: chỉ cần URL repo/PR, mentor review thủ công (không CI).
- Onboarding giải thích GitHub OAuth bằng 3 câu, tránh yêu cầu quyền thừa.

### 3.2 🟠 Xác thực thành viên bằng org GitHub có ma sát
Thành viên mới chưa vào org → đăng nhập thất bại, Ban phải mời tay.
**Hành động:** thay allowlist "org hoặc danh sách" bằng **luồng đăng ký có duyệt**: đăng nhập GitHub lần đầu → trạng thái `pending` → admin/mentor duyệt (hoặc tự động nếu email trường `@student.hcmute.edu.vn` đã xác minh ở Phase 2). Thêm `users.status = 'pending'`. Quyết định cuối: Open Question (CLB mở cho toàn trường hay chỉ thành viên?).

### 3.3 🟡 Học theo "đợt" (cohort/sprint), không chỉ tự học tự do
CLB quen chạy theo đợt (SQL Challenge 30 ngày, Study Jam, Gen mới): cả nhóm cùng nhịp, có deadline mềm, có buổi gặp. Bản nháp xếp cohort vào "Future".
**Hành động:** thêm cột nhẹ `enrollments.cohort` (text, null) ở MVP để gắn nhãn đợt ("sql-2027-spring") và lọc tiến độ theo đợt; cohort đầy đủ (lịch, deadline) ở P2.

### 3.4 🟡 Văn hóa "học, không chấm điểm"
Cho CLB học thuật, tiến độ nên khuyến khích chứ không xếp hạng. Giữ: không leaderboard; huy hiệu tắt được; hiển thị **tiến độ cá nhân** và "đang học cùng bạn" (số người cùng đợt) thay vì xếp hạng. P2: xem lời giải cộng đồng **sau khi tự hoàn thành** (giống Exercism) để học lẫn nhau.

### 3.5 🟡 Phạm vi chủ đề
CLB gồm cả CTF, AI, phỏng vấn (InterviewPrep101), không chỉ lập trình web. Schema nội dung đã không phụ thuộc ngôn ngữ; nhưng loại bài hiện có thiên về code. Để cho track "phỏng vấn/soft skill" dùng được: `project` + `self_check` + `quiz` đã đủ. Không thêm loại mới ở MVP.

## 4. Mentor & vận hành cộng đồng

### 4.1 🟠 Tải công việc của mentor
- Review PR mất 15–30 phút; 5 mentor × 3 bài/tuần ≈ 15 bài/tuần — đó là trần thực tế.
- **Hành động:** tự động hóa tối đa trước review (CI + checklist); chỉ cần review khi bài `project` hoặc khi member bấm "xin review" cho bài `code` đã `passed`; **giới hạn số mentee/mentor** (mặc định 5, cấu hình được); hàng đợi hiển thị "bài chờ lâu nhất"; P2: review ngang hàng (peer review) có hướng dẫn.
- Ghi nhận mentor (huy hiệu, chứng nhận hoạt động) như phần thưởng chính — đừng để mentor quá tải mà không được thấy giá trị.

### 4.2 🟡 Mentor là người chịu trách nhiệm?
Phân định rõ: mentor góp ý, **không** có trách nhiệm học thuật chính thức. Điều khoản ngắn trong onboarding (Open Question cho Ban chủ nhiệm).

## 5. An toàn, pháp lý, riêng tư (bối cảnh trường học)

| Mối quan tâm | Nhận định | Hành động |
| --- | --- | --- |
| Dữ liệu sinh viên | Có tên, email, GitHub, bài làm — dữ liệu cá nhân | Thu thập tối thiểu; quy trình xóa/ẩn danh (DATABASE §8); chính sách riêng tư ngắn bằng tiếng Việt; Open Question: có cần đăng ký với nhà trường? |
| Bản quyền nội dung | it-quiz, html-css, tài liệu sưu tầm | Mỗi bài ghi `license`, `authors`, `source`; import chỉ khi xác nhận (đã có trong CONTENT-SPECIFICATION) |
| Giấy phép nội dung | SQL Challenge là MIT | Chọn giấy phép content repo (đề xuất CC BY-SA 4.0 cho văn bản + MIT cho code mẫu) — Open Question |
| Đạo văn / sao chép | Không có điểm số → rủi ro thấp; nhưng test ẩn dễ bị đoán | Test ẩn + đa dạng dữ liệu; mentor hỏi lại khi nghi ngờ; không cấm thảo luận |
| Chạy code thành viên | Dù trên runner GitHub-hosted, vẫn là code không tin cậy | Không secret trong checker repo; token chỉ đọc; timeout; chỉ repo công khai |

## 6. Tiêu chí "đi / không đi" sau Phase 0 (đề xuất)

Chạy 1 đợt SQL (4–6 tuần) với ≥ 20 thành viên bằng công cụ có sẵn. Chỉ xây app nếu **ít nhất 3/5**:
1. ≥ 50% người đăng ký làm ≥ 5 bài.
2. ≥ 3 mentor cam kết review ≥ 1 bài/tuần suốt đợt.
3. ≥ 2 thành viên (không thuộc Ban) viết/sửa nội dung qua PR.
4. Người dùng chỉ ra ≥ 3 nỗi đau mà công cụ hiện tại không giải quyết được (tiến độ, hàng đợi, chấm tự động…).
5. Có ≥ 3 sinh viên cam kết làm dev trong ≥ 2 học kỳ và 1 người kế nhiệm.
Nếu không đạt, dừng ở content repo + site tĩnh (giá trị vẫn đủ lớn, chi phí gần 0).

## 7. Điều chỉnh chỉ số thành công

Mục tiêu bản nháp (40% weekly active, 25% hoàn thành SQL) có vẻ **lạc quan** cho học tự nguyện ngoài giờ. Điều chỉnh: dùng Phase 0 để **đo baseline** rồi đặt mục tiêu; ví dụ ngưỡng hợp lý ban đầu: 15–25% WAU, 10–15% hoàn thành track 30 ngày (Exercism/MOOC tự nguyện thường dưới 15%). Đã cập nhật PRD §11.

## 8. Quyết định cần Ban chủ nhiệm / nhóm mentor (tổng hợp)

1. Có chạy **Phase 0** trước khi xây app không? (khuyến nghị: có)
2. Ai là chủ sở hữu tài khoản/hạ tầng (GitHub App, domain, VM, secret)? Ngân sách hosting?
3. Mở cho toàn trường hay chỉ thành viên CLB? Cách duyệt thành viên mới?
4. Mentor: số lượng cam kết, giới hạn mentee, quy tắc SLA, hình thức ghi nhận.
5. Giấy phép nội dung và quy trình xác nhận bản quyền khi import.
6. Có cần liên kết điểm rèn luyện/chứng nhận hoạt động của trường không? (chưa kiểm chứng — hỏi Đoàn/Hội/khoa)
7. Review ghi ở hub hay GitHub (đề xuất: kết luận ở hub, thảo luận chi tiết ở cả hai).
