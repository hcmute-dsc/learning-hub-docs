# CONTENT-SPECIFICATION — Quản lý nội dung bằng Git

> Nội dung học nằm trong repo riêng `learning-hub-content`, **tách khỏi mã nguồn ứng dụng**. Git là nguồn sự thật; PostgreSQL giữ bản sao có chỉ mục (DECISIONS ADR-05). Mọi bản publish truy vết được tới **commit SHA**.

## 1. Cấu trúc repo

```
learning-hub-content/
  README.md  CONTRIBUTING.md  CODEOWNERS  LICENSE
  schema/                       # (tuỳ chọn, tương lai) JSON Schema sinh từ Zod để editor gợi ý; nguồn chuẩn là Zod trong @hub/content (§4)
    track.schema.json  module.schema.json  lesson.schema.json  exercise.schema.json
  tracks/
    sql-30-days/
      track.yaml
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
source:                              # nếu import
  repo: hcmute-dsc/sql-challenge
  commit: 0123abc…
position: 10                         # thứ tự trong catalog
status: active                       # active | archived
modules: [01-foundations, 02-aggregation]   # *  thứ tự module (tên thư mục)
```

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
  starterTemplate: hcmute-dsc/sql-select-where-template   # repo template (công khai)
  checker:
    workflow: check-exercise.yml            # workflow trong learning-hub-checker
    tests: exercises/sql-30-days/select-where-basic   # thư mục test ẩn trong checker repo
  runtime: { language: sql, timeoutSeconds: 120 }               # timeoutSeconds ≤ 600, mặc định 120
  submit: { kind: github_pr, allowedPaths: ["solution.sql"] }   # kind: github_pr | github_commit; allowedPaths: file thành viên được sửa
