# CONTENT-SPECIFICATION — Quản lý nội dung bằng Git

> Nội dung học nằm trong **mỗi track một repo** `learning-hub-track-<slug>`, **tách khỏi mã nguồn ứng dụng**; repo `learning-hub-content` là **registry** ghi commit được publish của từng track (DECISIONS ADR-10). Git là nguồn sự thật; PostgreSQL giữ bản sao có chỉ mục (DECISIONS ADR-05). Mọi bản publish truy vết được tới **commit SHA**.

## 1. Cấu trúc repo

Có hai loại repo:

**Repo track** `learning-hub-track-<slug>` (tạo từ template `learning-hub-track-template`). Chứa **đúng một** thư mục `tracks/<slug>/`, với `<slug>` trùng `slug` trong `track.yaml` và trong registry:

```
learning-hub-track-sql-30-days/
  README.md  CONTRIBUTING.md  AGENTS.md  CODEOWNERS  LICENSE
  .github/workflows/validate.yml      # gọi workflow dùng chung trong learning-hub
  tracks/
    sql-30-days/
      track.yaml
      modules/…                       # như cây bên dưới
```

**Registry** `learning-hub-content`:

```
learning-hub-content/
  README.md  CONTRIBUTING.md  CODEOWNERS
  tracks.yaml                         # danh sách track: slug, repo, commit
  .github/workflows/validate.yml      # tải mọi track ở commit đã ghim, validate cùng nhau
```

```yaml
# tracks.yaml
tracks:
  - slug: sql-30-days
    repo: hcmute-dsc/learning-hub-track-sql-30-days
    commit: 0123456789abcdef0123456789abcdef01234567   # 40 ký tự hex, không dùng tên nhánh
```

Cây thư mục của một track (giống nhau ở mọi repo track; trước ADR-10 cả cây này nằm trong `learning-hub-content`):

```
learning-hub-track-<slug>/
  README.md  CONTRIBUTING.md  CODEOWNERS  LICENSE
  schema/                       # (tuỳ chọn, tương lai) JSON Schema sinh từ Zod để editor gợi ý; nguồn chuẩn là Zod trong @hub/content (§4)
    track.schema.json  module.schema.json  lesson.schema.json  exercise.schema.json
  tracks/
    sql-30-days/
      track.yaml
      docs/                             # (tuỳ chọn) trang cấp track, §3.1b
        setup.md                        # trang "Cài đặt"
        resources.md                    # trang "Tài liệu thêm"
      modules/
        01-foundations/
          module.yaml
          lessons/
            01-intro-to-sql.md            # front-matter + Markdown
            03-select-where.md
          exercises/
            select-where-basic/
              exercise.yaml               # metadata + spec
              statement.md                # đề bài
              hints/                      # P2
                01.md
              mentor-notes.md             # tuỳ chọn: ghi chú cho mentor khi review — CÔNG KHAI, xem §3.10
              assets/                     # ảnh, dữ liệu mẫu nhỏ
              # KHÔNG có solution/ ở repo công khai — xem §3.8
    git-github-basics/
      …
  shared/                       # ảnh/dataset dùng chung (nhỏ)
  tools/                        # script import/convert (Node/TS); validate dùng CLI `hub-content-validate` của @hub/content
```

Quy tắc đặt tên: thư mục `kebab-case`; tiền tố số (`01-`) chỉ để sắp xếp trong editor — **thứ tự thật nằm ở `position` hoặc `order` trong YAML** để đổi thứ tự không đổi đường dẫn/ID.

## 2. Định danh ổn định

Mỗi đối tượng có `uid` (**UUID**, khuyến nghị **UUID v7**; sinh một lần, không bao giờ đổi) và `slug` (có thể đổi). `uid` phải là UUID (không dùng ULID) vì DB dùng thẳng `uid` làm khóa chính kiểu `uuid` (`tracks.id = uid`…) ⇒ đổi tên/di chuyển thư mục không làm mất tiến độ. Công cụ `pnpm content:new` (dự kiến) sinh `uid`. Validator từ chối `uid` không phải UUID hoặc trùng trong toàn repo; `uid` bị sửa được phát hiện lúc sync/publish (kiểm tra khóa tự nhiên, §4).

