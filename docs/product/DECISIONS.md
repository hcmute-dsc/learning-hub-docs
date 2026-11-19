# DECISIONS — Các quyết định kiến trúc

> Mỗi quyết định có: Options · Ưu · Nhược · Quyết định · Lý do. Trạng thái: **ADR-01 … ADR-09 đã chốt (16/11/2026)** — Chủ nhiệm CLB quyết định; bảng options giữ lại làm lịch sử. Nguyên tắc chung: không viết lại chỉ để đồng nhất công nghệ; ưu tiên chi phí bảo trì và dễ onboard thành viên mới.

## ADR-01 — Mở rộng Study Jam hay xây ứng dụng mới?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. Mở rộng study-jam-api/ui | Tận dụng user/role/CI/deploy sẵn; không phải dựng lại | Domain khác (event/task/marking); Java + Ant Design khác stack định hướng; repo không cập nhật từ 2025; thành viên mới ít người biết Spring; phải gỡ nhiều thứ trước khi thêm lesson/submission |
| B. Ứng dụng mới (NestJS + Next.js) | Domain sạch; một ngôn ngữ (TypeScript) cho cả FE/BE/content tooling; khớp stack yêu cầu | Phải dựng auth, CI/CD lại; hai hệ thống cùng tồn tại một thời gian |
| C. Mới nhưng đọc dữ liệu cũ qua API | Giữ dữ liệu Study Jam | Phụ thuộc hệ thống ít được bảo trì |

**Đã chốt (16/11/2026): B** (ứng dụng mới NestJS + Next.js), giữ Study Jam chạy nguyên trạng (KEEP) đến hết khóa hiện tại rồi DEPRECATE.
**Lý do:** study-jam-api chỉ có `User/Event/Task`, không có phần nào của domain học tập; chi phí "tận dụng" thấp hơn chi phí gỡ. Không migrate dữ liệu ở MVP; nếu cần, nhập danh sách thành viên một lần bằng script.

## ADR-02 — mentoring-program độc lập hay thành module của Learning Hub?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. Giữ độc lập | Đang hoạt động tốt, deploy Pages đơn giản, không rủi ro | Hai nơi cho thông tin mentoring |
| B. Gộp vào hub (route `/mentoring`) | Một cổng duy nhất | Phải port React/Vite canvas sang Next; không đem lại giá trị MVP |
| C. Tách đôi: bản đồ độc lập (nội dung), tác vụ mentoring (ghép cặp, review) trong hub | Mỗi bên làm đúng việc | Cần đồng bộ nội dung roadmap |

**Đã chốt (16/11/2026): C** — mentoring-program giữ vai trò "bản đồ chương trình" (KEEP); module `mentorship` của hub xử lý mentor assignment và review. Phase 2: roadmap mentoring là YAML trong content repo, bản đồ đọc từ đó.

## ADR-03 — SQL Challenge và Course-DSC có import vào content repo trung tâm?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. Import cả hai | Một nguồn | Course-DSC không có nội dung → vô nghĩa |
| B. Import SQL Challenge; HTML/CSS (dữ liệu); Course-DSC deprecate | Nội dung thật được tái dùng; chi phí thấp | Cần script convert front-matter |
| C. Không import, hub trỏ link ra ngoài | Không tốn công | Không theo dõi tiến độ/bài nộp được |

