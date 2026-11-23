# Tạo track mới (cho mentor và Ban chủ nhiệm)

Hướng dẫn từ lúc có ý tưởng tới lúc track hiện trên https://learn.hcmute-dsc.io.vn. Mỗi track nằm trong **một repo riêng** `hcmute-dsc/learning-hub-track-<slug>`, tạo từ [learning-hub-track-template](https://github.com/hcmute-dsc/learning-hub-track-template) ([DECISIONS ADR-10](../product/DECISIONS.md)). Mentor chỉ cần clone repo của track mình, không cần clone cả hệ thống.

Cập nhật lần cuối: 19/11/2026 (thêm trang Cài đặt / Tài liệu thêm, learning-hub#44). Đã chạy thử các bước của mentor (mục 3) trên bản sao của template: `init-track` và validator đều chạy đúng.

## 0. Ai làm gì

| Bước | Ai làm | Kết quả |
| --- | --- | --- |
| 1. Đề xuất | Mentor | Issue "Đề xuất" trong `learning-hub-content` |
| 2. Tạo repo, cấp quyền | Ban chủ nhiệm (admin org) | Repo `learning-hub-track-<slug>`, mentor có quyền Maintain |
| 3. Viết nội dung | Mentor | PR vào repo track, CI `validate` xanh |
| 4. Review và merge | Maintainer của track; track mới thì Ban chủ nhiệm duyệt PR đầu tiên | Nội dung nằm trên `main` của repo track |
| 5. Publish | Mentor mở PR, Ban chủ nhiệm duyệt và publish | Track hiện trên site |

Dùng `gh` (GitHub CLI) cho các lệnh bên dưới: cài và đăng nhập theo [GH-CLI.md](GH-CLI.md). Mentor chỉ cần `git`, Node 22 và một tài khoản GitHub.

## 1. Đề xuất track (mentor)

Mở issue mẫu **"Đề xuất bài mới hoặc lộ trình mới"** ở [learning-hub-content](https://github.com/hcmute-dsc/learning-hub-content/issues/new/choose). Ghi:

- tên track và `slug` dự kiến (chữ thường, số, dấu `-`; ví dụ `csharp-basics`). Slug sẽ là tên repo và đường dẫn `/tracks/<slug>` trên site, **không đổi được** sau khi publish;
- người học là ai, học xong làm được gì, khoảng bao nhiêu giờ;
- danh sách chương dự kiến;
- tài khoản GitHub của mentor (và người viết cùng, nếu có).

Ban chủ nhiệm điền đủ các field của issue trên Project #1 ([WORKFLOW §1.1](WORKFLOW.md#11-thông-tin-bắt-buộc-trên-mỗi-issue)) và trả lời trong issue.

## 2. Tạo repo và cấp quyền (Ban chủ nhiệm)

Thay `<slug>`, `<Tên track>` và `<mentor>` (tài khoản GitHub) rồi chạy:

```bash
R=hcmute-dsc/learning-hub-track-<slug>
gh repo create $R --public --template hcmute-dsc/learning-hub-track-template \
  --description "Track <slug> của DSC Learning Hub: <Tên track> (ADR-10)"
gh repo edit $R --enable-wiki=false
gh label create content -R $R --color 1D76DB --description "Nội dung bài học, bài tập" --force
gh label create ops -R $R --color 0E8A16 --description "Vận hành: CI/CD, GitHub Actions, hạ tầng" --force
gh api repos/$R/milestones -f title="Phase 0 — Dùng thử"
gh api -X PUT repos/$R/collaborators/<mentor> -f permission=maintain
```

- Repo phải **public**: site tải nội dung công khai, và CI của registry tải track bằng link công khai.
- Quyền **Maintain** cho mentor: tự merge được PR trong repo track, nhưng không đổi được cài đặt quan trọng của repo. Mentor sẽ nhận email mời; cần bấm **Accept**.
- **GitHub App** `hcmute-dsc-learning-hub` đã cài cho mọi repo của org, và `PROJECT_APP_CLIENT_ID` / `PROJECT_APP_PRIVATE_KEY` đặt ở cấp org cho mọi repo public (docs#14), nên issue và PR của repo mới tự lên Project #1, không cần làm gì thêm. Không tạo bản cấp repo cho hai giá trị này.
- Không bật branch protection khi chưa được Ban chủ nhiệm đồng ý ([WORKFLOW §7.5](WORKFLOW.md#75-cài-đặt-repo-để-ép-quy-trình-admin-làm-mỗi-repo-một-lần)).

Trả lời trong issue đề xuất: link repo, và nhắc mentor làm tiếp mục 3.

## 3. Viết nội dung (mentor)

### 3.1 Khởi tạo, làm một lần

```bash
git clone https://github.com/hcmute-dsc/learning-hub-track-<slug>
cd learning-hub-track-<slug>
git switch -c feat/content-<số issue đề xuất>-init-track
node tools/init-track.mjs <slug> "<Tên track>" <tài-khoản-github-của-bạn>
```

Lệnh này đổi `tracks/example-track` thành `tracks/<slug>`, ghi tên track, đặt bạn làm maintainer (`track.yaml` và `CODEOWNERS`), và **sinh `uid` mới** cho mọi bài. Chạy lần thứ hai sẽ báo lỗi "already initialized"; đó là bình thường.

### 3.2 Viết bài

Mỗi file mẫu có comment giải thích từng trường. Đặc tả đầy đủ: [CONTENT-SPECIFICATION](../product/CONTENT-SPECIFICATION.md).

```
tracks/<slug>/
  track.yaml                       # tóm tắt, cấp độ, thẻ, mục tiêu, danh sách chương
  docs/setup.md                    # (tuỳ chọn) trang "Cài đặt": cài công cụ, tự kiểm tra bài
  docs/resources.md                # (tuỳ chọn) trang "Tài liệu thêm"
  glossary.yaml                    # (tuỳ chọn) trang "Từ vựng": chữ tiếng Anh, dạng đầy đủ, nghĩa đen
  assets/                          # ảnh (≤ 500 KB/ảnh)
  modules/01-<tên-chương>/
    module.yaml                    # thứ tự bài học và bài tập của chương
    lessons/01-<tên-bài>.md        # bài học
    exercises/<tên-bài-tập>/
      exercise.yaml                # quiz, self_check hoặc project
      statement.md                 # đề bài
```

Quy tắc quan trọng:

- **`uid` không bao giờ đổi** sau khi publish: nó là khóa lưu tiến độ của người học. Bài mới cần `uid` mới: `node -e "console.log(crypto.randomUUID())"`.
- Mỗi bài học và bài tập phải có tên trong `module.yaml`, nếu không validator báo "orphan".
- Xóa bài mẫu không dùng thì xóa cả tên trong `module.yaml`.
- Mọi câu lệnh, đoạn code trong bài phải **chạy thử** trước khi commit. Đáp án quiz phải khớp câu hỏi.
- Nội dung lấy từ nơi khác: ghi nguồn và tác giả trong PR và trong `track.yaml` (`source`).
- Không đưa lời giải hay test ẩn vào repo (repo công khai).
- Track cần cài công cụ trước khi học (Docker, PostgreSQL, Python…): viết `docs/setup.md` thay vì một bài học "Cài đặt". Ghi từng bước cho Windows/macOS/Linux và một lệnh để tự kiểm tra, kèm kết quả mong đợi. Site hiện tab **Cài đặt** trên trang track và nhắc người mới enroll mở nó. Không cần thì xóa file mẫu; `docs/resources.md` cũng vậy ([CONTENT-SPECIFICATION §3.1b](../product/CONTENT-SPECIFICATION.md#31b-trang-cấp-track--docssetupmd-docsresourcesmd-learning-hub44)).

### 3.3 Kiểm tra trên máy

Làm một lần (cần Node 22):

```bash
git clone https://github.com/hcmute-dsc/learning-hub ../learning-hub
cd ../learning-hub && npx -y pnpm@9.15.4 install --filter @hub/content... && npx -y pnpm@9.15.4 --filter @hub/content build && cd -
```

Mỗi lần sửa xong:

```bash
node ../learning-hub/packages/content/dist/cli.js .
```

Kết quả đúng có dạng `✓ 1 tracks, … — all valid`.

### 3.4 Mở Pull Request

```bash
git add -A
git commit -m "<slug>: khởi tạo track" -m "Refs hcmute-dsc/learning-hub-content#<số issue đề xuất>"
git push -u origin HEAD
```

Mở PR trên GitHub theo mẫu có sẵn. CI `validate` phải xanh. Các lần sửa sau làm theo `CONTRIBUTING.md` của repo track: issue trong repo track, nhánh `<loại>/<số issue>-<mô-tả>`, PR có `Closes #<số>`.

## 4. Review và merge

- PR đầu tiên của một track mới: **Ban chủ nhiệm** review (cấu trúc, văn phong, kiến thức đúng), mentor sửa theo góp ý.
- Các PR sau: **maintainer của track** (người trong `CODEOWNERS`) review rồi **squash merge**.
- Agent (Claude Code, Codex…) mở PR thì tự review và đăng kết quả lên PR trước khi giao ([WORKFLOW §7.2](WORKFLOW.md#72-review)).

Merge vào `main` của repo track **chưa** làm thay đổi site.

## 5. Publish lên site

1. Lấy commit mới nhất trên `main` của repo track:

   ```bash
   git ls-remote https://github.com/hcmute-dsc/learning-hub-track-<slug> main
   ```

2. Mở PR vào [learning-hub-content](https://github.com/hcmute-dsc/learning-hub-content), thêm (lần đầu) hoặc sửa mục của track trong `tracks.yaml`:

   ```yaml
   tracks:
     - slug: <slug>
       repo: hcmute-dsc/learning-hub-track-<slug>
       commit: "<đủ 40 ký tự của commit>"
   ```

   Tiêu đề `<slug>: publish <7 ký tự đầu của commit>`. Thân PR liệt kê các PR của repo track từ lần publish trước.
3. CI `validate` tải **mọi** track ở commit đã ghim và kiểm tra cùng nhau (trùng `uid` hay `slug` giữa các track, prerequisite sang track khác).
4. Ban chủ nhiệm duyệt và squash merge.
5. Admin vào **Quản trị → Nội dung**, bấm **Đồng bộ từ GitHub**, mở bản nháp và kiểm tra số Thêm / Sửa / Xóa khớp với các PR (track mới: chỉ có Thêm), rồi bấm **Publish**. Site đăng thông báo lên Discord.
6. Mở `/tracks/<slug>`: dòng **Nguồn** ở cuối trang phải là `hcmute-dsc/learning-hub-track-<slug> @ <commit>`.

Issue đề xuất chỉ chuyển Done khi track đã lên site.

## 6. Lỗi thường gặp

| Thông báo | Nguyên nhân | Cách sửa |
| --- | --- | --- |
| `orphan` | File bài học/bài tập không có tên trong `module.yaml` | Thêm tên vào `module.yaml`, hoặc xóa file |
| `tracks/ must contain exactly one folder` | Repo có nhiều hơn một thư mục trong `tracks/` | Mỗi repo chỉ một track |
| `slug "…" must equal the folder name` | `slug` trong `track.yaml` khác tên thư mục | Sửa cho trùng nhau |
| `… may only contain tracks/<slug>/` (khi sync) | Repo track có thư mục track khác | Như trên |
| `tracks.yaml: … commit must be a full 40-character SHA` | Ghi tên nhánh hoặc commit rút gọn | Dán đủ 40 ký tự từ `git ls-remote` |
| `track "<slug>" is both in this repo and in …` | Track vẫn còn trong `tracks/` của learning-hub-content | Xóa thư mục cũ trong learning-hub-content |
| `uid` trùng | Copy bài từ track khác mà không đổi `uid` | Đổi `uid` của bài **chưa publish**; bài đã publish thì không đổi |
| `only docs/setup.md and docs/resources.md are published from docs/` | Gõ sai tên file trang track, hoặc thêm trang khác trong `docs/` | Đổi tên thành `setup.md`/`resources.md`; nội dung khác viết thành bài học |
| `front matter is not used here` | `docs/setup.md` hoặc `docs/resources.md` có khối `--- … ---` | Xóa khối đó, bắt đầu bằng nội dung |

## 7. Xong khi

- [ ] Repo `learning-hub-track-<slug>` public, mentor có quyền Maintain, CI `validate` xanh trên `main`.
- [ ] `tracks.yaml` trong learning-hub-content ghim đúng commit, CI xanh.
- [ ] Track đã publish, trang `/tracks/<slug>` mở được, dòng Nguồn đúng repo. Track có `docs/setup.md` thì tab **Cài đặt** mở được.
- [ ] Issue đề xuất Done trên Project #1.

Skill cho agent: `new-track` trong [learning-hub](https://github.com/hcmute-dsc/learning-hub/tree/main/.claude/skills/new-track) (phía Ban chủ nhiệm) và trong repo template (phía mentor).
