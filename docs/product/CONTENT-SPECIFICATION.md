# CONTENT-SPECIFICATION — Quản lý nội dung bằng Git

> Nội dung học nằm trong repo riêng `learning-hub-content`, **tách khỏi mã nguồn ứng dụng**. Git là nguồn sự thật; PostgreSQL giữ bản sao có chỉ mục (DECISIONS ADR-05). Mọi bản publish truy vết được tới **commit SHA**.

## 1. Cấu trúc repo

```
learning-hub-content/
  README.md  CONTRIBUTING.md  CODEOWNERS  LICENSE
  schema/                       # JSON Schema cho từng loại file (nguồn chuẩn)
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
  tools/                        # CLI validate, convert (Node/TS)
```

Quy tắc đặt tên: thư mục `kebab-case`; tiền tố số (`01-`) chỉ để sắp xếp trong editor — **thứ tự thật nằm ở `position` hoặc `order` trong YAML** để đổi thứ tự không đổi đường dẫn/ID.

## 2. Định danh ổn định

Mỗi đối tượng có `uid` (ULID/UUID sinh một lần, không bao giờ đổi) và `slug` (có thể đổi). DB dùng `uid` làm khóa chính (`tracks.id`…) ⇒ đổi tên/di chuyển thư mục không làm mất tiến độ. Công cụ `pnpm content:new` sinh `uid`; CI từ chối `uid` trùng hoặc bị sửa so với bản trước.

## 3. Schema

Ví dụ ở dạng YAML; JSON Schema chính thức nằm trong `schema/`. Trường **bắt buộc** đánh dấu `*`.

### 3.1 Track — `track.yaml`

```yaml
uid: 01JC2ZK6V3Q8X0A1B2C3D4E5F6      # *
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
uid: …
slug: foundations
title: "Nền tảng và truy vấn đơn giản"
summary: "Ngày 1–7"
lessons: [01-intro-to-sql, 03-select-where]       # thứ tự lesson (tên file, không .md)
exercises: [select-where-basic, order-limit-basic] # thứ tự exercise (tên thư mục)
```

### 3.3 Lesson — `lessons/*.md`

```markdown
---
uid: 01JC…
slug: select-where
title: "SELECT và WHERE: truy vấn và lọc dữ liệu"
estimatedMinutes: 40
objectives:
  - Viết được câu SELECT lấy đúng các cột cần
prerequisites:             # lesson/exercise cần biết trước (tham chiếu theo slug đầy đủ)
  - sql-30-days/foundations/intro-to-sql
authors: [github:tiennhm]
license: CC-BY-SA-4.0
---

Nội dung Markdown (CommonMark + bảng + khối code có ngôn ngữ). Ảnh dùng đường dẫn tương đối `./assets/…`.
```

Giới hạn: body ≤ 200 KB; ảnh ≤ 500 KB/ảnh. HTML thô và `<script>` bị **cấm** (sanitize khi render, và CI cảnh báo).

### 3.4 Exercise — `exercise.yaml` + `statement.md`

```yaml
uid: 01JC…
slug: select-where-basic
title: "Lọc khách hàng theo thành phố"
type: code                 # *  code | quiz | self_check | project
difficulty: easy           # *  easy | medium | hard
lesson: select-where       # lesson liên quan (tuỳ chọn)
estimatedMinutes: 20
requiresReview: false      # project mặc định true
prerequisites:             # exercise phải hoàn thành trước (DAG, không vòng)
  - sql-30-days/foundations/intro-check
tags: [select, where]
authors: [github:tiennhm]
license: CC-BY-SA-4.0
spec:                      # *  tuỳ theo type, xem 3.5
  ...
```

### 3.5 `spec` theo loại

**`code`** (chấm bằng `learning-hub-checker`):
```yaml
spec:
  starterTemplate: hcmute-dsc/sql-select-where-template   # repo template (công khai)
  checker:
    workflow: check-exercise.yml            # workflow trong learning-hub-checker
    tests: exercises/sql-30-days/select-where-basic   # thư mục test ẩn trong checker repo
  runtime: { language: sql, timeoutSeconds: 120 }
  submit: { kind: github_pr, allowedPaths: ["solution.sql"] }   # file thành viên được sửa
```

**`quiz`**:
```yaml
spec:
  shuffle: true
  passScore: 0.7
  questions:
    - id: q1
      type: single            # single | multiple | short_text
      prompt: "`NULL = NULL` trả về gì?"
      options: [TRUE, FALSE, NULL]
      answer: 2               # chỉ số; khi chấm ở server, KHÔNG gửi `answer` xuống client
      explanation: "So sánh với NULL cho kết quả NULL (logic ba trị)."
```

**`self_check`**: `spec: { checklist: [..], reveal: statement-solution.md }` — member tự đánh dấu hoàn thành.

**`project`**: `spec: { deliverable: github_repo_or_pr, rubric: [ {criterion: "...", weight: 2} ] }` — mentor đánh giá theo rubric.