```

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

**`project`**: `spec: { deliverable: github_repo_or_pr, rubric: [ {criterion: "...", weight: 2} ] }` — `deliverable`: `github_repo_or_pr` hoặc `link`; `rubric` ≥ 1 mục, `weight` số nguyên 1–10 (mặc định 1). Mentor đánh giá theo rubric.

### 3.6 Test cases (cho `code`)
Test ẩn nằm trong `learning-hub-checker` (repo riêng, chỉ Ban kỹ thuật có quyền ghi) để thành viên không đọc/sửa được. Định dạng test do exercise quyết định (vd so sánh kết quả truy vấn với tập kỳ vọng); ví dụ test hiển thị công khai (2–3 ca) đặt trong `statement.md` hoặc `examples:` trong `exercise.yaml`. Tác giả content gửi test qua PR vào checker repo (review bởi CODEOWNERS), liên kết bằng `spec.checker.tests`.

### 3.7 Hints (P2)
`hints/NN.md`, mở dần: mỗi hint có `cost` mềm (số lần thử tối thiểu trước khi mở). MVP: không có.

### 3.8 Solutions (P2)
Repo content là **công khai** (khuyến khích đóng góp) ⇒ lời giải và test ẩn **không** được đặt ở đây. Chúng nằm ở `learning-hub-checker` (**riêng tư**): `exercises/<track>/<exercise>/{tests/, solution.md}`. Hub chỉ trả lời giải cho member **sau khi** exercise `approved/passed` (và có thể tắt theo bài). Với `self_check`, đáp án gập cần hiển thị ngay nên nằm trong `statement.md` công khai (chấp nhận được vì không chấm điểm).

### 3.9 Prerequisites
Hai tầng: `lesson.prerequisites` (khuyến nghị, hiển thị; trỏ tới lesson **hoặc** exercise) và `exercise.prerequisites` (bắt buộc, khóa bài; **chỉ** trỏ tới exercise). Tham chiếu theo đường dẫn logic `<track>/<module>/<slug>` (slug, không phải tên thư mục). Đồ thị prerequisite của exercise phải là **DAG** (validator kiểm tra vòng).

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
| Khóa tự nhiên (lúc sync/publish, cần DB) | item mới dùng lại khóa tự nhiên (cha + slug, hoặc track slug) của bản ghi đang có với `id` khác ⇒ thường do `uid` bị sửa | Đã làm (API) |
| Link | link nội bộ gãy; link ngoài (cảnh báo, không fail) | Dự kiến — bước CI của content repo |
| Secret | secret lọt vào nội dung (gitleaks) | Dự kiến — bước CI của content repo |
| Ảnh | ảnh > 500 KB | Dự kiến |
| Giấy phép | thiếu `license`/`authors` khi có `source` | Dự kiến |
| Code exercise | `spec.checker` tồn tại trong checker repo (job CI chéo repo, tuỳ chọn) | Dự kiến |

Kết quả lưu trong `content_versions.summary`: diff khi `draft`, danh sách lỗi khi `rejected` (§5).

## 5. Phiên bản & truy vết

- Mỗi sync tại commit `C` tạo `content_versions(commit_sha = C)`: `draft` (kèm diff thêm/sửa/xóa/không đổi theo loại) nếu hợp lệ, `rejected` (kèm danh sách lỗi) nếu không. Sync idempotent theo `(source_repo, commit_sha)` và **không ghi** bảng catalog.
- `published` chỉ do admin bấm publish (MVP-Lite: maintainer chạy CLI `content:sync --publish`): đọc lại nguồn **tại đúng commit SHA đã ghim**, validate lại, rồi ghi catalog trong một giao dịch (USER-FLOWS §10).
- Đối tượng nội dung lưu `content_version_id` (version **published** gần nhất thay đổi nó) và `content_hash` (sha256 nội dung chuẩn hóa) để publish chỉ upsert bản ghi mới/đổi. Item biến mất khỏi Git → `status = archived`, không xóa cứng.
- `submissions.content_version_id` lưu version lúc nộp ⇒ biết bài được chấm theo đề nào.
- Release có nhãn người đọc được: tag Git `v2027.03.1` (CalVer) trên content repo; `content_versions.summary.tag` lưu tag nếu có.
- Hiển thị cuối mỗi lesson: "Cập nhật lần cuối: `<ngày>` · `<sha ngắn>` · Sửa trên GitHub".
- Đổi **phá vỡ** (xóa exercise đã có người nộp, đổi tiêu chí chấm): CI gắn nhãn `breaking`; hub không xóa mà `archived`; thông báo mentor.

## 6. Quy trình Pull Request

1. Fork/branch từ `main`, viết nội dung, chạy `hub-content-validate .` (CLI của `@hub/content`) + `pnpm content:preview` (dự kiến).
2. Mở PR dùng template (mục đích, loại thay đổi, kiểm tra tự làm, có breaking không).
3. CI bắt buộc: validate (`hub-content-validate`); link check, gitleaks, build preview (dự kiến, chưa làm).
4. Review: ≥ 1 maintainer của track (theo `CODEOWNERS`); thay đổi `exercise` loại `code` cần thêm 1 người ở Ban kỹ thuật (liên quan test ẩn).
5. Merge vào `main` (squash). Branch protection: không push trực tiếp.
6. Webhook `push` (hoặc admin bấm Sync, hoặc maintainer chạy CLI `content:sync` ở MVP-Lite) → hub tạo `content_version` draft → admin xem diff và publish.

## 7. Sở hữu nội dung

| Cấp | Chủ sở hữu | Quyền |
| --- | --- | --- |
| Repo | Ban kỹ thuật + Chủ nhiệm | admin repo, quản lý checker |
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
| `GitlabContentSource`, `TarballContentSource` | tương lai | |

Cấu hình (env): `CONTENT_SOURCE=local|github`, `CONTENT_LOCAL_DIR`, `CONTENT_REPO` (vd `hcmute-dsc/learning-hub-content`), `CONTENT_BRANCH`, `CONTENT_GITHUB_TOKEN` (tuỳ chọn).

Thay đổi cấu trúc thư mục chỉ ảnh hưởng parser + schema trong `@hub/content`, và có `contentSpecVersion` trong `tracks/…` hoặc file gốc `content.config.yaml` để hỗ trợ nhiều phiên bản schema.

## 9. Import tài liệu hiện có

| Nguồn | Cách import | Việc cần làm | Trạng thái |
| --- | --- | --- | --- |
| `sql-challenge/lessons/*.md` (30 bài + 3 ôn tập) | Script `tools/import-sql-challenge`: mỗi file → `lessons/NN-slug.md` giữ nguyên Markdown, tự sinh front-matter (`title` từ H1, `objectives` từ "Sau bài này bạn có thể", `prerequisites` từ "Cần biết trước"); ngày ôn tập → module riêng | Tách bài tập ở cuối mỗi bài thành `exercise.yaml` (`self_check` trước, `code` cho một số bài chọn lọc); chuyển ảnh `lessons/img` vào `assets/`; dataset DSC Shop dùng `setup/init` làm `starterTemplate` | Verified (cấu trúc bài); phần bài tập cần đọc từng bài |
| `html-css/src/data/exercises.json` | Script convert → `exercise.yaml` | Cần đọc file để ánh xạ trường; xác nhận nguồn/bản quyền | Partially verified |
| `mentoring-program/src/diagrams/*.ts` | Giữ nguyên ở MVP; P2 chuyển roadmap thành YAML | — | Verified |
| `it-quiz` | Ánh xạ ngân hàng câu hỏi → `quiz` | Xác nhận định dạng/nguồn | Unknown |
| Study Jam Git & GitHub (slide/Tài liệu) | Viết lại thành track `git-github-basics` | Tìm tài liệu gốc | Requires human confirmation |

Quy trình import: chạy script → PR → validate → maintainer rà soát từng track → publish. Ghi `source.repo/commit` để truy vết. Không xóa site VitePress cho đến khi hub phục vụ track tương đương.

## 10. Giấy phép
Mặc định đề xuất: **CC BY-SA 4.0** cho văn bản/hình, **MIT** cho mã mẫu (ghi tại `LICENSE` và từng file qua `license`). Sql-challenge hiện là MIT; giữ ghi nhận tác giả. **Cần Ban chủ nhiệm quyết định** (CRITIQUE §8).