## 3. Schema

Ví dụ ở dạng YAML; schema chính thức là **Zod** trong package `@hub/content` (`packages/content/src/schemas.ts` của monorepo `learning-hub`). Mọi file là **strict**: trường lạ (vd gõ sai tên trường) bị từ chối. Trường **bắt buộc** đánh dấu `*`.

### 3.1 Track — `track.yaml`

```yaml
uid: 0193a000-0000-7000-8000-000000000001   # *  UUID (v7)
slug: sql-30-days                    # *  duy nhất trong repo
title: "30 ngày học SQL"             # *
summary: "Từ SELECT tới window function với PostgreSQL 16."   # *
language: vi                         # *  vi | en
level: beginner                      # *  beginner | intermediate | advanced
estimatedHours: 30
tags: [sql, postgresql, database]
prerequisites: []                    # danh sách track slug khuyến nghị học trước
outcomes:                            # *  "sau khi học xong bạn có thể…"
  - Viết truy vấn SELECT/JOIN/GROUP BY đúng và đọc được kế hoạch thực thi cơ bản
maintainers: [github:tiennhm]        # * ít nhất 1 (xem Ownership)
license: CC-BY-SA-4.0                # *  nội dung
starterRepo: hcmute-dsc/git-github-exercises  # bắt buộc nếu track có bài `code`: MỘT template repo cho cả track
source:                              # nếu import
  repo: hcmute-dsc/sql-challenge
  commit: 0123abc…
position: 10                         # thứ tự trong catalog
status: active                       # active | archived
modules: [01-foundations, 02-aggregation]   # *  thứ tự module (tên thư mục)
```

### 3.1b Trang cấp track — `docs/setup.md`, `docs/resources.md` (learning-hub#44)

Hai file **tuỳ chọn** trong thư mục track, giống trang "Installation" và "Resources" của một track Exercism:

| File | Trang trên site | Viết gì |
| --- | --- | --- |
| `docs/setup.md` | **Cài đặt** (`/tracks/<slug>/setup`) | công cụ cần cài (Docker Desktop, WSL2, PostgreSQL…), từng bước cho Windows/macOS/Linux, và cách **tự kiểm tra** trên máy (lệnh chạy thử, kết quả mong đợi) |
| `docs/resources.md` | **Tài liệu thêm** (`/tracks/<slug>/resources`) | sách, trang tham khảo, video để học sâu hơn |

- Markdown thuần, **không front matter** và không cần tiêu đề `#`: site tự đặt tiêu đề "Cài đặt" / "Tài liệu thêm". Theo cùng luật Markdown như bài học (§4); ảnh tương đối tính từ thư mục `docs/` (ví dụ `../assets/docker-desktop.png`).
- Không có file thì không có trang và không có tab. Trang track chỉ hiện tab "Tổng quan · Cài đặt · Tài liệu thêm" khi track có ít nhất một trang.
- Track có `docs/setup.md`: dashboard nhắc thành viên mở trang **Cài đặt** ngay sau khi enroll, đến khi họ xong mục đầu tiên của track.
- Sync/publish lưu hai trang theo phiên bản nội dung như mọi trường của track (DATABASE `tracks.setup_md`, `tracks.resources_md`). Sửa trang làm track hiện "Sửa" trong bản nháp.
- Ví dụ: `learning-hub/content-example/tracks/sql-sample/docs/` và `tracks/example-track/docs/` trong `learning-hub-track-template`.

### 3.2 Module — `module.yaml`

```yaml
uid: 0193a000-0000-7000-8000-000000000002
slug: foundations
title: "Nền tảng và truy vấn đơn giản"
summary: "Ngày 1–7"
lessons: [01-intro-to-sql, 03-select-where]       # thứ tự lesson (tên file, không .md)
exercises: [select-where-basic, order-limit-basic] # thứ tự exercise (tên thư mục)
```

