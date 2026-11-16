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
**Hành động:** thêm **Phase 0 (xác thực nhu cầu)** ở MVP-BACKLOG: trước khi xây thêm tính năng app, dựng `learning-hub-content` + import SQL Challenge + chạy thử một đợt (cohort) bằng công cụ có sẵn (site VitePress + GitHub PR/Issue + kênh Discord). Chỉ xây app khi đạt tiêu chí "đi/không đi" (xem §6). Mục tiêu SLA review hạ từ 48 giờ → **72 giờ** làm chỉ số theo dõi, không cam kết; 72 giờ chỉ là **mặc định**, cam kết mentor chốt theo từng học kỳ (§8).
**Đã chốt (16/11/2026): có chạy Phase 0.** Khung app E1–E3 đã dựng từ trước; Phase 0 lúc này = content repo + import SQL Challenge + một đợt thử, chưa thêm tính năng app cho tới quyết định đi/không đi.

## 2. Độ phức tạp so với năng lực đội sinh viên

### 2.1 🟠 Quá nhiều thứ cho nhóm 3–5 sinh viên bán thời gian
Bản nháp có 11 module Nest, 20 bảng, GitHub App, Discord, content sync, checker repo. Một học kỳ khó làm hết.
**Hành động:** MVP chia hai mức trong MVP-BACKLOG:
- **MVP-Lite (≈ 6–8 tuần):** đăng nhập GitHub, catalog đọc từ Git (sync thủ công bằng lệnh/endpoint admin), enroll + lesson progress, nộp bài `quiz` và `self_check`, **một** loại bài `code` qua checker repo, review của mentor. Không Discord bot, không outbox (gửi webhook Discord đơn giản, lỗi chỉ log).
- **MVP-Full:** phần còn lại (mentorship UI đầy đủ, dashboard admin, reconcile job, audit UI) + theo quyết định 16/11/2026: `sql-checker` (làm ngay sau MVP-Lite), bảng xếp hạng opt-in, Web Push (DECISIONS ADR-07, ADR-08).
Gộp module khi cài đặt để giảm ma sát: `catalog + content + learning` có thể ở cùng một Nest module "learning" ban đầu (giữ **thư mục con** theo ranh giới để tách sau). Quy tắc ranh giới vẫn giữ, chỉ giảm số `*.module.ts`.

### 2.2 🟠 Bus factor và kế nhiệm
Sinh viên tốt nghiệp sau 2–3 năm; stack gồm NestJS, Next.js, Prisma, Docker, GitHub App, Discord, VM. Ít người nắm hết.
**Hành động:**
- Thêm tài liệu bàn giao bắt buộc (SECURITY/ARCHITECTURE đã nhắc; MVP-BACKLOG thêm epic "Vận hành & bàn giao": runbook, danh sách tài khoản/secret, checklist đổi người giữ).
- **Đã chốt (16/11/2026):** người giữ hiện tại của tài khoản/hạ tầng (GitHub App/OAuth app, domain, VPS, secret, Discord webhook) là **Chủ nhiệm CLB đương nhiệm** (GitHub `TienNHM`), đồng thời tài trợ VPS. Để giảm bus factor, runbook bàn giao liệt kê mọi tài khoản/secret **theo chức danh** (không chứa giá trị) để chuyển quyền sở hữu cho Chủ nhiệm kế tiếp; secret lưu trong trình quản lý chung (vd Bitwarden của CLB).
- Chọn công nghệ **phổ biến, dễ tìm tài liệu**; từ chối thêm thư viện hiếm. Mỗi quý "dependency day" cập nhật gói.
- Hosting **đã chốt**: 1 VPS (được tài trợ) chạy Docker Compose như ARCHITECTURE §14. Hosting managed (Render/Fly/Railway + Postgres managed) chỉ xem lại nếu không còn VPS tài trợ.

### 2.3 🟠 Có nên tự xây? So sánh với công cụ sẵn có
| Lựa chọn | Hợp lý khi | Vì sao chưa chọn |
| --- | --- | --- |
| **Không xây gì mới**, dùng repo template trên GitHub + PR/Issue + site tĩnh + Discord | Muốn chạy ngay, ngân sách 0 | Không có tiến độ chung, không có hàng đợi review; vẫn là bước tốt cho Phase 0. (GitHub Classroom đã ngừng hỗ trợ nên không còn là lựa chọn.) |
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
**Hành động (đã chốt 16/11/2026):** duyệt bằng **danh sách email** do CLB duy trì (email thành viên, bảng `approved_emails`). Khi đăng nhập GitHub, API đọc **mọi** email đã xác minh của người dùng (`GET /user/emails`, không chỉ email chính); nếu có email khớp danh sách → `active`, ngược lại `pending` để admin/mentor duyệt tay (`users.status = 'pending'`). Thành viên có email trường trong danh sách có thể thêm email đó vào GitHub làm email phụ đã xác minh. Membership org `hcmute-dsc` **không** còn là quy tắc tự duyệt (chỉ là tùy chọn bổ sung). Admin nhập/thay danh sách trong app (dán từ form thành viên của CLB); danh sách là dữ liệu cá nhân → chỉ admin, thay đổi ghi audit, không xuất cho member. Xem SECURITY §1, DATABASE §3.1b, USER-FLOWS §1.