### 3.6 Test cases (cho `code`)
Test ẩn nằm trong `learning-hub-checker` (repo riêng, chỉ Ban kỹ thuật có quyền ghi) để thành viên không đọc/sửa được. Định dạng test do exercise quyết định (vd so sánh kết quả truy vấn với tập kỳ vọng); ví dụ test hiển thị công khai (2–3 ca) đặt trong `statement.md` hoặc `examples:` trong `exercise.yaml`. Tác giả content gửi test qua PR vào checker repo (review bởi CODEOWNERS), liên kết bằng `spec.checker.tests`.

### 3.7 Hints (P2)
`hints/NN.md`, mở dần: mỗi hint có `cost` mềm (số lần thử tối thiểu trước khi mở). MVP: không có.

### 3.8 Solutions (P2)
Repo content là **công khai** (khuyến khích đóng góp) ⇒ lời giải và test ẩn **không** được đặt ở đây. Chúng nằm ở `learning-hub-checker` (**riêng tư**): `exercises/<track>/<exercise>/{tests/, solution.md}`. Hub chỉ trả lời giải cho member **sau khi** exercise `approved/passed` (và có thể tắt theo bài). Với `self_check`, đáp án gập cần hiển thị ngay nên nằm trong `statement.md` công khai (chấp nhận được vì không chấm điểm).

### 3.9 Prerequisites
Hai tầng: `lesson.prerequisites` (khuyến nghị, hiển thị) và `exercise.prerequisites` (bắt buộc, khóa bài). Tham chiếu theo đường dẫn logic `track/module/slug`. Phải tạo **DAG** (CI kiểm tra vòng).

## 4. Validation

`tools/validate` (Node + Ajv + Zod, chạy cả local và CI, cũng được API gọi khi sync):

| Kiểm tra | Lỗi nếu |
| --- | --- |
| Schema | thiếu trường bắt buộc, sai enum/kiểu |
| `uid` | thiếu, trùng, hoặc đổi so với base commit |
| `slug` | trùng trong cùng cha |
| Tham chiếu | `modules/lessons/exercises/prerequisites/lesson` trỏ vào thứ không tồn tại |
| Prerequisite graph | có vòng |
| File | lesson/exercise không được liệt kê trong module (mồ côi) |
| Link | link nội bộ gãy; link ngoài (cảnh báo, không fail) |
| Nội dung | HTML nguy hiểm, ảnh quá nặng, secret (gitleaks) |
| Giấy phép | thiếu `license`/`authors` khi có `source` |
| Quiz | `answer` hợp lệ, ít nhất 2 option |
| Code exercise | `spec.checker` tồn tại trong checker repo (job CI chéo repo, tuỳ chọn) |

Kết quả validate lưu trong `content_versions.summary`.

## 5. Phiên bản & truy vết

- Mỗi sync tại commit `C` tạo `content_versions(commit_sha = C)`. `published` chỉ do admin bấm.
- Đối tượng nội dung lưu `content_version_id` (version gần nhất thay đổi nó) và `content_hash` (sha256 nội dung chuẩn hóa) để bỏ qua bản ghi không đổi.
- `submissions.content_version_id` lưu version lúc nộp ⇒ biết bài được chấm theo đề nào.
- Release có nhãn người đọc được: tag Git `v2027.03.1` (CalVer) trên content repo; `content_versions.summary.tag` lưu tag nếu có.
- Hiển thị cuối mỗi lesson: "Cập nhật lần cuối: `<ngày>` · `<sha ngắn>` · Sửa trên GitHub".
- Đổi **phá vỡ** (xóa exercise đã có người nộp, đổi tiêu chí chấm): CI gắn nhãn `breaking`; hub không xóa mà `archived`; thông báo mentor.

## 6. Quy trình Pull Request

1. Fork/branch từ `main`, viết nội dung, chạy `pnpm content:validate` + `pnpm content:preview`.
2. Mở PR dùng template (mục đích, loại thay đổi, kiểm tra tự làm, có breaking không).
3. CI bắt buộc: validate, link check, gitleaks, build preview.
4. Review: ≥ 1 maintainer của track (theo `CODEOWNERS`); thay đổi `exercise` loại `code` cần thêm 1 người ở Ban kỹ thuật (liên quan test ẩn).
5. Merge vào `main` (squash). Branch protection: không push trực tiếp.
6. Webhook `push` (hoặc admin bấm Sync) → hub tạo `content_version` draft → admin xem diff và publish.

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

Bộ phân tích (`ContentParser`) nằm trong module `content`, nhận `ContentSource` và trả về mô hình nội dung đã validate (`ContentTree`) — **không biết** GitHub. Triển khai: `GithubContentSource` (MVP), `LocalDirContentSource` (dev/test, đọc thư mục local), có thể thêm `GitlabContentSource`, `TarballContentSource`. Thay đổi cấu trúc thư mục chỉ ảnh hưởng `ContentParser` + schema, và có `contentSpecVersion` trong `tracks/…` hoặc file gốc `content.config.yaml` để hỗ trợ nhiều phiên bản schema.

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