### 3.3 Lesson — `lessons/*.md`

```markdown
---
uid: 0193a000-0000-7000-8000-000000000003
slug: select-where
title: "SELECT và WHERE: truy vấn và lọc dữ liệu"
estimatedMinutes: 40
objectives:
  - Viết được câu SELECT lấy đúng các cột cần
prerequisites:             # lesson hoặc exercise cần biết trước, dạng <track>/<module>/<slug>
  - sql-30-days/foundations/intro-to-sql
authors: [github:tiennhm]
license: CC-BY-SA-4.0
---

Nội dung Markdown (CommonMark + bảng + khối code có ngôn ngữ). Ảnh dùng đường dẫn tương đối `./assets/…`.
```

Giới hạn: Markdown ≤ 200 KB; ảnh ≤ 500 KB/ảnh (dự kiến, chưa kiểm tra). **Ngoài khối code**, validator từ chối `<script>`, `<iframe>`, `javascript:` và thuộc tính sự kiện inline (`onclick=`…); bên trong khối ``` hoặc inline code thì được phép (vd bài HTML cần minh họa `<script>`). Web vẫn sanitize khi render.

### 3.4 Exercise — `exercise.yaml` + `statement.md`

```yaml
uid: 0193a000-0000-7000-8000-000000000004
slug: select-where-basic
title: "Lọc khách hàng theo thành phố"
type: code                 # *  code | quiz | self_check | project
difficulty: easy           # *  easy | medium | hard
lesson: select-where       # slug lesson liên quan, phải cùng module (tuỳ chọn)
estimatedMinutes: 20
requiresReview: false      # mặc định: true nếu type = project, ngược lại false
prerequisites:             # chỉ exercise, dạng <track>/<module>/<slug>; phải hoàn thành trước (DAG, không vòng)
  - sql-30-days/foundations/intro-check
tags: [select, where]
authors: [github:tiennhm]
license: CC-BY-SA-4.0
spec:                      # *  tuỳ theo type, xem 3.5
  ...
```

`statement.md` **bắt buộc** cho mọi exercise (kể cả `quiz`).

### 3.5 `spec` theo loại

**`code`** (chấm bằng `learning-hub-checker`):
```yaml
spec:
  starterPath: select-where-basic       # thư mục của bài này trong `starterRepo` của track
  checker:
    workflow: check-exercise.yml            # workflow trong learning-hub-checker
    tests: exercises/sql-30-days/select-where-basic   # thư mục test ẩn trong checker repo
  runtime: { language: sql, timeoutSeconds: 120 }               # timeoutSeconds ≤ 600, mặc định 120
  submit: { kind: github_pr, allowedPaths: ["solution.sql"] }   # kind: github_pr | github_commit; allowedPaths: file thành viên được sửa
```

**Repo mẫu theo track (đã chốt 16/11/2026):** mỗi track có bài `code` khai báo **một** `starterRepo` trong `track.yaml` (vd `hcmute-dsc/git-github-exercises`), mỗi bài `code` là một thư mục `starterPath` trong repo đó. Thành viên bấm "Use this template" **một lần** cho cả track, repo nằm trong **tài khoản cá nhân** (công khai), làm từng bài trong thư mục của bài và mở PR. Org CLB chỉ chứa repo mẫu, nên số repo trong org không tăng theo số bài nộp. `submit.allowedPaths` tính từ `starterPath`. Validator báo lỗi nếu track có bài `code` mà thiếu `starterRepo`.

**`quiz`**:
```yaml
spec:
  shuffle: true               # mặc định false
  passScore: 0.7              # 0–1, mặc định 0.7
  questions:                  # ≥ 1 câu; `id` duy nhất trong bài
    - id: q1
      type: single            # single | multiple | short_text
      prompt: "`NULL = NULL` trả về gì?"
      options: [TRUE, FALSE, NULL]   # ≥ 2 option
      answer: 2               # single: 1 chỉ số; khi chấm ở server, KHÔNG gửi `answer` xuống client
      explanation: "So sánh với NULL cho kết quả NULL (logic ba trị)."
    - id: q2
      type: multiple
      prompt: "Mệnh đề nào dùng để lọc dữ liệu?"
      options: [WHERE, HAVING, ORDER BY]
      answer: [0, 1]          # multiple: mảng chỉ số
    - id: q3
      type: short_text        # không có options
      prompt: "Từ khóa nào sắp xếp kết quả?"
      answer: ["ORDER BY", "order by"]   # short_text: mảng chuỗi được chấp nhận