### 3.3 🟡 Học theo "đợt" (cohort/sprint), không chỉ tự học tự do
CLB quen chạy theo đợt (SQL Challenge 30 ngày, Study Jam, Gen mới): cả nhóm cùng nhịp, có deadline mềm, có buổi gặp. Bản nháp xếp cohort vào "Future".
**Hành động:** thêm cột nhẹ `enrollments.cohort` (text, null) ở MVP để gắn nhãn đợt ("sql-2027-spring") và lọc tiến độ theo đợt; cohort đầy đủ (lịch, deadline) ở P2.

### 3.4 🟡 Văn hóa "học, không chấm điểm"
Cho CLB học thuật, tiến độ nên khuyến khích chứ không xếp hạng. Bản nháp đề xuất: không leaderboard; huy hiệu tắt được; hiển thị **tiến độ cá nhân** và "đang học cùng bạn" (số người cùng đợt) thay vì xếp hạng. P2: xem lời giải cộng đồng **sau khi tự hoàn thành** (giống Exercism) để học lẫn nhau.
**Đã chốt (16/11/2026):** Chủ nhiệm CLB chọn **giữ bảng xếp hạng ở MVP-Full** (DECISIONS ADR-08) với "thiết kế an toàn" để giảm các rủi ro trên:
- **Opt-in:** mặc định ẩn (`users.leaderboard_opt_in = false`); chỉ người tự bật mới xuất hiện, tắt là biến mất ngay; chỉ thành viên đã đăng nhập xem được.
- **Theo học kỳ/đợt, reset mỗi kỳ;** không có xếp hạng toàn thời gian.
- **Xếp theo số bài hoàn thành** (`approved` hoặc tự động `passed`); `self_check` có trọng số thấp hơn bài chấm tự động hoặc mentor duyệt; **không bao giờ** xếp theo tốc độ.
- **Chỉ hiện top 10 + hạng của chính người xem;** điểm tính bằng truy vấn/view, không lưu.
Tiến độ cá nhân và "đang học cùng bạn" vẫn là hiển thị chính; huy hiệu vẫn tắt được (P2).

### 3.5 🟡 Phạm vi chủ đề
CLB gồm cả CTF, AI, phỏng vấn (InterviewPrep101), không chỉ lập trình web. Schema nội dung đã không phụ thuộc ngôn ngữ; nhưng loại bài hiện có thiên về code. Để cho track "phỏng vấn/soft skill" dùng được: `project` + `self_check` + `quiz` đã đủ. Không thêm loại mới ở MVP.

## 4. Mentor & vận hành cộng đồng

### 4.1 🟠 Tải công việc của mentor
- Review PR mất 15–30 phút; 5 mentor × 3 bài/tuần ≈ 15 bài/tuần — đó là trần thực tế.
- **Hành động:** tự động hóa tối đa trước review (CI + checklist); chỉ cần review khi bài `project` hoặc khi member bấm "xin review" cho bài `code` đã `passed`; **giới hạn số mentee/mentor** (mặc định 5, cấu hình `MENTOR_MAX_MENTEES`; giá trị chốt theo học kỳ); hàng đợi hiển thị "bài chờ lâu nhất"; P2: review ngang hàng (peer review) có hướng dẫn.
- Ghi nhận mentor (huy hiệu, chứng nhận hoạt động) như phần thưởng chính — đừng để mentor quá tải mà không được thấy giá trị.

### 4.2 🟡 Mentor là người chịu trách nhiệm?
Phân định rõ: mentor góp ý, **không** có trách nhiệm học thuật chính thức. Điều khoản ngắn trong onboarding. **Đã chốt (16/11/2026): có điều khoản này.**

## 5. An toàn, pháp lý, riêng tư (bối cảnh trường học)

