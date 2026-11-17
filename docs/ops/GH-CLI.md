# Cấu hình GitHub CLI (`gh`) cho người vận hành

Phần lớn việc vận hành Learning Hub làm bằng `gh` trên máy của tech lead: đặt secret cho deploy, chạy workflow, quản lý GitHub Project và issue. Tài liệu này ghi cách cài, các quyền cần cấp, và những lỗi đã gặp.

Cập nhật lần cuối: 18/11/2026 (`gh` 2.97, Windows 11; trên macOS/Linux lệnh giống hệt).

> `gh` đăng nhập bằng **tài khoản GitHub của bạn**, nên mọi thao tác ghi tên bạn. Token nằm trong keyring của máy (Windows Credential Manager / macOS Keychain), không nằm trong file. Không dùng máy chung để đăng nhập.

## 1. Cài đặt

| Hệ điều hành | Lệnh |
| --- | --- |
| Windows | `winget install --id GitHub.cli` |
| macOS | `brew install gh` |
| Ubuntu/Debian | xem https://github.com/cli/cli/blob/trunk/docs/install_linux.md |

Kiểm tra: `gh --version`.

## 2. Đăng nhập với đủ quyền

```bash
gh auth login --hostname github.com --git-protocol https --web --scopes "repo,workflow,read:org,project"
```

Lệnh mở trình duyệt. Dán mã một lần hiện trên terminal vào trang đó, rồi bấm **Authorize**.

Đã đăng nhập từ trước nhưng thiếu quyền thì **thêm** quyền, không cần đăng xuất:

```bash
gh auth refresh -h github.com -s project
```

Kiểm tra quyền hiện có:

```bash
gh auth status
```

Dòng `Token scopes` phải có ít nhất `repo`, `workflow`, `read:org`, `project`.

## 3. Quyền nào dùng cho việc gì

| Scope | Cần cho | Ví dụ lệnh |
| --- | --- | --- |
| `repo` | đọc/ghi repo, issue, PR; đặt **secret của repo / environment** | `gh secret set VPS_SSH_KEY --env production --repo hcmute-dsc/learning-hub` |
| `workflow` | chạy và sửa GitHub Actions | `gh workflow run deploy.yml --repo hcmute-dsc/learning-hub --ref main` |
| `read:org` | xem thành viên, team của org | `gh api orgs/hcmute-dsc/members` |
| `project` | tạo/sửa GitHub Project, field, item | `gh project item-list 1 --owner hcmute-dsc` |
| `admin:org` | **không cấp mặc định.** Chỉ cần cho secret **của org**, hook của org | `gh secret set X --org hcmute-dsc` |

Nguyên tắc: chỉ cấp quyền khi cần. `admin:org` cho `gh` quyền quản trị cả org, nên mọi secret của Learning Hub được đặt **theo repo** để khỏi cần quyền này.

## 4. Các lệnh vận hành hay dùng

```bash
# Deploy lại (sau khi sửa .env trên VPS) và theo dõi
gh workflow run deploy.yml --repo hcmute-dsc/learning-hub --ref main
gh run watch --repo hcmute-dsc/learning-hub

# Xem các lần chạy gần nhất
gh run list --repo hcmute-dsc/learning-hub --limit 5

# Secret: đặt (dán giá trị khi được hỏi, KHÔNG dùng --body với giá trị thật), liệt kê
gh secret set DISCORD_ALERT_WEBHOOK_URL --repo hcmute-dsc/learning-hub
gh secret list --repo hcmute-dsc/learning-hub
gh secret list --env production --repo hcmute-dsc/learning-hub

# Biến (không bí mật)
gh variable set DEPLOY_ENABLED --repo hcmute-dsc/learning-hub --body true

# Project và issue
gh project item-list 1 --owner hcmute-dsc
gh issue create --repo hcmute-dsc/learning-hub --title "[Ops] …" --body-file mo-ta.md
```

Không dán secret vào lệnh (`--body "giá trị"`): nó sẽ nằm lại trong lịch sử của shell. Để `gh secret set` tự hỏi giá trị, hoặc đọc từ file: `gh secret set VPS_SSH_KEY … < learning-hub-deploy`.

Danh sách đầy đủ secret và biến của production: [DEPLOYMENT §1](DEPLOYMENT.md#1-hiện-trạng-production).

## 5. Lỗi đã gặp

| Thông báo | Nguyên nhân | Cách sửa |
| --- | --- | --- |
| `your authentication token is missing required scopes [read:project]` | chưa có quyền `project` | `gh auth refresh -h github.com -s project` |
| `failed to fetch public key: HTTP 403 … needs the "admin:org" scope` khi `gh secret set --org` | secret của org cần `admin:org` | dùng secret **của repo** (`--repo …`) thay vì `--org`. Chỉ cấp `admin:org` khi thật sự cần secret dùng chung cho nhiều repo |
| `HTTP 404` với repo/org đang có | tài khoản không phải thành viên org, hoặc chưa được cấp quyền repo | nhờ admin org thêm vào org/team |
| `gh: command not found` (hoặc `'gh' is not recognized`) | chưa cài, hoặc terminal mở trước khi cài | cài lại theo mục 1, mở terminal mới |

## 6. Bàn giao

1. Người mới tự cài và đăng nhập `gh` bằng **tài khoản của họ** (mục 1–2). Không chia sẻ token.
2. Admin org thêm họ vào org với quyền phù hợp (owner nếu là tech lead).
3. Người cũ: `gh auth logout -h github.com` trên máy không còn dùng cho CLB.
4. Các token **riêng** gắn với tài khoản người cũ phải tạo lại bằng tài khoản người mới, ví dụ `ADD_TO_PROJECT_PAT` ([WORKFLOW §4](WORKFLOW.md)). Secret của deploy (`VPS_*`) không gắn với tài khoản: chỉ cần đổi khóa SSH theo [DEPLOYMENT §8](DEPLOYMENT.md#8-bàn-giao-cho-tech-lead-mới).