```

**`self_check`**: `spec: { checklist: [..] }` (≥ 1 mục) — member tự đánh dấu hoàn thành; đáp án gập nằm trong `statement.md` (§3.8).

**`project`**: `spec: { deliverable: github_repo_or_pr, rubric: [ {criterion: "...", weight: 2} ] }` — `deliverable`: `github_repo_or_pr` hoặc `link`; `rubric` ≥ 1 mục, `weight` số nguyên 1–100 (mặc định 1; có thể là điểm, ví dụ rubric 100 điểm của dự án cuối khóa SQL). Mentor đánh giá theo rubric.

### 3.6 Test cases (cho `code`)
Test ẩn nằm trong `learning-hub-checker` (repo riêng, chỉ Ban kỹ thuật có quyền ghi) để thành viên không đọc/sửa được. Định dạng test do exercise quyết định (vd so sánh kết quả truy vấn với tập kỳ vọng); ví dụ test hiển thị công khai (2–3 ca) đặt trong `statement.md` hoặc `examples:` trong `exercise.yaml`. Tác giả content gửi test qua PR vào checker repo (review bởi CODEOWNERS), liên kết bằng `spec.checker.tests`.

### 3.7 Hints (P2, learning-hub#46)
Mỗi hint là một file `hints/NN.md` trong thư mục exercise: `01.md`, `02.md`… đánh số liên tục từ `01`, tối đa 10 hint. Thành viên mở **lần lượt** từng hint. Front matter không bắt buộc:

```markdown
---
cost: 1        # số lần nộp tối thiểu trước khi mở được hint này; mặc định 0
---

Dùng `WHERE` để giữ lại các dòng có `city` đúng bằng chuỗi cần tìm.
```

`cost` là "mềm": mở hint **không** trừ điểm, hub chỉ ghi lại ai đã mở hint nào (DATABASE `hint_unlocks`). Hint theo cùng luật Markdown như bài học (§4). Ví dụ: `content-example/tracks/sql-sample/.../select-where-basic/hints/`.

### 3.8 Solutions (P2)
Repo content là **công khai** (khuyến khích đóng góp) ⇒ lời giải và test ẩn **không** được đặt ở đây (validator báo lỗi nếu thấy `solution.md` hay thư mục `solution/` trong thư mục exercise). Chúng nằm ở `learning-hub-checker` (**riêng tư**): `exercises/<track-slug>/<exercise-slug>/{tests/, solution.md}`. Hai exercise cùng slug trong một track thì lời giải bị coi là mơ hồ (lỗi validate): đặt slug khác nhau.

- API đọc lời giải lúc sync và publish, ở commit mới nhất của nhánh `SOLUTIONS_BRANCH` (commit ghi vào `content_versions.summary.solutionsFrom`). Repo checker không được ghim trong `tracks.yaml`, nên chỉ sửa lời giải thì lời giải mới lên site cùng lần publish nội dung kế tiếp.
- Hub chỉ trả lời giải cho member **khi đã hoàn thành bài** (trạng thái `completed`: bài đạt, hoặc được mentor duyệt nếu bài cần review). Nộp mà chưa đạt thì chưa thấy (CLB chốt trên learning-hub#53). Mentor/admin luôn xem được (SECURITY §2). Không muốn có lời giải cho bài nào thì không tạo `solution.md` cho bài đó.
- Dev/test dùng `learning-hub/checker-example/` (cùng cấu trúc) với `SOLUTIONS_SOURCE=local`.

Với `self_check`, đáp án gập cần hiển thị ngay nên nằm trong `statement.md` công khai (chấp nhận được vì không chấm điểm).

### 3.9 Prerequisites
Hai tầng: `lesson.prerequisites` (khuyến nghị, hiển thị; trỏ tới lesson **hoặc** exercise) và `exercise.prerequisites` (bắt buộc, khóa bài; **chỉ** trỏ tới exercise). Tham chiếu theo đường dẫn logic `<track>/<module>/<slug>` (slug, không phải tên thư mục). Đồ thị prerequisite của exercise phải là **DAG** (validator kiểm tra vòng).

### 3.10 Ghi chú cho mentor — `mentor-notes.md` (tuỳ chọn, learning-hub#43)
Mỗi exercise có thể có file `mentor-notes.md` cạnh `statement.md` (giống "mentor notes" của Exercism): những điểm cần xem khi review, lỗi hay gặp, cách góp ý. Nhờ đó các mentor review đồng đều, mentor mới đỡ bỡ ngỡ.

```markdown
## Khi review bài này
- PR chỉ sửa `ABOUT.md`; file khác bị sửa thì nhắc thành viên tách ra.

