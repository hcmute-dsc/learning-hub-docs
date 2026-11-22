# Bộ công cụ đợt dùng thử Phase 0 (P0-3)

Mọi thứ Ban chủ nhiệm cần để chạy đợt dùng thử track SQL: checklist trước ngày đầu, tin Discord soạn sẵn, cách lấy số liệu mỗi tuần, form phản hồi cuối đợt và mẫu báo cáo cho buổi quyết định.

- Việc: [docs#1](https://github.com/hcmute-dsc/learning-hub-docs/issues/1) (P0-3). Quyết định sau đợt: [docs#2](https://github.com/hcmute-dsc/learning-hub-docs/issues/2) (P0-4).
- Tiêu chí đi/không đi: [CRITIQUE §6](../product/CRITIQUE.md#6-tiêu-chí-đi--không-đi-sau-phase-0-đề-xuất). Kế hoạch Phase 0: [MVP-BACKLOG §0](../product/MVP-BACKLOG.md#0-phase-0--xác-thực-nhu-cầu-trước-khi-viết-app--must).
- Nền tảng: app Learning Hub (https://learn.hcmute-dsc.io.vn), track `sql-30-days` (nội dung ở [learning-hub-track-sql-30-days](https://github.com/hcmute-dsc/learning-hub-track-sql-30-days)), Discord CLB.
- Trong Phase 0 **không** thêm tính năng app. Số nào app chưa đo được thì ghi tay hoặc chạy truy vấn đọc (§5), và ghi rõ issue backlog sẽ thêm nó (§8).

> Repo này công khai. Tài liệu, báo cáo và bảng số liệu ở đây chỉ chứa **số tổng hợp**. Tên, email, GitHub login của thành viên chỉ nằm trong app (trang quản trị) và trong file riêng của Ban chủ nhiệm.

## 1. Ai làm gì

| Vai trò | Việc trong đợt thử |
| --- | --- |
| Chủ nhiệm / Ban chủ nhiệm | Chọn nhóm thử, đăng tin Discord, nhắc hằng tuần, gửi form cuối đợt, viết báo cáo §7 |
| Admin app (Ban chủ nhiệm) | Thêm email vào **Quản trị → Thành viên**, duyệt tài khoản chờ, kiểm tra track đã publish |
| Tech lead | Chạy truy vấn §5 trên production mỗi tuần, gửi số cho Ban chủ nhiệm; theo dõi lỗi app |
| Mentor (≥ 3) | Trả lời câu hỏi trên Discord, review bài dự án cuối khóa trong **Review**, ghi số lượt hỗ trợ mỗi tuần (§4.3) |

## 2. Checklist trước ngày 1

Làm theo thứ tự. Ghi ngày làm xong (DD/MM/YYYY) vào cột cuối.

| # | Việc | Ai | Cách kiểm tra | Xong ngày |
| --- | --- | --- | --- | --- |
| 1 | Cài đặt org `hcmute-dsc` xong **trước** khi mời ai vào org: [docs#3](https://github.com/hcmute-dsc/learning-hub-docs/issues/3). Tài liệu này không đổi cài đặt nào; nếu docs#3 chưa đóng thì **chưa** mời thành viên vào org GitHub (đăng nhập app không cần vào org) | Chủ nhiệm (admin org) | docs#3 đã đóng | |
| 2 | Chốt lịch: ngày bắt đầu (nên là **thứ Hai**), số tuần (4–6), ngày gửi form, ngày họp P0-4 | Ban chủ nhiệm | Ghi vào docs#1 | |
| 3 | Chốt ≥ 3 mentor cam kết suốt đợt; mỗi mentor có role `mentor` trong app | Ban chủ nhiệm | **Quản trị → Thành viên**, cột vai trò | |
| 4 | Track `sql-30-days` đang ở bản **published** mới nhất | Admin app | **Quản trị → Nội dung**; mở https://learn.hcmute-dsc.io.vn/tracks/sql-30-days thấy 4 chặng | |
| 5 | Tạo form đăng ký riêng (Google Form, chỉ Ban chủ nhiệm xem câu trả lời): họ tên, **email đã xác minh trên GitHub** (email trường thêm vào GitHub làm email phụ cũng được), GitHub login, cam kết học 4–6 tuần | Ban chủ nhiệm | Link form để dán vào tin §3.1 | |
| 6 | Tạo kênh Discord riêng cho đợt, ví dụ `#sql-dung-thu`, và một kênh/thread hỏi đáp | Ban chủ nhiệm | Kênh đã có | |
| 7 | Đăng tin §3.1, nhận đăng ký tới khi đủ **≥ 20 người** | Ban chủ nhiệm | Số câu trả lời trong form | |
| 8 | Dán email từ form vào **Quản trị → Thành viên → Danh sách email** (chọn "Thêm vào danh sách hiện có"; ô ghi chú ví dụ `Đợt thử SQL 11/2026`) | Admin app | Số "Danh sách email thành viên (N)" tăng đúng | |
| 9 | Đi thử hết luồng bằng một tài khoản có role `admin` hoặc `mentor` (truy vấn §5 bỏ qua hai role này, nên số liệu không bị lệch): đăng nhập GitHub → **Bắt đầu học** track → đánh dấu 1 bài học → nộp 1 bài tự đánh giá. Nếu dùng tài khoản thành viên thường để thử, nó sẽ bị tính vào M1 | Admin app | Tài khoản thử thấy tiến độ trên Dashboard | |
| 10 | Tech lead chạy thử truy vấn §5.2 với tuần 0 (trước ngày bắt đầu) để chắc lệnh chạy được | Tech lead | Có bảng số, không lỗi | |
| 11 | Ngày 1: đăng tin khai mạc (§3.2) | Ban chủ nhiệm | | |

Trong tuần đầu, mở **Quản trị → Thành viên** mỗi ngày: ai đăng nhập bằng GitHub mà email không khớp danh sách sẽ nằm ở mục chờ duyệt; đối chiếu với form đăng ký rồi duyệt tay.

## 3. Tin Discord soạn sẵn

Thay các chỗ `<…>` rồi dán. Không nêu tên hay email thành viên trong kênh chung.

### 3.1 Tin mở đăng ký

```text
📢 DSC mở đợt HỌC THỬ "30 ngày học SQL" trên Learning Hub của CLB!

🗓️ Thời gian: <N> tuần, từ <DD/MM> đến <DD/MM>
🎯 Dành cho: thành viên CLB muốn học SQL từ cơ bản (không cần biết trước)
📚 Nội dung: 4 chặng — SELECT/WHERE → GROUP BY, JOIN → view, index, transaction → tối ưu và bảo mật, kết thúc bằng một dự án nhỏ có mentor review
🧑‍🏫 Có <số> mentor hỗ trợ trong kênh #<kênh-hỏi-đáp>

Bạn sẽ:
• Học trên https://learn.hcmute-dsc.io.vn (đăng nhập bằng GitHub)
• Làm bài, tự theo dõi tiến độ, nộp dự án để mentor góp ý
• Góp ý giúp CLB quyết định có đầu tư tiếp cho nền tảng này không

👉 Đăng ký trước <DD/MM>: <LINK_FORM_ĐĂNG_KÝ>
Lưu ý: điền email đã xác minh trên tài khoản GitHub của bạn (email trường thêm vào GitHub làm email phụ cũng được) để vào được ngay.

Cần ít nhất 20 bạn để đợt thử có ý nghĩa — rủ bạn cùng học nhé! 💪
```

### 3.2 Tin khai mạc (ngày 1)

```text
🚀 Đợt học thử SQL bắt đầu hôm nay!

1️⃣ Vào https://learn.hcmute-dsc.io.vn → Đăng nhập với GitHub
2️⃣ Mở track "30 ngày học SQL" → bấm "Bắt đầu học"
3️⃣ Cài môi trường theo bài đầu tiên (PostgreSQL 16 + bộ dữ liệu DSC Shop)
4️⃣ Học xong bài nào thì bấm đánh dấu đã học; làm bài tập rồi nộp ngay trên trang bài tập

📌 Gợi ý nhịp học: khoảng <8> bài/tuần, mỗi ngày 30–60 phút.
❓ Kẹt chỗ nào hỏi ở #<kênh-hỏi-đáp>, mentor sẽ trả lời.
🔒 Đăng nhập xong mà thấy trang "chờ duyệt"? Nhắn riêng Ban chủ nhiệm, đừng đăng email lên kênh chung.
🐞 Gặp lỗi trên web: báo ở #<kênh-hỏi-đáp> kèm ảnh chụp màn hình.
```

### 3.3 Nhắc hằng tuần (mỗi thứ Hai)

Chỉ dùng số tổng của cả nhóm (lấy từ bảng §5.4), không xếp hạng, không nêu tên.

```text
📅 Tuần <k>/<N> đợt học thử SQL

Tuần qua cả nhóm:
• <A> bạn đã học trên web
• <B> bài học và <C> bài tập hoàn thành
• <D> bạn đã làm từ 5 bài tập trở lên

🎯 Tuần này: chặng <tên chặng> (<gợi ý bài>)
💡 Mẹo: <một mẹo ngắn hoặc câu hỏi hay trong kênh tuần qua>
🧑‍🏫 Mentor trực hỏi đáp tuần này: <@vai-trò-mentor>

Bị chậm cũng không sao — học tiếp từ bài đang dở, tiến độ được lưu sẵn. Cố lên! 💪
```

### 3.4 Tin gửi form cuối đợt

```text
🏁 Đợt học thử SQL kết thúc! Cảm ơn cả nhóm đã cùng học.

Dành 5 phút trả lời form này giúp CLB quyết định có làm tiếp Learning Hub không: <LINK_FORM_PHẢN_HỒI>
Hạn: <DD/MM>. Bạn chưa học xong cũng nên trả lời — ý kiến của bạn quan trọng nhất.

Bạn vẫn học tiếp được trên web như bình thường.
```

## 4. Số liệu mỗi tuần

### 4.1 Ghi gì, lấy ở đâu

"Người học" = tài khoản đã bấm **Bắt đầu học** track `sql-30-days`, không có role `admin` hay `mentor`. Tuần tính theo giờ Việt Nam, từ thứ Hai 00:00 tới thứ Hai tuần sau.

| # | Số liệu | Nghĩa | Lấy ở đâu | Đo được hôm nay? |
| --- | --- | --- | --- | --- |
| M1 | Người đăng ký | Số người học (tích lũy) | **Quản trị → Báo cáo** hoặc truy vấn §5.2 | Có |
| M2 | Người học hoạt động | Người học có đánh dấu bài học hoặc nộp bài trong tuần | **Quản trị → Báo cáo** hoặc truy vấn §5.2 | Có |
| M3 | Bài học hoàn thành | Lượt bài học được đánh dấu "đã học" trong tuần | **Quản trị → Báo cáo** hoặc truy vấn §5.2 | Có |
| M4 | Bài tập hoàn thành | Lượt bài tập lần đầu xong trong tuần (bài tự đánh giá, quiz đạt, dự án được duyệt) | **Quản trị → Báo cáo** hoặc truy vấn §5.2 | Có |
| M5 | Người học có ≥ 5 bài tập xong | Tích lũy từ đầu đợt; dùng cho tiêu chí 1 | **Quản trị → Báo cáo** hoặc truy vấn §5.2 | Có |
| M6 | Bài chờ review | Bài nộp đang chờ mentor, tại lúc chạy | **Review** (trang hàng đợi: "N bài đang chờ, trong đó M bài quá hạn"), **Quản trị → Báo cáo** hoặc truy vấn §5.2 | Có (trang Review) |
| M7 | Thời gian review | Trung vị và lâu nhất (giờ) từ lúc nộp tới lúc mentor kết luận, các review xong trong tuần | **Quản trị → Báo cáo** hoặc truy vấn §5.2 | Có |
| M8 | Mentor có review | Số mentor kết luận ≥ 1 review trong tuần | **Quản trị → Báo cáo** hoặc truy vấn §5.2 | Có |
| M9 | Lượt mentor hỗ trợ trên Discord | Câu hỏi được mentor trả lời trong tuần | Mentor tự ghi (§4.3) | Chỉ ghi tay |
| M10 | PR nội dung từ thành viên ngoài Ban | PR vào repo track trong tuần (mở / đã merge) | `gh pr list` (§5.3) | Có (lệnh `gh`) |

**Lấy M1–M8 trên app** ([learning-hub#34](https://github.com/hcmute-dsc/learning-hub/issues/34)). Admin mở **Quản trị → Báo cáo**, bấm **Tuần trước** (thứ Hai → Chủ nhật vừa qua), chọn track `sql-30-days` và đợt của đợt thử, rồi chép dòng tổng vào bảng §4.2, hoặc tải **CSV tổng hợp** (mỗi dòng có sẵn cột từ ngày / đến ngày nên dán các tuần nối nhau được). Cách đếm giống truy vấn §5.2, khác hai điểm nhỏ: M5 đếm mọi bài tập xong tới cuối tuần kể từ lúc ghi danh (lọc theo đợt để chỉ tính người của đợt thử); M6 là số lúc mở trang. Gắn nhãn đợt lúc ghi danh (`enrollments.cohort`) thì lọc được theo đợt; ai chưa có nhãn nằm ở "Chưa gắn đợt". Khi gửi CSV ra ngoài Ban chủ nhiệm, chọn **Ẩn danh**: bảng từng người học chỉ còn "Người học 001"…. Mỗi lần tải được ghi vào **Nhật ký**.

**Vì sao cần M9.** Trong track `sql-30-days` hiện có 33 bài tập: 31 bài tự đánh giá (`self_check`, tự xong khi nộp), 1 quiz (app tự chấm) và **chỉ 1 bài cần mentor review** là dự án cuối khóa (`project`). Nên trong các tuần đầu hàng đợi review gần như trống và M6–M8 bằng 0 là bình thường. Tiêu chí 2 ("mentor review ≥ 1 bài/tuần") vì vậy được đánh giá bằng **M8 + M9**: mentor review bài trong app khi có, còn lại tính lượt hỗ trợ trên Discord.

**Giới hạn cần ghi trong báo cáo.**

- Bài `self_check` là thành viên tự đánh giá, app không kiểm tra đáp án. M4 đo "đã làm và nộp", không đo "làm đúng".
- M2 không tính người chỉ mở trang đọc mà không đánh dấu hay nộp gì: app chưa ghi lượt xem.

### 4.2 Bảng điền mỗi tuần

Sao vào docs#1 (comment) mỗi thứ Hai, hoặc giữ trong file riêng của Ban chủ nhiệm. Chỉ ghi số.

| Tuần | Từ – đến | M1 đăng ký | M2 hoạt động | M3 bài học | M4 bài tập | M5 ≥ 5 bài | M6 chờ review | M7 trung vị / lâu nhất (giờ) | M8 mentor review | M9 lượt hỗ trợ Discord | M10 PR mở / merge | Ghi chú |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | | | | | | | | | | | | |
| 2 | | | | | | | | | | | | |
| 3 | | | | | | | | | | | | |
| 4 | | | | | | | | | | | | |
| 5 | | | | | | | | | | | | |
| 6 | | | | | | | | | | | | |

### 4.3 Mentor tự ghi lượt hỗ trợ (M9)

Mỗi mentor giữ một dòng trong file riêng của Ban chủ nhiệm (không đưa lên repo): tuần, số câu hỏi đã trả lời, số bài đã review (trong app hoặc góp ý repo của thành viên). Báo cáo chỉ dùng: tổng lượt mỗi tuần, và **số mentor có ≥ 1 lượt ở mọi tuần**.

## 5. Truy vấn cho tech lead

### 5.1 Quy tắc

- Chỉ tech lead chạy, trên VPS, trong thư mục `/opt/learning-hub` (như [DEPLOYMENT §5.3](DEPLOYMENT.md#53-kiểm-tra-bản-sao-lưu-mỗi-tháng)).
- Truy vấn chạy trong `BEGIN TRANSACTION READ ONLY` và kết thúc bằng `ROLLBACK`: Postgres từ chối mọi lệnh ghi, kể cả khi gõ nhầm.
- Chỉ trả về **số đếm**. Không `SELECT` email, tên, GitHub login hay id. Nếu cần sửa truy vấn, giữ nguyên quy tắc này.
- Gửi kết quả cho Ban chủ nhiệm, điền bảng §4.2.

### 5.2 Số liệu một tuần (M1–M8)

Đặt `WS` = thứ Hai đầu tuần, `WE` = thứ Hai tuần sau, `TS` = ngày bắt đầu đợt (định dạng `YYYY-MM-DD`).

```bash
cd /opt/learning-hub
WS=2026-11-23; WE=2026-11-30; TS=2026-11-23
docker compose -f docker-compose.prod.yml exec -T db \
  psql -U hub -d learning_hub -v ON_ERROR_STOP=1 -P footer=off \
  -v track=sql-30-days -v ws="$WS" -v we="$WE" -v ts="$TS" <<'SQL'
BEGIN TRANSACTION READ ONLY;

WITH p AS (SELECT
    (:'ws'::date)::timestamp AT TIME ZONE 'Asia/Ho_Chi_Minh' AS ws,
    (:'we'::date)::timestamp AT TIME ZONE 'Asia/Ho_Chi_Minh' AS we,
    (:'ts'::date)::timestamp AT TIME ZONE 'Asia/Ho_Chi_Minh' AS ts),
-- Người học: ghi danh track, chưa xóa, không phải admin/mentor
learners AS (
  SELECT e.user_id FROM enrollments e
  JOIN tracks t ON t.id = e.track_id AND t.slug = :'track'
  JOIN users u ON u.id = e.user_id AND u.deleted_at IS NULL
  WHERE NOT EXISTS (SELECT 1 FROM user_roles ur JOIN roles r ON r.id = ur.role_id
                    WHERE ur.user_id = e.user_id AND r.code IN ('admin', 'mentor'))),
track_lessons AS (
  SELECT l.id FROM lessons l JOIN modules m ON m.id = l.module_id
  JOIN tracks t ON t.id = m.track_id AND t.slug = :'track'),
track_ex AS (
  SELECT x.id, x.requires_review FROM exercises x JOIN modules m ON m.id = x.module_id
  JOIN tracks t ON t.id = m.track_id AND t.slug = :'track'),
-- Lần đầu mỗi người xong mỗi bài tập: dự án tính lúc mentor duyệt, bài khác tính lúc nộp
first_done AS (
  SELECT s.user_id, s.exercise_id,
         min(CASE WHEN s.status = 'approved' THEN rv.submitted_at ELSE s.submitted_at END) AS done_at
  FROM submissions s
  JOIN track_ex x ON x.id = s.exercise_id
  JOIN learners lr ON lr.user_id = s.user_id
  LEFT JOIN reviews rv ON rv.submission_id = s.id AND rv.status = 'approved'
  WHERE s.status = 'approved' OR (s.status = 'passed' AND NOT x.requires_review)
  GROUP BY s.user_id, s.exercise_id),
-- Review đã kết luận trong tuần (bài thuộc track)
week_reviews AS (
  SELECT rv.reviewer_id,
         extract(epoch FROM rv.submitted_at - s.submitted_at) / 3600 AS hours
  FROM reviews rv
  JOIN submissions s ON s.id = rv.submission_id
  JOIN track_ex x ON x.id = s.exercise_id, p
  WHERE rv.status IN ('approved', 'changes_requested')
    AND rv.submitted_at >= p.ws AND rv.submitted_at < p.we),
active AS (
  SELECT lp.user_id FROM lesson_progress lp
  JOIN track_lessons tl ON tl.id = lp.lesson_id, p
  WHERE lp.updated_at >= p.ws AND lp.updated_at < p.we
  UNION
  SELECT s.user_id FROM submissions s
  JOIN track_ex x ON x.id = s.exercise_id, p
  WHERE s.submitted_at >= p.ws AND s.submitted_at < p.we)
SELECT
  (SELECT count(*) FROM learners) AS m1_dang_ky,
  (SELECT count(*) FROM active WHERE user_id IN (SELECT user_id FROM learners)) AS m2_hoat_dong,
  (SELECT count(*) FROM lesson_progress lp
     JOIN track_lessons tl ON tl.id = lp.lesson_id
     JOIN learners lr ON lr.user_id = lp.user_id, p
   WHERE lp.status = 'completed' AND lp.completed_at >= p.ws AND lp.completed_at < p.we) AS m3_bai_hoc,
  (SELECT count(*) FROM first_done, p WHERE done_at >= p.ws AND done_at < p.we) AS m4_bai_tap,
  (SELECT count(*) FROM (SELECT user_id FROM first_done, p WHERE done_at >= p.ts AND done_at < p.we
                         GROUP BY user_id HAVING count(*) >= 5) f) AS m5_tu_5_bai,
  (SELECT count(*) FROM submissions s JOIN track_ex x ON x.id = s.exercise_id
   WHERE s.status = 'awaiting_review') AS m6_cho_review,
  (SELECT round((percentile_cont(0.5) WITHIN GROUP (ORDER BY hours))::numeric, 1)
   FROM week_reviews) AS m7_trung_vi_gio,
  (SELECT round(max(hours)::numeric, 1) FROM week_reviews) AS m7_lau_nhat_gio,
  (SELECT count(DISTINCT reviewer_id) FROM week_reviews) AS m8_mentor_review;

ROLLBACK;
SQL
```

Kết quả là **một dòng số**. M7 để trống (NULL) khi trong tuần không có review nào.

### 5.3 Số liệu cuối đợt (cho báo cáo)

**Tiêu chí 1** — tỉ lệ người học có ≥ 5 bài tập xong: lấy M5 và M1 của tuần cuối, tính `M5 / M1`.

**Tiêu chí 2** — số mentor có review trong app ở **mọi** tuần của đợt. `TS` = ngày bắt đầu, `WE` = thứ Hai sau tuần cuối, `N` = số tuần:

```bash
cd /opt/learning-hub
TS=2026-11-23; WE=2027-01-04; N=6
docker compose -f docker-compose.prod.yml exec -T db   psql -U hub -d learning_hub -v ON_ERROR_STOP=1 -P footer=off   -v track=sql-30-days -v ts="$TS" -v we="$WE" -v n="$N" <<'SQL'
BEGIN TRANSACTION READ ONLY;
SELECT count(*) AS mentor_review_moi_tuan FROM (
  SELECT rv.reviewer_id FROM reviews rv
  JOIN submissions s ON s.id = rv.submission_id
  JOIN exercises x ON x.id = s.exercise_id
  JOIN modules m ON m.id = x.module_id
  JOIN tracks t ON t.id = m.track_id AND t.slug = :'track'
  WHERE rv.status IN ('approved', 'changes_requested')
    AND rv.submitted_at >= (:'ts'::date)::timestamp AT TIME ZONE 'Asia/Ho_Chi_Minh'
    AND rv.submitted_at <  (:'we'::date)::timestamp AT TIME ZONE 'Asia/Ho_Chi_Minh'
  GROUP BY rv.reviewer_id
  HAVING count(DISTINCT date_trunc('week', rv.submitted_at AT TIME ZONE 'Asia/Ho_Chi_Minh')) >= :n
) m;
ROLLBACK;
SQL
```

Vì chỉ dự án cuối khóa cần review, số này thường thấp; cộng thêm số liệu M9 (§4.3) để đánh giá tiêu chí 2.

**Tiêu chí 3** — PR nội dung trong đợt (máy có `gh`, xem [GH-CLI](GH-CLI.md)):

```bash
gh pr list --repo hcmute-dsc/learning-hub-track-sql-30-days --state all \
  --search "created:<TS>..<NGÀY_KẾT_THÚC>" --json author,state,mergedAt \
  -q '.[] | [.author.login, .state] | @tsv'
```

Ban chủ nhiệm tự đối chiếu login với danh sách Ban (trên máy mình, không đăng lên repo) rồi chỉ ghi **số người ngoài Ban** có PR được merge.

## 6. Form phản hồi cuối đợt

Google Form, không bắt buộc đăng nhập, câu trả lời chỉ Ban chủ nhiệm xem. 11 câu, trả lời khoảng 5 phút. Cột cuối cho biết câu hỏi dùng cho tiêu chí nào của CRITIQUE §6.

| # | Câu hỏi | Kiểu trả lời | Tiêu chí |
| --- | --- | --- | --- |
| Q1 | Bạn đã làm khoảng bao nhiêu bài tập trong track SQL? | Chọn: 0 / 1–4 / 5–10 / 11–20 / trên 20 | 1 (đối chiếu với số app) |
| Q2 | Nếu bạn dừng giữa chừng, lý do chính là gì? | Chọn nhiều: bận học / bài khó / không biết hỏi ai / web khó dùng / mất hứng / khác | 1 |
| Q3 | Khi kẹt, bạn có nhận được trợ giúp từ mentor kịp thời không? | Thang 1–5 (1 = không bao giờ, 5 = luôn luôn) | 2 |
| Q4 | Bạn nhận góp ý của mentor nhanh tới mức nào sau khi hỏi hoặc nộp bài? | Chọn: trong ngày / 1–3 ngày / trên 3 ngày / chưa từng nhận | 2 |
| Q5 | Bạn có muốn tự viết hoặc sửa bài học/bài tập (qua PR) không? | Chọn: đã làm / muốn, cần hướng dẫn / không | 3 |
| Q6 | So với học bằng tài liệu + Discord thông thường, web Learning Hub giúp bạn ở điểm nào? | Chọn nhiều: xem tiến độ / biết học gì tiếp / nộp bài một chỗ / mentor góp ý theo bài / không giúp gì / khác | 4 |
| Q7 | Kể 1–3 điều khó chịu nhất khi học trong đợt này (web, nội dung, cách tổ chức…) | Trả lời ngắn | 4 |
| Q8 | Nếu CLB **không** làm tiếp web mà chỉ giữ tài liệu + Discord, bạn sẽ thiếu điều gì? | Trả lời ngắn | 4 |
| Q9 | Bạn có muốn tham gia **làm dev** cho Learning Hub (ít nhất 2 học kỳ) không? | Chọn: có / có thể, cần trao đổi / không | 5 |
| Q10 | Bạn có sẵn sàng **nhận phụ trách** (kế nhiệm tech lead) trong năm tới không? | Chọn: có / có thể / không | 5 |
| Q11 | Bạn có giới thiệu đợt học kế tiếp cho bạn bè không? | Thang 0–10 | Chung (ghi trong báo cáo, không tính vào 5 tiêu chí) |

Ở cuối form thêm một ô **tùy chọn** "GitHub login hoặc Discord (nếu bạn trả lời Có ở Q9/Q10, để Ban liên hệ)". Câu trả lời này chỉ Ban chủ nhiệm đọc, không đưa vào báo cáo.

Gợi ý cách đọc kết quả:

- **Q7 + Q8** là bằng chứng chính cho tiêu chí 4. Gom các câu trả lời thành nhóm nỗi đau (ví dụ: không biết tiến độ, không biết hỏi ai, chờ góp ý lâu, không có chấm tự động); đếm số người nhắc mỗi nhóm.
- **Q9 + Q10** chỉ là tín hiệu. Tiêu chí 5 chỉ "đạt" khi có cam kết trực tiếp với Ban chủ nhiệm (gặp hoặc nhắn riêng), ghi trong biên bản.

## 7. Mẫu báo cáo cho P0-4

Ban chủ nhiệm sao mẫu này vào [docs#2](https://github.com/hcmute-dsc/learning-hub-docs/issues/2) (comment) trước buổi họp. Chỉ ghi số tổng hợp.

**Quy tắc quyết định** (CRITIQUE §6): **đi tiếp** (xây MVP-Lite) khi đạt **ít nhất 3/5** tiêu chí. Nếu không, **dừng** ở repo nội dung + site tĩnh.

```markdown
## Báo cáo đợt dùng thử SQL (P0-3)

- Thời gian: DD/MM/YYYY – DD/MM/YYYY (<N> tuần)
- Người đăng ký (M1 cuối đợt): <số>. Trả lời form: <số> (<%>)
- Mentor tham gia: <số>
- Bảng số liệu tuần: <link comment trong docs#1>

### Kết quả theo 5 tiêu chí

| # | Tiêu chí (CRITIQUE §6) | Số đo được | Nguồn | Đạt? |
| --- | --- | --- | --- | --- |
| 1 | ≥ 50% người đăng ký làm ≥ 5 bài | M5 / M1 = <a>/<b> = <%> | Truy vấn §5.2, form Q1 | Có / Không |
| 2 | ≥ 3 mentor review ≥ 1 bài/tuần suốt đợt | <x> mentor có lượt ở mọi tuần (app: <y>, Discord: <z>) | §5.3, §4.3, form Q3–Q4 | Có / Không |
| 3 | ≥ 2 thành viên ngoài Ban viết/sửa nội dung qua PR | <số> người có PR được merge | §5.3, form Q5 | Có / Không |
| 4 | Người dùng chỉ ra ≥ 3 nỗi đau công cụ hiện tại không giải quyết | <số> nhóm nỗi đau, mỗi nhóm ≥ <k> người nhắc: <liệt kê ngắn> | Form Q6–Q8 | Có / Không |
| 5 | ≥ 3 sinh viên cam kết làm dev ≥ 2 học kỳ + 1 người kế nhiệm | <số> cam kết dev, <số> kế nhiệm | Biên bản gặp trực tiếp, form Q9–Q10 | Có / Không |

**Tổng: <k>/5 tiêu chí đạt → Đề xuất: ĐI TIẾP / DỪNG**

### Diễn biến theo tuần

<2–4 câu: tuần nào đông nhất, tuần nào rơi, vì sao (theo form Q2 và kênh hỏi đáp).>

### Giới hạn của số liệu

- Bài self_check do người học tự đánh giá; số bài tập xong không đo độ đúng.
- Chỉ dự án cuối khóa cần mentor review trong app, nên tiêu chí 2 dùng thêm số ghi tay trên Discord.
- <khác, ví dụ: tuần nghỉ thi, lỗi app làm gián đoạn — ghi ngày>

### Góp ý nổi bật

- <3–5 ý đại diện từ Q7, Q8, viết lại không kèm tên>

### Quyết định của Ban chủ nhiệm (điền trong buổi họp P0-4)

- Ngày họp: DD/MM/YYYY. Thành phần: <chức danh, không cần tên>
- Quyết định: ĐI TIẾP / DỪNG. Lý do: …
- Việc tiếp theo: …
```

## 8. Số chưa đo được trong app hôm nay

| Cần | Hôm nay làm bằng | Issue sẽ thêm vào app |
| --- | --- | --- |
| Số người học, hoạt động 7 ngày, bài chờ review trên một trang | Truy vấn §5.2 do tech lead chạy | [learning-hub#29](https://github.com/hcmute-dsc/learning-hub/issues/29) — trang tổng quan quản trị (E10) |
| Thời gian review (trung vị, lâu nhất) theo tuần | Truy vấn §5.2 (trang **Review** chỉ cho thấy bài đang chờ và bài quá hạn) | Chưa có issue riêng; nên thêm vào learning-hub#29 khi làm |
| Lượt mentor hỗ trợ ngoài app (Discord) | Mentor tự ghi (§4.3) | Không định đưa vào app; MVP-Full có mentorship (E8) |
| Lượt xem bài (đọc mà không đánh dấu) | Không đo | Không làm trong Phase 0 |
| Bài tự đánh giá có đúng không | Không đo | Chấm tự động SQL: E13 `sql-checker` (MVP-Full) |