| Mối quan tâm | Nhận định | Hành động |
| --- | --- | --- |
| Dữ liệu sinh viên | Có tên, email, GitHub, bài làm — dữ liệu cá nhân | Thu thập tối thiểu; quy trình xóa/ẩn danh (DATABASE §8); chính sách riêng tư ngắn bằng tiếng Việt; **Đã chốt (16/11/2026): không cần đăng ký với nhà trường; dữ liệu lưu riêng tư, chỉ admin truy cập.** |
| Bản quyền nội dung | it-quiz, html-css, tài liệu sưu tầm | Mỗi bài ghi `license`, `authors`, `source`; người import xác nhận nguồn/tác giả trong mô tả PR trước khi import (CONTENT-SPECIFICATION §10) |
| Giấy phép nội dung | SQL Challenge là MIT | **Đã chốt (16/11/2026):** nội dung import giữ giấy phép gốc (track `sql-30-days` dùng MIT, giữ ghi nhận tác giả + link nguồn); nội dung mới mặc định CC BY-SA 4.0 cho văn bản + MIT cho code mẫu (CONTENT-SPECIFICATION §10) |
| Đạo văn / sao chép | Không có điểm số → rủi ro thấp; bảng xếp hạng opt-in (MVP-Full) tạo thêm động cơ nhỏ; test ẩn dễ bị đoán | Test ẩn + đa dạng dữ liệu; `self_check` trọng số thấp, không xếp theo tốc độ (§3.4); mentor hỏi lại khi nghi ngờ; không cấm thảo luận |
| Chạy code thành viên | Dù trên runner GitHub-hosted, vẫn là code không tin cậy; truy vấn SQL trên `sql-checker` (MVP-Full) chạy trên VPS của CLB | Không secret trong checker repo; token chỉ đọc; timeout; chỉ repo công khai. `sql-checker`: PostgreSQL riêng, role ít quyền, read-only, timeout, không mạng ra; Judge0/Piston không bao giờ chung VPS với hub (SECURITY §7) |

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

> **Đã chốt 16/11/2026** (Chủ nhiệm CLB quyết định). Câu hỏi còn mở: xem README.

1. Có chạy **Phase 0** trước khi xây app không? → **Có.** Khung app E1–E3 đã có; Phase 0 = content repo + import SQL Challenge + một đợt thử, chưa thêm tính năng app tới khi có quyết định đi/không đi (§6).
2. Ai là chủ sở hữu tài khoản/hạ tầng (GitHub App, domain, VM, secret)? Ngân sách hosting? → **Chủ nhiệm CLB đương nhiệm** (GitHub `TienNHM`) giữ GitHub App/OAuth app, domain, VPS, secret và tài trợ VPS. Hosting = 1 VPS + Docker Compose. Giữ biện pháp bus factor: runbook bàn giao liệt kê mọi tài khoản/secret theo chức danh để chuyển cho Chủ nhiệm kế tiếp (§2.2, MVP-BACKLOG E11).
3. Mở cho toàn trường hay chỉ thành viên CLB? Cách duyệt thành viên mới? → **Duyệt bằng danh sách email** của CLB: email đã xác minh trên GitHub (mọi email, không chỉ email chính) khớp danh sách → `active`; còn lại `pending`, admin/mentor duyệt tay. Org GitHub chỉ là tùy chọn (§3.2).
4. Mentor: số lượng cam kết, giới hạn mentee, quy tắc SLA, hình thức ghi nhận. → **Chốt theo từng học kỳ.** Thiết kế giữ các con số này ở cấu hình (`MENTOR_MAX_MENTEES`, `REVIEW_TARGET_HOURS`), không hard-code; 72 giờ chỉ là mục tiêu mặc định.
5. Giấy phép nội dung và quy trình xác nhận bản quyền khi import. → **Giao tech lead**; ghi nhận: nội dung import giữ giấy phép gốc (SQL Challenge MIT), nội dung mới CC BY-SA 4.0 (văn bản) + MIT (code mẫu), người import xác nhận nguồn/tác giả trong mô tả PR (CONTENT-SPECIFICATION §10).
6. Có cần liên kết điểm rèn luyện/chứng nhận hoạt động của trường không? → Hub cấp **chứng nhận hoàn thành riêng** cho mỗi track (trang/link xác minh được) ở **P2** (PRD F19). Quy đổi điểm rèn luyện: **ngoài phạm vi**, chưa xác nhận với trường.
7. Review ghi ở hub hay GitHub? → **Kết luận** (`approved`/`changes_requested`) ghi ở hub; thảo luận chi tiết ở cả hub và GitHub (USER-FLOWS §8).
8. Các quyết định kiến trúc (DECISIONS) → **Đã chốt toàn bộ ADR-01…ADR-09.** ADR-01 = B (ứng dụng mới NestJS + Next.js); ADR-02 = C; ADR-03 = B (SQL Challenge đã import, HTML/CSS sau, Course-DSC deprecate); ADR-04 = A (monorepo pnpm, content repo riêng); ADR-05 = C; ADR-06 = C (Google/email trường ở P2); ADR-09 chấp nhận. **ADR-07:** A + D ở MVP; ngay sau MVP-Lite thêm `sql-checker` chỉ cho SQL trên VPS (PostgreSQL riêng, read-only, timeout); Judge0/Piston nhiều ngôn ngữ chỉ Future và không bao giờ chung VPS với hub. **ADR-08:** giữ **bảng xếp hạng opt-in theo học kỳ** (§3.4) và **Web Push** ở MVP-Full; các mục loại khác giữ nguyên.