## Lỗi hay gặp
- Mở PR vào repo template gốc thay vì repo của mình.
```

- Markdown thuần, **không** front matter; cùng luật Markdown như bài học (§4). File rỗng là lỗi validate: không cần ghi chú thì đừng tạo file.
- Sync/publish lưu ghi chú theo phiên bản nội dung như các trường khác (DATABASE `exercises.mentor_notes_md`). Sửa ghi chú làm exercise được tính là "đã đổi" trong diff.
- Hub chỉ hiện ghi chú trên trang review `/review/[submissionId]` cho `mentor`/`admin`; API không trả ghi chú cho member (SECURITY §2).
- **Lưu ý: repo track công khai, nên ghi chú cũng công khai**: ai cũng đọc được trên GitHub. Hub chỉ ẩn ghi chú khỏi giao diện thành viên, **không** giữ bí mật. **Không đặt đáp án, lời giải hay test cần giấu ở đây**: chúng thuộc repo checker riêng tư (§3.6, §3.8). Ghi chú nên nói *cần xem gì*, không nói *đáp án là gì*.
- Ví dụ: `content-example/tracks/git-github-basics/.../first-pull-request/mentor-notes.md` và bài mẫu trong `learning-hub-track-template`.

## 4. Validation

Validator/parser nằm trong monorepo ứng dụng, package **`@hub/content`** (`packages/content`), dùng **Zod** làm nguồn sự thật cho schema (JSON Schema trong `schema/` chỉ là tuỳ chọn/tương lai). Cùng một code chạy ở ba nơi:

- **CLI** `hub-content-validate [dir]` (mặc định thư mục hiện tại): in mọi lỗi theo file, **exit 1** nếu có lỗi ⇒ CI của `learning-hub-content` chạy nó để chặn PR; contributor chạy cục bộ.
- **API**: module `content` gọi parser khi sync và khi publish (BACKEND-DESIGN §2).
- **Dev/test**: cây nội dung mẫu nhỏ ở `learning-hub/content-example` (cùng cấu trúc §1).

Validator không dừng ở lỗi đầu tiên: trả về **toàn bộ** lỗi (đường dẫn file + thông điệp) để contributor sửa một lần.

| Kiểm tra | Lỗi nếu | Trạng thái |
| --- | --- | --- |
| Schema | thiếu trường bắt buộc, sai enum/kiểu, **trường lạ** (strict); YAML/front-matter hỏng | Đã làm |
| `uid` | không phải UUID, hoặc trùng trong toàn repo | Đã làm |
| `slug` | track slug trùng trong repo; module/lesson/exercise slug trùng trong cùng cha | Đã làm |
| Tham chiếu | module/lesson/exercise được liệt kê nhưng không tồn tại; `exercise.lesson` không phải lesson cùng module; `lesson.prerequisites` không trỏ tới lesson/exercise nào; `exercise.prerequisites` không trỏ tới exercise | Đã làm |
| Prerequisite graph | đồ thị prerequisite của exercise có vòng (phải là DAG) | Đã làm |
| File | file mồ côi (module/lesson/exercise có trong thư mục nhưng không được liệt kê); exercise thiếu `statement.md` | Đã làm |
| Nội dung | Markdown > 200 KB; `<script>`, `<iframe>`, `javascript:`, thuộc tính sự kiện inline **ngoài khối code** | Đã làm |
| Quiz | `answer` ngoài phạm vi option, ít hơn 2 option, `id` câu hỏi trùng | Đã làm |
| Hint | tên file không phải `NN.md`; số thứ tự có chỗ trống; quá 10 hint; hint rỗng; front matter sai (`cost` ngoài 0–100, trường lạ) | Đã làm |
| Trang track | file `.md` khác `setup.md`/`resources.md` trong `docs/` của track (thường do gõ sai tên); trang rỗng; trang có front matter; Markdown vi phạm luật nội dung | Đã làm |
| Lời giải | `solution.md`/`solution/` nằm trong repo nội dung công khai; lời giải trong repo checker trùng hai exercise cùng slug; Markdown của lời giải vi phạm luật nội dung | Đã làm |
| Ghi chú mentor | `mentor-notes.md` rỗng, có front matter, hoặc vi phạm luật Markdown (§3.10) | Đã làm |
| Khóa tự nhiên (lúc sync/publish, cần DB) | item mới dùng lại khóa tự nhiên (cha + slug, hoặc track slug) của bản ghi đang có với `id` khác ⇒ thường do `uid` bị sửa | Đã làm (API) |
| Link | link nội bộ gãy; link ngoài (cảnh báo, không fail) | Dự kiến — bước CI của content repo |
| Secret | secret lọt vào nội dung (gitleaks) | Dự kiến — bước CI của content repo |
| Ảnh | ảnh > 500 KB | Dự kiến |
| Giấy phép | thiếu `license`/`authors` khi có `source` (nội dung import phải giữ giấy phép gốc, §10) | Dự kiến |
| Code exercise | `spec.checker` tồn tại trong checker repo (job CI chéo repo, tuỳ chọn) | Dự kiến |

Kết quả lưu trong `content_versions.summary`: diff khi `draft`, danh sách lỗi khi `rejected` (§5).

## 5. Phiên bản & truy vết

- Với registry, `C` là commit của `learning-hub-content`; mỗi track còn lưu **repo và commit nguồn** của nó (từ `tracks.yaml`) để footer, link "Sửa trên GitHub" và ảnh trỏ đúng repo track.
- Mỗi sync tại commit `C` tạo `content_versions(commit_sha = C)`: `draft` (kèm diff thêm/sửa/xóa/không đổi theo loại) nếu hợp lệ, `rejected` (kèm danh sách lỗi) nếu không. Sync idempotent theo `(source_repo, commit_sha)` và **không ghi** bảng catalog.
- `published` chỉ do admin bấm publish (MVP-Lite: maintainer chạy CLI `content:sync --publish`): đọc lại nguồn **tại đúng commit SHA đã ghim**, validate lại, rồi ghi catalog trong một giao dịch (USER-FLOWS §10).
- Đối tượng nội dung lưu `content_version_id` (version **published** gần nhất thay đổi nó) và `content_hash` (sha256 nội dung chuẩn hóa) để publish chỉ upsert bản ghi mới/đổi. Item biến mất khỏi Git → `status = archived`, không xóa cứng.
- `submissions.content_version_id` lưu version lúc nộp ⇒ biết bài được chấm theo đề nào.
- Release có nhãn người đọc được: tag Git `v2027.03.1` (CalVer) trên content repo; `content_versions.summary.tag` lưu tag nếu có.
- Hiển thị cuối mỗi lesson: "Cập nhật lần cuối: `<ngày>` · `<sha ngắn>` · Sửa trên GitHub".
- Đổi **phá vỡ** (xóa exercise đã có người nộp, đổi tiêu chí chấm): CI gắn nhãn `breaking`; hub không xóa mà `archived`; thông báo mentor.

## 6. Quy trình Pull Request

**Trong repo track** (mentor của track làm):

1. Fork/branch từ `main`, viết nội dung, chạy `hub-content-validate .` (CLI của `@hub/content`) + `pnpm content:preview` (dự kiến).
2. Mở PR dùng template (mục đích, loại thay đổi, kiểm tra tự làm, có breaking không).
3. CI bắt buộc: validate (`hub-content-validate`); link check, gitleaks, build preview (dự kiến, chưa làm).
4. Review: ≥ 1 maintainer của track (theo `CODEOWNERS`); thay đổi `exercise` loại `code` cần thêm 1 người ở Ban kỹ thuật (liên quan test ẩn).
5. Merge vào `main` (squash). Branch protection: không push trực tiếp.

**Publish qua registry** (Ban chủ nhiệm duyệt):

6. Mở PR vào `learning-hub-content` đổi `commit` của track trong `tracks.yaml` sang commit mới trên `main` của repo track. Mô tả PR tóm tắt thay đổi (link các PR của repo track).
7. CI của registry tải **mọi** track ở commit đã ghim và validate cùng nhau (slug/uid trùng giữa các track, prerequisite chéo track).
8. Ban chủ nhiệm duyệt, squash merge → hub tạo `content_version` draft (webhook, nút Sync, hoặc CLI `content:sync` ở MVP-Lite) → admin xem diff và publish.

## 7. Sở hữu nội dung

| Cấp | Chủ sở hữu | Quyền |
| --- | --- | --- |
| Registry `learning-hub-content` | Ban kỹ thuật + Chủ nhiệm | duyệt PR publish (đổi commit trong `tracks.yaml`), quản lý checker |
| Repo track | team của track (mentor) | admin repo track; Ban kỹ thuật giữ quyền admin để bàn giao |
| Track | `maintainers` trong `track.yaml` (CODEOWNERS sinh/kiểm tra tự động) | duyệt PR của track |
| Bài viết | `authors` | ghi nhận công; được sửa bài mình |
Khi maintainer rời CLB: Ban kỹ thuật gán lại trong cùng PR dọn dẹp học kỳ; track không có maintainer **không được publish bản mới**.

## 8. Lớp trừu tượng `ContentSource`

Ứng dụng không phụ thuộc cấu trúc GitHub; chỉ phụ thuộc interface:

```ts
interface ContentSource {
  /** commit hiện tại của nhánh được cấu hình */
  getHead(): Promise<{ sha: string; committedAt: Date }>;
  /** danh sách file (đường dẫn tương đối) tại sha */
  listFiles(sha: string): Promise<string[]>;
  /** đọc 1 file tại sha */
  readFile(sha: string, path: string): Promise<Buffer>;
}
```

Bộ phân tích (`parseContent()` trong `@hub/content`) nhận nội dung file đọc qua `ContentSource` và trả về mô hình nội dung (`ContentTree`) + danh sách lỗi — **không biết** GitHub; module `content` của API nối hai phần này. Triển khai:

| Triển khai | Dùng khi | Ghi chú |
| --- | --- | --- |
| `LocalDirContentSource` | dev/test (vd `content-example`) | commit SHA = sha1 trên danh sách đường dẫn (đã sắp xếp) + nội dung file; publish **báo lỗi rõ** nếu thư mục đã đổi kể từ lúc sync (SHA không còn khớp) |
| `GithubContentSource` | MVP | đọc repo/branch cấu hình qua GitHub REST: `commits/{branch}` (HEAD), `git/trees/{sha}?recursive=1`, `contents/{path}?ref={sha}`; chạy không cần token với repo công khai, token tuỳ chọn để nâng rate limit; đích dài hạn là GitHub App (SECURITY §10) |
| `RegistryContentSource` | production sau ADR-10 | đọc `tracks.yaml` của registry tại HEAD, rồi đọc `tracks/<slug>/…` của từng repo track tại commit đã ghim (cùng các gọi REST như `GithubContentSource`); báo lỗi khi repo track chứa thư mục track khác slug, commit không đủ 40 ký tự hex, hoặc slug trùng |
| `GitlabContentSource`, `TarballContentSource` | tương lai | |

Cấu hình (env): `CONTENT_SOURCE=local|github|registry` (`registry` đọc `CONTENT_REPO` như registry), `CONTENT_LOCAL_DIR`, `CONTENT_REPO` (vd `hcmute-dsc/learning-hub-content`), `CONTENT_BRANCH`, `CONTENT_GITHUB_TOKEN` (tuỳ chọn).

Thay đổi cấu trúc thư mục chỉ ảnh hưởng parser + schema trong `@hub/content`, và có `contentSpecVersion` trong `tracks/…` hoặc file gốc `content.config.yaml` để hỗ trợ nhiều phiên bản schema.

## 9. Import tài liệu hiện có

| Nguồn | Cách import | Việc cần làm | Trạng thái |
| --- | --- | --- | --- |
| `sql-challenge/lessons/*.md` (30 bài + 3 ôn tập) | Script `tools/import-sql-challenge`: mỗi file → `lessons/NN-slug.md` giữ nguyên Markdown, tự sinh front-matter (`title` từ H1, `objectives` từ "Sau bài này bạn có thể", `prerequisites` từ "Cần biết trước"); ngày ôn tập → module riêng | Tách bài tập ở cuối mỗi bài thành `exercise.yaml` (`self_check` trước, `code` cho một số bài chọn lọc); chuyển ảnh `lessons/img` vào `assets/`; dataset DSC Shop nằm ở `tracks/sql-30-days/setup` (bài SQL không cần repo nộp bài) | Verified (cấu trúc bài); phần bài tập cần đọc từng bài |
| `html-css/src/data/exercises.json` | Script convert → `exercise.yaml` | Cần đọc file để ánh xạ trường; xác nhận nguồn/tác giả trong mô tả PR (§10) | Partially verified |
| `mentoring-program/src/diagrams/*.ts` | Giữ nguyên ở MVP; P2 chuyển roadmap thành YAML | — | Verified |
| `it-quiz` | Ánh xạ ngân hàng câu hỏi → `quiz` | Xác nhận định dạng; xác nhận nguồn/tác giả trong mô tả PR (§10) | Unknown |
| Study Jam Git & GitHub (slide/Tài liệu) | Viết lại thành track `git-github-basics` | Tìm tài liệu gốc | Requires human confirmation |

Quy trình import: chạy script → PR → validate → maintainer rà soát từng track → publish. Ghi `source.repo/commit` để truy vết. Không xóa site VitePress cho đến khi hub phục vụ track tương đương.

## 10. Giấy phép
**Đã chốt (16/11/2026, giao tech lead quyết):**
- Nội dung **import giữ giấy phép gốc**: sql-challenge là MIT → track `sql-30-days` dùng `license: MIT`, giữ ghi nhận tác giả (`authors`) và link nguồn (`source`).
- Nội dung **mới** mặc định **CC BY-SA 4.0** cho văn bản/hình, **MIT** cho mã mẫu (ghi tại `LICENSE` và từng file qua `license`).
- Trước khi import tài liệu bên thứ ba (it-quiz, dữ liệu html-css, …), người import **xác nhận nguồn/tác giả trong mô tả PR**; maintainer không merge nếu thiếu.
