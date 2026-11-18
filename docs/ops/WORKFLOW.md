# Quy trình làm việc với GitHub Project

Mọi việc của Learning Hub (code, nội dung, tài liệu, vận hành, việc của CLB) được theo dõi trên một bảng chung:

**[DSC Learning Hub · GitHub Project #1](https://github.com/orgs/hcmute-dsc/projects/1)** (project của org `hcmute-dsc`, chỉ thành viên org xem được)

Đã liên kết 3 repo: `learning-hub`, `learning-hub-content`, `learning-hub-docs`. Tạo ngày 18/11/2026.

> Issue nằm trong các repo **public**: ai cũng đọc được. Không ghi secret, mật khẩu, URL webhook, IP VPS hay email thành viên vào issue, comment hoặc PR.

## 1. Các field

| Field | Giá trị | Dùng để |
| --- | --- | --- |
| **Status** | Backlog → Ready → In progress → In review → Done | biết việc đang ở đâu |
| **Priority** | P0 - Gấp · P1 - Cao · P2 - Bình thường | chọn việc làm trước |
| **Area** | Backend · Frontend · Content · Ops · Docs · Club | ai phù hợp nhận việc |
| **Epic** | E6 Chấm tự động · E8 Mentorship · E13 SQL checker · E14 Bảng xếp hạng · E15 Web Push · Phase 0 · Vận hành | nhóm việc theo [MVP-BACKLOG](../product/MVP-BACKLOG.md) |
| **Size** | S (≤ 1 buổi) · M (vài buổi) · L (cả sprint, nên tách nhỏ) | ước lượng |
| **Sprint** | 2 tuần, bắt đầu thứ Hai (Sprint 1: 16/11/2026) | lên kế hoạch theo đợt |

Ý nghĩa Status:

| Status | Khi nào |
| --- | --- |
| Backlog | đã ghi nhận, chưa đủ thông tin hoặc chưa tới lượt |
| Ready | mô tả rõ, có tiêu chí hoàn thành, ai rảnh cũng nhận được |
| In progress | đã có người nhận (Assignees) và đang làm |
| In review | đã mở PR, chờ review |
| Done | PR đã merge hoặc việc đã xong (đóng issue) |

Thêm Epic mới: Project → ⚙ Settings → Epic → thêm option. Khi thêm epic vào MVP-BACKLOG thì thêm luôn ở đây.

### 1.1 Thông tin bắt buộc trên mỗi issue

Mọi issue phải điền **đủ** các ô ở khung bên phải. Issue thiếu thông tin coi như chưa tạo xong.

| Ô | Giá trị | Chọn thế nào |
| --- | --- | --- |
| **Type** (của org) | Bug · Feature · Task | Bug: chạy sai. Feature: tính năng mới cho người dùng. Task: mọi việc khác (vận hành, tài liệu, test, refactor, việc CLB) |
| **Labels** | `bug` · `enhancement` · `documentation` · `ops` · `club` (repo docs) · `content` (repo nội dung) | khớp với Type và Area |
| **Assignees** | người đang làm | việc ở Backlog để trống: ai rảnh thì nhận |
| **Priority** (của org) | Urgent · High · Medium | tự suy ra từ Priority của project: P0 → Urgent, P1 → High, P2 → Medium |
| **Effort** (của org) | Low · Medium · High | tự suy ra từ Size: S → Low, M → Medium, L → High |
| **Start date / Target date** (của org) | ngày | mặc định là ngày đầu và cuối Sprint. Việc đã xong thì ghi ngày bắt đầu và ngày đóng thật |
| **Milestone** | `Phase 0 — Dùng thử` · `MVP-Full` (chỉ repo code) | MVP-Full = các epic E6, E8, E13, E14, E15. Còn lại là Phase 0 |
| **Projects** | Status, Priority, Area, Epic, Size, Sprint | như bảng ở §1 |
| **Relationships** | blocked by · sub-issue | ghi khi có. Ví dụ: docs#2 bị chặn bởi docs#1 |
| **Development** | PR | tự có khi PR ghi `Closes #n`. PR ở repo khác thì comment link vào issue |

Một lệnh điền tất cả, kèm cách dùng chi tiết: skill `start-task` của repo `learning-hub`.

```bash
bash .claude/skills/start-task/project-item.sh <link issue> \
  Type=Task "Status=In progress" "Priority=P1 - Cao" Area=Backend Size=S "Epic=Vận hành" \
  Sprint=@current "Milestone=Phase 0 — Dùng thử" Assignee=@me Label=enhancement
```

PR cũng điền: assignee, label, milestone, và các field trên project giống issue của nó (Status = In review).

Ngày 18/11/2026 đã điền bổ sung cho toàn bộ issue và PR đang có trên board.

## 2. Vòng đời một việc

**Không có issue thì không làm.** Mọi yêu cầu, kể cả sửa nhỏ hay lỗi phát hiện rồi sửa ngay, đều có issue trên project. Chi tiết về nhánh, commit, PR, review ở [§7](#7-tiêu-chuẩn-nhánh-commit-pr-review-bug).

1. **Tạo issue** trong repo đúng chỗ, bằng mẫu có sẵn (issue trống bị tắt): code → `learning-hub`, bài học → `learning-hub-content`, tài liệu và việc của CLB → `learning-hub-docs`. Tìm issue cũ trước khi tạo mới.
   - Tiêu đề có tiền tố: `[E13] …`, `[Content] …`, `[Ops] …`, `[Org] …`, `[Phase 0] …`, `[Bug] …`.
   - Thân issue có: bối cảnh (link tài liệu), checklist việc cần làm, **"Hoàn thành khi: …"**.
2. **Điền đủ thông tin** theo [§1.1](#11-thông-tin-bắt-buộc-trên-mỗi-issue) (issue ở repo `learning-hub` tự vào project với Status = Backlog; hai repo kia cần token ở §4, chưa có thì script thêm vào). Đủ thông tin thì chuyển Ready.
3. **Nhận việc:** tự gán mình vào Assignees, chuyển Status = In progress, đặt Sprint. Tạo nhánh theo §7.
4. **Mở PR** theo mẫu, dòng đầu `Closes #<số>` (khác repo: `Closes hcmute-dsc/<repo>#<số>`). Project tự chuyển sang In review.
5. **Review rồi squash merge:** issue tự đóng và tự sang Done.
6. **Kiểm tra trên production** sau khi Deploy xanh (code) hoặc sau khi publish (nội dung). Có lỗi thì mở lại issue.

Việc lớn (Size L) thì tách thành nhiều issue nhỏ, hoặc dùng checklist trong issue và tách dần.

## 3. Họp sprint (2 tuần một lần, khoảng 30 phút)

1. Xem view **Board**: việc nào Done, việc nào kẹt ở In progress / In review quá lâu.
2. Chọn việc cho sprint mới: ưu tiên P0 → P1, chuyển từ Backlog sang Ready nếu đã rõ, đặt Sprint.
3. Mỗi người nhận 1–2 việc phù hợp thời gian học. CLB là việc tình nguyện: nhận ít mà xong còn hơn nhận nhiều mà bỏ dở.
4. Thêm sprint mới khi hết: Project → ⚙ Settings → Sprint → Add iteration.

## 4. Cài đặt làm bằng tay trên web (API của GitHub chưa hỗ trợ)

Chỉ làm một lần, hoặc khi tạo lại project. Ai có quyền admin project làm:

**View** (tab ở đầu project → New view):

| Tên view | Kiểu | Cấu hình |
| --- | --- | --- |
| Board | Board | Column by: Status; Group by: Priority (tùy chọn) |
| Backlog | Table | Group by: Epic; Sort: Priority; hiện cột Area, Size, Sprint, Assignees |
| Sprint hiện tại | Table hoặc Board | Filter: `sprint:@current` |
| Roadmap | Roadmap | Date field: Sprint; Group by: Epic |

**Workflows** (nút **Workflows** góc phải trên của project). Status đã được đổi tên thành Backlog/Ready/…, nên các workflow mặc định trỏ vào option cũ (Todo/In Progress) và hiện **dấu ! đỏ**. Mở từng cái, bấm **Edit**, chọn lại giá trị, rồi **Save and turn on workflow**:

| Workflow | Cấu hình |
| --- | --- |
| Item added to project | Status = Backlog |
| Item reopened | Status = Ready |
| Pull request linked to issue | Status = In review |
| Code changes requested | Status = In progress (người làm sửa tiếp) |
| Code review approved | Status = In review (chờ merge) |
| Item closed | Status = Done |
| Pull request merged | Status = Done |
| Auto-close issue | khi Status = Done thì đóng issue |
| Auto-archive items | `is:issue is:closed updated:<@today-2w`: việc xong quá 2 tuần được lưu trữ (xem lại ở ⋯ → Archived items) |
| Auto-add sub-issues | bật, để sub-issue tự vào project |
| Auto-add to project | repo `learning-hub`, filter **`is:issue,pr is:open`** (xóa `label:bug` mặc định) |

Đã cấu hình và kiểm tra trên web ngày 18/11/2026: cả 11 workflow bật, giá trị như bảng trên.

**Giới hạn của gói Free:** mỗi workflow auto-add chỉ chọn được **1 repo**, và org gói Free chỉ có **1** workflow auto-add (nút *Duplicate workflow* báo "Maximum number of auto add workflows reached"). Vì vậy `learning-hub-content` và `learning-hub-docs` dùng GitHub Action riêng: `.github/workflows/add-to-project.yml` (dùng `actions/add-to-project`, đã ghim theo commit). Nó thêm mọi issue/PR mới vào project. Khi chưa cài GitHub App, job chỉ ghi chú và bỏ qua, không báo đỏ.

Action lấy token từ một **GitHub App của org**, không dùng token cá nhân. Trước đây job dùng token classic của một thành viên (`ADD_TO_PROJECT_PAT`), nên dùng chung giới hạn API với tài khoản đó: tài khoản gọi API nhiều thì job đỏ với lỗi `API rate limit already exceeded for user ID …` ([learning-hub-docs#8](https://github.com/hcmute-dsc/learning-hub-docs/issues/8)). GitHub App có giới hạn riêng, thuộc về org, không hết hạn và không phải tạo lại khi đổi người.

**Cài GitHub App (một lần, owner của org làm):**

1. Tạo App tại https://github.com/organizations/hcmute-dsc/settings/apps/new:
   - **GitHub App name:** `hcmute-dsc-learning-hub` (tên phải chưa ai dùng trên GitHub; trùng thì thêm hậu tố).
   - **Homepage URL:** `https://github.com/hcmute-dsc/learning-hub`.
   - **Webhook:** bỏ tick **Active**.
   - **Permissions:**
     - Repository → **Issues**: Read-only, **Pull requests**: Read-only (Metadata: Read-only tự có).
     - Organization → **Projects**: Read and write.
     - Các quyền khác để No access.
   - **Where can this GitHub App be installed?** Only on this account.
   - Bấm **Create GitHub App**.
2. Trên trang App vừa tạo:
   - Ghi lại **Client ID** (dạng `Iv23…`; không phải bí mật).
   - Kéo xuống **Private keys** → **Generate a private key**. Trình duyệt tải về file `.pem`. File này là bí mật: không gửi qua chat, issue hay commit.
3. Menu trái **Install App** → **Install** cạnh `hcmute-dsc` → **Only select repositories**: `learning-hub-content`, `learning-hub-docs` → **Install**.
4. Lưu vào **từng repo** (chạy trên máy mình; thay `<client-id>` và đường dẫn file `.pem`):

   ```bash
   gh variable set PROJECT_APP_CLIENT_ID --repo hcmute-dsc/learning-hub-content --body "<client-id>"
   gh variable set PROJECT_APP_CLIENT_ID --repo hcmute-dsc/learning-hub-docs --body "<client-id>"
   gh secret set PROJECT_APP_PRIVATE_KEY --repo hcmute-dsc/learning-hub-content < ~/Downloads/<file>.pem
   gh secret set PROJECT_APP_PRIVATE_KEY --repo hcmute-dsc/learning-hub-docs < ~/Downloads/<file>.pem
   ```

   Xong thì xóa file `.pem` khỏi máy (hoặc cất vào trình quản lý mật khẩu của CLB). Mất file thì chỉ cần tạo key mới ở bước 2 rồi chạy lại 2 lệnh `gh secret set`.

   (Cũng có thể lưu thành variable/secret của org bằng `--org hcmute-dsc --repos learning-hub-content,learning-hub-docs`. Cách này cần `gh` có quyền `admin:org`, nếu thiếu sẽ báo lỗi `HTTP 403 … needs the "admin:org" scope`. Lưu theo repo chỉ cần quyền `repo` sẵn có, nên dùng cách trên.)

5. Kiểm tra: tạo thử 1 issue trong `learning-hub-docs`. Tab Actions → *Add to project* phải xanh (bước *Get a short-lived token from the GitHub App* chạy, không bị bỏ qua), và issue xuất hiện trong project với Status = Backlog. Xong thì đóng issue thử.
6. Gỡ token cũ, nếu còn:

   ```bash
   gh secret delete ADD_TO_PROJECT_PAT --repo hcmute-dsc/learning-hub-content
   gh secret delete ADD_TO_PROJECT_PAT --repo hcmute-dsc/learning-hub-docs
   ```

   Rồi xóa token classic `learning-hub add-to-project` tại https://github.com/settings/tokens.

**Bàn giao:** App thuộc org nên không phải làm gì khi đổi tech lead; người mới chỉ cần là owner của org để sửa App. Nghi lộ private key: trang App → **Private keys** → tạo key mới, chạy lại 2 lệnh `gh secret set` ở bước 4, rồi xóa key cũ. Nếu cần thêm việc thủ công: khung **Projects** bên phải issue → DSC Learning Hub.

**Quyền** (⋯ → Settings → Manage access): thêm team `mentors` / Ban chủ nhiệm với quyền *Write*. Thành viên chỉ cần *Read*.

## 5. Tạo lại bằng `gh` (khi cần)

Cần quyền `project` cho `gh`: `gh auth refresh -s project` (cách cài và các quyền khác: [GH-CLI.md](GH-CLI.md)).

```bash
gh project create --owner hcmute-dsc --title "DSC Learning Hub"
gh project field-create <số> --owner hcmute-dsc --name Priority --data-type SINGLE_SELECT \
  --single-select-options "P0 - Gấp,P1 - Cao,P2 - Bình thường"
# tương tự cho Area, Epic, Size; Status sửa option bằng GraphQL updateProjectV2Field;
# Sprint tạo bằng GraphQL createProjectV2Field (dataType: ITERATION, duration: 14)
gh project link <số> --owner hcmute-dsc --repo hcmute-dsc/learning-hub
gh project link <số> --owner hcmute-dsc --repo hcmute-dsc/learning-hub-content
gh project link <số> --owner hcmute-dsc --repo hcmute-dsc/learning-hub-docs
```

## 6. Việc đã đưa lên lúc tạo (18/11/2026)

| Issue | Ưu tiên | Sprint 1 |
| --- | --- | --- |
| [docs#1](https://github.com/hcmute-dsc/learning-hub-docs/issues/1) Phase 0: chạy đợt dùng thử (P0-3) | P0 | ✓ |
| [docs#2](https://github.com/hcmute-dsc/learning-hub-docs/issues/2) Phase 0: quyết định đi tiếp (P0-4) | P0 | sau P0-3 |
| [docs#3](https://github.com/hcmute-dsc/learning-hub-docs/issues/3) Cài đặt org trước khi mời thành viên | P0 | ✓ |
| [content#1](https://github.com/hcmute-dsc/learning-hub-content/issues/1) Áp dụng kết quả rà soát track SQL | P1 | ✓ |
| [hub#1](https://github.com/hcmute-dsc/learning-hub/issues/1) Chép bản sao lưu ra ngoài VPS | P1 | ✓ |
| [hub#2](https://github.com/hcmute-dsc/learning-hub/issues/2) Kiểm tra thông báo Discord qua app thật | P1 | ✓ |
| [hub#3](https://github.com/hcmute-dsc/learning-hub/issues/3) E13 sql-checker | P1 | |
| [hub#4](https://github.com/hcmute-dsc/learning-hub/issues/4) E6 chấm tự động qua checker repo | P2 | |
| [hub#5](https://github.com/hcmute-dsc/learning-hub/issues/5) E14 bảng xếp hạng | P2 | |
| [hub#6](https://github.com/hcmute-dsc/learning-hub/issues/6) E15 Web Push | P2 | |
| [hub#7](https://github.com/hcmute-dsc/learning-hub/issues/7) E8 mentorship | P2 | |

Từ nay, project là nơi theo dõi tiến độ. MVP-BACKLOG giữ phần thiết kế (story, acceptance).

## 7. Tiêu chuẩn: nhánh, commit, PR, review, bug

Áp dụng từ 18/11/2026 cho cả 3 repo. Bản đầy đủ cho repo code: [CONTRIBUTING của learning-hub](https://github.com/hcmute-dsc/learning-hub/blob/main/CONTRIBUTING.md); cho repo nội dung: [CONTRIBUTING của learning-hub-content](https://github.com/hcmute-dsc/learning-hub-content/blob/main/CONTRIBUTING.md).

### 7.1 Nhánh, commit, PR

| Mục | Quy ước | Ví dụ |
| --- | --- | --- |
| Nhánh | `<loại>/<số issue>-<mô-tả-ngắn>`, tách từ `main` mới nhất. Loại: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`. Issue ở repo khác thì thêm tên repo | `fix/8-track-source-commit`, `chore/docs-4-work-standards` |
| Commit (code, tài liệu) | `Phần: tóm tắt dạng mệnh lệnh` (≤ 72 ký tự). Thân ghi **vì sao**, dòng cuối `Refs #<số>` | `Catalog: show the published commit on the track page` |
| Commit (nội dung) | `<track>: <tóm tắt>` | `sql-30-days: sửa đáp án bài 3 Ngày 10` |
| PR | Theo mẫu; dòng đầu `Closes …`; một chủ đề; nên dưới ~400 dòng; đang làm thì mở **draft** | |
| Merge | **Squash merge** sau khi CI xanh và review xong; xóa nhánh | |

**Không đẩy thẳng lên `main`**, kể cả sửa gấp trên production: vẫn mở PR. Nếu không có người review kịp, tác giả merge khi CI xanh và nhờ review sau ngay trong PR.

### 7.2 Review

Người review xem theo thứ tự:
1. Đúng yêu cầu của issue chưa.
2. Bảo mật: phân quyền, kiểm tra đầu vào, không lộ dữ liệu cá nhân hay secret.
3. Thiết kế: theo quy tắc trong CONTRIBUTING.
4. Có test.
5. Dễ đọc cho người sau.
6. Tài liệu đã cập nhật.

Cách góp ý:
- Góp ý bắt buộc sửa ghi `must:`, góp ý tùy chọn ghi `nit:`.
- Approve khi không còn `must:`.
- Tác giả trả lời hoặc resolve từng thread trước khi merge.

Repo nội dung còn xem thêm: kiến thức đúng, câu lệnh chạy được, văn phong dễ hiểu, có ghi nguồn.

**Review tự động.** Mọi PR do agent (Claude Code, Codex…) mở đều được agent tự review theo checklist trên **trước khi** giao cho maintainer, theo skill `review-pr` (learning-hub#22):
- review sau khi CI xanh, đọc cả code xung quanh diff để kiểm tra các giả định;
- sửa hết `must:` rồi chạy lại CI;
- đăng kết quả lên PR thành comment "Review tự động" (GitHub không cho tự approve PR của mình), và gửi link review khi giao.

Maintainer đọc review đó trước, rồi quyết định merge. Review tự động không thay thế người duyệt.

Hiện chỉ có 1 maintainer, nên PR do Claude Code soạn sẽ được người giữ repo review rồi merge. Khi có thêm maintainer thì bật "1 approval" (§7.5).

### 7.3 Vòng đời bug

1. **Ghi nhận:** issue nhãn `bug` theo mẫu: các bước tái hiện, mong đợi, thực tế, mức ảnh hưởng → Priority (P0 site sập, mất hoặc lộ dữ liệu; P1 chức năng chính hỏng; P2 sai hiển thị).
2. **Xử lý:**
   - P0 làm ngay, ghi thêm vào [nhật ký sự cố DEPLOYMENT §7](DEPLOYMENT.md#7-nhật-ký-sự-cố).
   - P1 vào sprint hiện tại.
   - P2 xếp theo sprint.
3. **Sửa:** PR có mục **Nguyên nhân gốc** (vì sao xảy ra, không chỉ triệu chứng). Kèm test thất bại khi chưa sửa; không viết được test thì ghi lý do.
4. **Đóng:** khi đóng issue, ghi lại:
   - nguyên nhân gốc;
   - cách sửa (PR hoặc commit);
   - cách đã kiểm tra (test, và production sau deploy).

   Ví dụ mẫu: [learning-hub#8](https://github.com/hcmute-dsc/learning-hub/issues/8).
5. Lỗi không tái hiện được hoặc không sửa: đóng với lý do `not planned` và giải thích. Không đóng im lặng.

### 7.4 Definition of Done

Một việc chỉ chuyển sang Done khi đủ cả 4 điều:
- PR đã merge, CI xanh.
- Đã lên production: Deploy xanh với code; đã publish với nội dung.
- Đã kiểm tra trên production nếu người dùng thấy được thay đổi.
- Tài liệu liên quan đã cập nhật: thiết kế, `DATABASE.md`, runbook, hướng dẫn cài đặt (ghi ở cả `deploy/README.md` và `docs/ops/`).

### 7.5 Cài đặt repo để ép quy trình (admin làm, mỗi repo một lần)

Chưa bật tại thời điểm viết (18/11/2026). Dùng `gh` với tài khoản admin repo (quyền `repo` là đủ, [GH-CLI.md](GH-CLI.md)).

Chỉ cho squash merge và tự xóa nhánh sau merge:

```bash
for r in learning-hub learning-hub-content learning-hub-docs; do
  gh api -X PATCH repos/hcmute-dsc/$r \
    -F allow_squash_merge=true -F allow_merge_commit=false -F allow_rebase_merge=false \
    -F delete_branch_on_merge=true -f squash_merge_commit_title=PR_TITLE -f squash_merge_commit_message=COMMIT_MESSAGES >/dev/null
done
```

Bảo vệ nhánh `main` (bắt buộc PR, CI phải xanh, cấm force push và xóa nhánh). Check bắt buộc: `build-test` cho `learning-hub`, `validate` cho `learning-hub-content`; `learning-hub-docs` chưa có CI nên để trống:

```bash
protect() { # protect <repo> [check]
  checks='null'; [ -n "$2" ] && checks="{\"strict\":true,\"contexts\":[\"$2\"]}"
  gh api -X PUT repos/hcmute-dsc/$1/branches/main/protection --input - <<JSON
{"required_status_checks":$checks,"enforce_admins":false,
 "required_pull_request_reviews":{"required_approving_review_count":0},
 "restrictions":null,"allow_force_pushes":false,"allow_deletions":false,
 "required_conversation_resolution":true}
JSON
}
protect learning-hub build-test
protect learning-hub-content validate
protect learning-hub-docs
```

- `required_approving_review_count` để `0` khi chỉ có 1 maintainer, vì GitHub không cho tự approve PR của mình. Khi có maintainer thứ hai thì đổi thành `1`.
- `enforce_admins: false` để admin còn xử lý được sự cố. Dùng quyền này chỉ khi khẩn cấp và ghi vào nhật ký sự cố.

Kiểm tra: `gh api repos/hcmute-dsc/learning-hub/branches/main/protection -q '.required_status_checks.contexts'`.

### 7.6 Quy tắc cho AI agent (Claude Code, Codex, Cursor…)

Agent làm theo đúng quy trình như người. Quy tắc nằm trong repo để ai mở repo bằng agent cũng nhận được:

| Repo | File quy tắc | Skills (`.claude/skills/`) |
| --- | --- | --- |
| `learning-hub` | `AGENTS.md` (`CLAUDE.md` nạp file này) | `start-task` (issue → project → nhánh), `fix-bug` (vòng đời bug), `ship-pr` (commit → PR → CI → review → kiểm tra production), `review-pr` (checklist review, review tự động), `new-track` (phía Ban chủ nhiệm: tạo repo track, cấp quyền, đưa vào registry) |
| `learning-hub-content` (registry) | `AGENTS.md` | `content-change` (sửa/thêm bài trong repo track), `publish-content` (ghim commit, đồng bộ và publish lên site) |
| `learning-hub-track-<slug>` | `AGENTS.md` (từ template) | `new-track` trong repo template (phía mentor: khởi tạo, viết, PR, publish) |
| `learning-hub-docs` | `AGENTS.md` | dùng skills của `learning-hub` |

Script `learning-hub/.claude/skills/start-task/project-item.sh` đưa issue/PR vào project và điền đủ thông tin ở §1.1 theo tên. Ví dụ:

```bash
bash .claude/skills/start-task/project-item.sh <link issue> Type=Task "Status=In progress" "Priority=P1 - Cao" \
  Area=Backend Size=S "Epic=Vận hành" Sprint=@current "Milestone=Phase 0 — Dùng thử" Assignee=@me Label=ops
```

Agent không tự merge PR, không đổi cài đặt repo hay production, và không publish nội dung khi chưa có người đồng ý trong chat.

Khi đổi quy trình ở §7, sửa luôn `AGENTS.md` và skills cho khớp, trong cùng đợt PR.

### 7.7 Nội dung: mỗi track một repo

Từ ADR-10, mỗi track nằm trong repo `hcmute-dsc/learning-hub-track-<slug>`; `learning-hub-content` chỉ còn là registry (`tracks.yaml`). Việc nào mở issue ở đâu, ai review:

| Việc | Issue ở repo | Ai review và merge |
| --- | --- | --- |
| Sửa lỗi, thêm bài trong một track | Repo của track đó | Maintainer của track (`CODEOWNERS`, `maintainers` trong `track.yaml`) |
| Đề xuất track mới | `learning-hub-content` (mẫu "Đề xuất") | Ban chủ nhiệm quyết định, tạo repo, duyệt PR đầu tiên |
| Publish một track (đổi `commit` trong `tracks.yaml`) | `learning-hub-content` | Ban chủ nhiệm duyệt PR, admin sync và publish |
| Lỗi của validator hay trang web | `learning-hub` | Tech lead |

Mọi issue ở các repo này đều nằm trên Project #1 và điền đủ thông tin ở §1.1. Hướng dẫn đầy đủ cho mentor: [NEW-TRACK.md](NEW-TRACK.md).