**Đã chốt (16/11/2026): B.** SQL Challenge **đã import** vào [hcmute-dsc/learning-hub-content](https://github.com/hcmute-dsc/learning-hub-content) (track `sql-30-days`); dữ liệu HTML/CSS import sau; Course-DSC deprecate. SQL Challenge là lộ trình mẫu (30 bài, "cần biết trước", bài tập, đáp án) — gần như đã đúng mô hình Track → Lesson → Exercise. Import bằng script một lần, giữ lịch sử bằng subtree/ghi `source:` trong front-matter, site VitePress vẫn chạy song song cho đến khi hub đủ dùng.

## ADR-04 — Monorepo?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. Monorepo pnpm (`apps/api`, `apps/web`, `packages/shared`) | Chia sẻ Zod schema/type API; 1 PR cho thay đổi xuyên FE/BE; 1 lần setup lint/CI | CI lâu hơn khi lớn (dùng path filter) |
| B. Nhiều repo (api, web) | Phân quyền tách biệt | Đồng bộ type thủ công; nhiều chi phí cho nhóm nhỏ |

**Đã chốt (16/11/2026): A** cho code (monorepo pnpm); **content ở repo riêng** (ADR-05) vì vòng đời, người sở hữu và quy trình review khác code.

## ADR-05 — Nội dung lưu ở PostgreSQL, Git, hay cả hai?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. Chỉ DB (CMS) | Sửa qua UI | Không review bằng PR; khó đóng góp; mất truy vết |
| B. Chỉ Git, đọc trực tiếp lúc request | Đơn giản | Chậm, rate limit GitHub, khó query/tìm kiếm/ràng buộc khóa ngoại |
| C. **Git là nguồn sự thật + sync sang DB** | PR review, lịch sử, mỗi bản publish gắn commit SHA; DB phục vụ truy vấn nhanh, tiến độ, FK | Cần pipeline sync + validation |

**Đã chốt (16/11/2026): C** (Git là nguồn sự thật, sync sang DB). Tiến độ, bài nộp, review (dữ liệu người dùng) chỉ ở DB. Nội dung (track/module/lesson/exercise) chỉ chỉnh ở Git; DB là bản sao có chỉ mục, lưu `content_version_id` → `commit_sha`. Truy cập repo thông qua interface `ContentSource` (xem CONTENT-SPECIFICATION).

## ADR-06 — Đăng nhập bằng Google, GitHub, hay cả hai?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. GitHub only | Bài nộp qua GitHub cần identity này; thành viên CNTT đều có | Thành viên mới chưa có GitHub phải tạo |
| B. Google only | Quen thuộc, email trường | Phải liên kết GitHub riêng để nộp bài |
| C. **GitHub là chính, liên kết Google/email trường tùy chọn** | Một identity cho bài nộp + review; xác thực thành viên qua email đã xác minh trên GitHub khớp danh sách của CLB | Hai luồng OAuth |

**Đã chốt (16/11/2026): C** (GitHub là chính; liên kết Google/email trường tùy chọn, P2) — MVP chỉ cần GitHub OAuth (hoặc GitHub App user-to-server). Google để Phase 2 cho xác thực `@student.hcmute.edu.vn` nếu CLB muốn. Không lưu mật khẩu. Quyền vào hệ thống (đã chốt 16/11/2026) = tài khoản GitHub có email **đã xác minh** (chính hoặc phụ) nằm trong **danh sách email** của CLB (`approved_emails`); tài khoản khác ở `pending` chờ admin/mentor duyệt tay. Membership org `hcmute-dsc` chỉ là kiểm tra tùy chọn (SECURITY §1).

## ADR-07 — Chấm bài tự động: GitHub Actions, Judge0, Piston, hay khác?

| Option | Ưu | Nhược |
| --- | --- | --- |
| A. **GitHub Actions** — workflow **do CLB sở hữu** (repo `learning-hub-checker`) checkout bài làm tại `head_sha` và chạy test ẩn (**không** chạy workflow nằm trong repo thành viên: họ sửa được để luôn pass — xem CRITIQUE §1.1) | Không tự dựng sandbox; chạy mọi ngôn ngữ trên VM GitHub-hosted tạm; thành viên học PR; an toàn cho server hub; kết quả đáng tin | Chậm hơn (30–120 s); phụ thuộc GitHub; cần GitHub App + webhook; chỉ nhận repo công khai |
| B. Judge0 / Piston (tự host) | Phản hồi nhanh, tích hợp UI | Phải vận hành sandbox (rủi ro bảo mật), tốn tài nguyên, đội sinh viên khó duy trì |
| C. Chạy trong API | — | **Cấm** (không an toàn) |
| D. Quiz/self-check không cần chạy code | Rẻ, cho khái niệm | Không kiểm tra kỹ năng viết code |

**Đã chốt (16/11/2026): A + D ở MVP → ngay sau MVP-Lite thêm checker chỉ cho SQL ("B-lite", thuộc MVP-Full) → B tổng quát chỉ ở Future.**

1. **MVP:** A cho bài `code`/`project` cần chạy test (workflow trong repo `learning-hub-checker` do CLB sở hữu); D cho `quiz` và `self_check` (không cần chạy code).
2. **Ngay sau MVP-Lite — `sql-checker` trên VPS của CLB (MVP-Full).** Lý do: phần lớn nội dung là SQL; 31 bài `self_check` của `sql-30-days` có thể chuyển thành bài chấm tự động, phản hồi tính bằng giây thay vì 30–120 s của Actions. Thiết kế: service/container `sql-checker` riêng với **PostgreSQL riêng** (không phải DB của hub) nạp dataset DSC Shop; API hub gọi qua **mạng nội bộ**; truy vấn chạy bằng role ít quyền trong `BEGIN READ ONLY … ROLLBACK` (hoặc schema/database dùng một lần cho bài sửa dữ liệu), có `statement_timeout`, giới hạn dòng/kích thước kết quả, không mạng ra ngoài; so sánh với tập kết quả kỳ vọng (có thứ tự khi đề yêu cầu `ORDER BY`); kết quả kỳ vọng/lời giải nằm trong `learning-hub-checker` (riêng tư). Chi tiết: ARCHITECTURE §7 (sql-checker), §14; SECURITY §7.
3. **Loại bài:** thêm type `sql` mới hay dùng `code` với `runtime.language: sql` + `checker.kind: sql` — **quyết khi cài đặt**, nghiêng về biến thể spec mới trong `code` (không thêm type).
4. **Judge0/Piston nhiều ngôn ngữ (B tổng quát): chỉ Future, và không bao giờ chạy chung VPS với hub** — chúng cần container đặc quyền (privileged); một lần thoát sandbox sẽ chạm tới DB và secret của hub. Nếu thật sự cần (vd > ~200 bài nộp/tuần ngoài SQL, có người sở hữu `learning-hub-runner`), đặt trên **một VPS nhỏ riêng** (SECURITY §7).

## ADR-08 — Loại khỏi MVP

**Đã chốt (16/11/2026).** Loại: online judge tự dựng nhiều ngôn ngữ (Judge0/Piston — Future, VPS riêng; `sql-checker` chỉ cho SQL là ngoại lệ ở MVP-Full, ADR-07); code editor chạy code trong trình duyệt phía server; gamification ngoài streak/huy hiệu đơn giản (Phase 2) và bảng xếp hạng opt-in theo học kỳ; chat realtime (dùng Discord); thanh toán; multi-tenant; Redis/BullMQ (chỉ thêm khi webhook/Discord cần retry – xem ARCHITECTURE); MinIO/upload file (bài nộp là link GitHub); tìm kiếm toàn văn nâng cao (dùng Postgres `tsvector` khi cần); cổng Google OAuth; tích hợp dữ liệu từ repo private.

**Giữ trong MVP (MVP-Full)** theo quyết định của Chủ nhiệm CLB (16/11/2026), trước đây nằm trong danh sách loại:
- **Bảng xếp hạng** theo "thiết kế an toàn": chỉ hiện thành viên đã **opt-in** (mặc định ẩn); theo học kỳ/đợt và reset mỗi kỳ, không có xếp hạng toàn thời gian; xếp theo số bài hoàn thành (`approved` hoặc tự động `passed`), `self_check` có trọng số thấp hơn bài chấm tự động/mentor duyệt, **không bao giờ** theo tốc độ; hiện top 10 + hạng của chính người xem. Xem CRITIQUE §3.4, FRONTEND-DESIGN §13, DATABASE §6.
- **Thông báo push trên trình duyệt** (Web Push: service worker + khóa VAPID): bài nộp được review, có yêu cầu review mới (mentor), track mới được publish. Discord vẫn là kênh thứ hai. Xem ARCHITECTURE §10, SECURITY §8.

## ADR-09 — Chiến lược repo GitHub (tóm tắt)

**Đã chốt (16/11/2026)** — chấp nhận chiến lược dưới đây.

| Repo | Vai trò | Khi nào tạo |
| --- | --- | --- |
| `learning-hub` | Monorepo code (`apps/api`, `apps/web`, `packages/shared`, `infra/`; MVP-Full thêm `apps/sql-checker` — đề xuất) | Đã tạo (16/11/2026) |
| `learning-hub-content` | Track/module/lesson/exercise (YAML + Markdown), CI validate | Đã tạo (16/11/2026), đã import sql-challenge (`sql-30-days`) |
| `learning-hub-track-<slug>` | Một track nội dung (YAML + Markdown), CI validate; `learning-hub-content` thành registry ghim commit (ADR-10) | Tạo từ `learning-hub-track-template` khi có track mới; `sql-30-days` chuyển sang từ 18/11/2026 |
| `learning-hub-docs` | Bộ tài liệu này; có thể chuyển vào `learning-hub/docs` nếu muốn một repo | Đã tạo (16/11/2026) |
| `learning-hub-checker` | Workflow + test ẩn chấm bài `code` trên GitHub Actions (ADR-07 A); kết quả kỳ vọng/lời giải cho `sql-checker` (MVP-Full); chỉ Ban kỹ thuật có quyền ghi | Cùng lúc với bài `code` đầu tiên |
| `learning-hub-runner` | Sandbox chạy code nhiều ngôn ngữ (Judge0/Piston, **tương lai**); chạy trên VPS riêng, không chung VPS với hub | Chỉ khi cần B tổng quát (ADR-07 mục 4) |
| `<track>-exercises` (vd `git-github-exercises`) | **Một** template repo cho mỗi track có bài `code`, mỗi bài là một thư mục (đã chốt 16/11/2026). Thành viên tạo **một** repo trong tài khoản cá nhân cho cả track; bài nộp không bao giờ tạo repo trong org | Khi track có bài `code` đầu tiên |

**`learning-hub-content` giữ public (đã chốt 17/11/2026).** Lý do: repo không chứa đáp án (đáp án và test ẩn chỉ nằm ở `learning-hub-checker`, private); app tải ảnh bài học thẳng từ `raw.githubusercontent.com` và đồng bộ nội dung không cần token; 28 bài hướng dẫn thành viên clone repo để lấy dữ liệu thực hành (`setup/`); nội dung mang giấy phép mở (CC BY-SA / MIT). Nếu sau này cần giấu một phần (đáp án, đề thi), tách phần đó sang repo private riêng thay vì đổi repo này. Đổi repo này sang private đòi hỏi trước: token chỉ-đọc `CONTENT_GITHUB_TOKEN`, API tự tải ảnh bằng token, và chuyển `setup/` ra repo public.

**Quyền thành viên org `hcmute-dsc` (chốt 19/11/2026, [docs#3](https://github.com/hcmute-dsc/learning-hub-docs/issues/3)): để mở, không siết.** Bản nháp trước định đổi base permission sang *No permission* và tắt quyền tạo repo; Chủ nhiệm CLB quyết định giữ mở để thành viên tự do tạo repo và thử nghiệm. Cài đặt hiện tại:

| Cài đặt (Settings → Member privileges) | Giá trị |
| --- | --- |
| Base permission | **Read**: mọi thành viên đọc được mọi repo của org, kể cả repo private |
| Members can create repositories (public và private) | Bật |
| Members can delete or transfer repositories | Bật: chỉ người có quyền **Admin** trên chính repo đó |
| Members can change repository visibility | Bật: chỉ người có quyền **Admin** trên chính repo đó |

Lớp bảo vệ còn lại: nhánh `main` của các repo Learning Hub cần PR và CI xanh ([WORKFLOW §7.5](../ops/WORKFLOW.md)); thành viên chỉ có Read nên không sửa hay xoá được repo người khác. **Lưu ý khi tạo `learning-hub-checker`:** với base permission Read, mọi thành viên org đọc được test ẩn trong repo private đó. Lúc ấy phải chọn một trong hai: hạ base permission xuống *No permission* rồi cấp quyền qua team, hoặc đặt repo checker ngoài org này. Có thể bật `GITHUB_ORG=hcmute-dsc` để duyệt tài khoản tự động khi mời thành viên vào org.

Chuyển đổi: (1) tạo content repo, import sql-challenge, giữ site cũ; (2) dựng hub; (3) ngừng cập nhật Course-DSC/study-jam-*; (4) cập nhật `.github` profile và website CLB. Không có repo nào bị xóa; chỉ archive sau khi owner đồng ý.

## ADR-10 — Mỗi track một repo, `learning-hub-content` thành registry

**Đã chốt (18/11/2026)** — epic hcmute-dsc/learning-hub#17.

**Bối cảnh.** Track mới (C#, Java…) do mentor có nền tảng khác nhau viết. Với một repo nội dung chung, ai cũng phải clone toàn bộ track, quyền ghi và review chỉ chia được bằng `CODEOWNERS`, và lịch sử, issue của mọi track trộn vào nhau.

**Các lựa chọn.**

| | Cách | Vì sao không/có |
| --- | --- | --- |
| A | Giữ 1 repo, chia người phụ trách bằng `CODEOWNERS` | Ít việc nhất, nhưng mentor vẫn clone mọi thứ; quyền repo không chia theo track được |
| B | Repo chung dùng git submodule trỏ tới repo track | API tarball/tree của GitHub không kèm submodule; submodule khó với người mới |
| C | Mỗi track một repo, hub đọc thẳng nhánh `main` của từng repo | Mentor merge là lên site, không còn bước duyệt chung của Ban chủ nhiệm |
| **D** | **Mỗi track một repo; `learning-hub-content` là registry ghim commit của từng track** | **Chọn** |

**Quyết định (D).**
- Mỗi track nằm trong `hcmute-dsc/learning-hub-track-<slug>`, tạo từ repo template `learning-hub-track-template`. Bên trong giữ đúng cấu trúc `tracks/<slug>/…` (CONTENT-SPECIFICATION §1), nên validator và parser không đổi.
- `learning-hub-content` chỉ còn file `tracks.yaml`: mỗi track ở repo nào, **commit nào được publish**. Publish một track = PR đổi commit trong `tracks.yaml`; CI validate **mọi track cùng nhau**; Ban chủ nhiệm duyệt và merge.
- Hub đọc registry qua `RegistryContentSource` (§8). Một `content_version` vẫn ứng với một commit của registry; mỗi track lưu thêm repo và commit nguồn của nó để footer, link "Sửa trên GitHub" và ảnh trỏ đúng repo track.
- Workflow `validate` và `add-to-project` của repo track gọi workflow dùng chung (reusable workflow) trong `learning-hub`, để sửa một chỗ là mọi track được cập nhật.

**Hệ quả.**
- Mentor chỉ cần repo track của mình; quyền ghi cấp theo repo (team của track).
- Thêm repo cần quản lý: GitHub App `hcmute-dsc-learning-hub` phải được cài vào mỗi repo track mới (WORKFLOW §4).
- Slug/uid trùng giữa các track và prerequisite chéo track chỉ được kiểm tra ở CI của registry, không ở repo track.
- Lịch sử commit của track SQL được giữ khi chuyển sang `learning-hub-track-sql-30-days`; `uid` không đổi nên tiến độ học không mất.
