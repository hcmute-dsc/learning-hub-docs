# Kiến trúc đang chạy: tài liệu bàn giao kỹ thuật

> Dành cho người mới nhận Learning Hub. Mỗi mục có một sơ đồ và vài dòng giải thích.
> Đối chiếu với code `hcmute-dsc/learning-hub@7a5ee54` và production ngày 19/11/2026.
> Thiết kế đầy đủ (gồm phần chưa làm): [ARCHITECTURE](../product/ARCHITECTURE.md).
> Cách vận hành: [DEPLOYMENT](DEPLOYMENT.md). Quy trình làm việc: [WORKFLOW](WORKFLOW.md).

Khi code đổi làm sơ đồ sai, sửa sơ đồ trong cùng PR, rồi cập nhật commit ở dòng đầu.

## Đọc theo thứ tự nào

1. §1 và §2: hệ thống gồm những gì, chạy ở đâu.
2. §3: code backend chia module ra sao.
3. §4 đến §6: ba luồng chính (đăng nhập, publish nội dung, nộp bài và review).
4. §7 và §8: code đi từ PR lên production thế nào, dữ liệu được sao lưu ra sao.
5. §9: trạng thái của bài nộp và phiên bản nội dung.
6. §10: những gì đang làm dở và chỗ sơ đồ sẽ phải sửa.

## 1. Bản đồ repo và dịch vụ

```mermaid
flowchart LR
  subgraph Users["Người dùng"]
    Mem["Thành viên"]
    Men["Mentor"]
    Adm["Admin / Ban chủ nhiệm"]
  end

  subgraph GitHub["GitHub org hcmute-dsc"]
    App["learning-hub<br/>code: api + web"]
    Reg["learning-hub-content<br/>registry: tracks.yaml"]
    Tracks["learning-hub-track-*<br/>mỗi track một repo"]
    Tpl["learning-hub-track-template"]
    Docs["learning-hub-docs<br/>tài liệu"]
    Board["Project #1<br/>HCMUTE DSC Learning Hub"]
    GApp["GitHub App<br/>hcmute-dsc-learning-hub"]
    OAuth["OAuth app<br/>đăng nhập"]
    GHCR["GHCR<br/>image api, web"]
  end

  VPS["VPS của CLB<br/>learn.hcmute-dsc.io.vn"]
  DC["Discord<br/>kênh CLB + kênh vận hành"]
  CF["Cloudflare DNS"]

  Users -->|HTTPS| CF --> VPS
  VPS -->|đăng nhập| OAuth
  VPS -->|đọc nội dung tại commit| Reg
  VPS -->|đọc nội dung tại commit| Tracks
  Tpl -. tạo repo mới .-> Tracks
  Reg -. ghim commit .-> Tracks
  App -->|Actions build| GHCR -->|deploy qua SSH| VPS
  VPS -->|webhook| DC
  App & Reg & Docs & Tracks -->|issue, PR| GApp --> Board
```

| Thành phần | Vai trò | Ghi chú |
| --- | --- | --- |
| `learning-hub` | Monorepo pnpm: `apps/api` (NestJS), `apps/web` (Next.js), `packages/content` (validator), `packages/shared` | Deploy tự động khi merge vào `main` |
| `learning-hub-content` | Registry: `tracks.yaml` ghim `{slug, repo, commit}` của từng track (ADR-10) | CI `validate` kiểm tra mọi track |
| `learning-hub-track-<slug>` | Bài học (Markdown) và bài tập (YAML) của một track | Tạo từ template bằng skill `new-track` ([NEW-TRACK](NEW-TRACK.md)) |
| GitHub App | Thêm issue và PR của mọi repo vào Project #1 | Biến `PROJECT_APP_CLIENT_ID` và secret `PROJECT_APP_PRIVATE_KEY` đặt ở cấp org ([WORKFLOW §4](WORKFLOW.md)) |
| OAuth app | Đăng nhập GitHub cho site | Client ID và secret nằm trong `.env` trên VPS |
| Discord | Webhook kênh CLB (`DISCORD_WEBHOOK_URL`) và kênh vận hành (`DISCORD_ALERT_WEBHOOK_URL`) | Chỉ gửi một chiều |

## 2. Các container trên VPS

```mermaid
flowchart TB
  Browser["Trình duyệt"] -->|"HTTPS learn.hcmute-dsc.io.vn"| Traefik

  subgraph VPS["VPS: Docker"]
    Traefik["Traefik có sẵn trên VPS<br/>network edge, TLS Let's Encrypt"]

    subgraph Stack["Compose stack learning-hub (/opt/learning-hub)"]
      Web["web: Next.js :3000<br/>rewrite /api/* sang api"]
      Api["api: NestJS :4000<br/>alias learning-hub-api"]
      Mig["migrate: chạy một lần mỗi deploy<br/>prisma migrate deploy"]
      Db[("db: postgres 16<br/>volume db-data")]
    end

    Cron["cron user learnhub<br/>backup.sh mỗi đêm<br/>offsite-export.sh Chủ nhật"]
    Bk[("/opt/learning-hub/backups")]
  end

  Traefik --> Web
  Web -->|"/api/v1/*"| Api
  Api --> Db
  Mig --> Db
  Cron -->|pg_dump| Db
  Cron --> Bk
```

- Chỉ container `web` nối vào network `edge` của Traefik. `api` và `db` chỉ nằm trong network riêng của stack, nên các site khác trên VPS không gọi tới được.
- Trình duyệt chỉ nói chuyện với `web`. Next.js chuyển mọi đường dẫn `/api/*` sang `http://learning-hub-api:4000`; địa chỉ này được ghi cứng lúc build image (`API_INTERNAL_URL`).
- Thứ tự khởi động: `db` khỏe → `migrate` chạy xong → `api` khỏe → `web`. Migration lỗi thì deploy dừng, container cũ vẫn chạy.
- Secret chỉ nằm trong `/opt/learning-hub/.env` trên VPS. Mẫu: `deploy/.env.example`.
- Chưa có: `sql-checker` (E13, [#3](https://github.com/hcmute-dsc/learning-hub/issues/3)) với PostgreSQL riêng trên mạng nội bộ riêng (ARCHITECTURE §7).

## 3. Module backend

`apps/api/src` chia thành:
- `modules/*`: nghiệp vụ;
- `integrations/*`: adapter GitHub, Discord và thư mục local;
- `infrastructure/*`: Prisma, config;
- `common/*`.

Mũi tên dưới đây là "import từ", gồm cả import kiểu sự kiện (event type).

```mermaid
flowchart BT
  catalog["catalog<br/>track, module, lesson, exercise"]
  users["users<br/>hồ sơ, vai trò, duyệt email"]
  identity["identity<br/>đăng nhập GitHub, session"]
  content["content<br/>sync, publish, ContentSource"]
  submissions["submissions<br/>quiz, self_check, nộp link"]
  learning["learning<br/>enroll, tiến độ, dashboard"]
  reviews["reviews<br/>hàng chờ, review, rubric"]
  notifications["notifications<br/>Discord"]
  audit["audit<br/>nhật ký"]
  overview["overview<br/>số liệu /admin"]
  health["health"]

  identity --> users
  content --> catalog
  content --> users
  submissions --> catalog
  learning --> catalog
  learning --> submissions
  learning --> users
  reviews --> catalog
  reviews --> learning
  reviews --> submissions
  reviews --> users
  notifications --> catalog
  notifications --> content
  notifications --> reviews
  notifications --> submissions
  notifications --> users
  audit --> content
  audit --> reviews
  audit --> users
  overview --> audit
  overview --> content
  overview --> reviews
  overview --> submissions
  overview --> users
```

Phản ứng phụ (ghi nhật ký, gửi thông báo, cập nhật tiến độ) chạy qua domain event trong cùng tiến trình (`@nestjs/event-emitter`):

```mermaid
flowchart LR
  subgraph Emit["Phát sự kiện"]
    U["users<br/>user.registered, approved,<br/>suspended, reactivated, roles"]
    C["content<br/>content.synced, published, rejected"]
    S["submissions<br/>submission.created, passed,<br/>approved, awaiting_review"]
    R["reviews<br/>review.claimed, released, completed"]
  end
  subgraph Listen["Lắng nghe"]
    AL["audit.listener<br/>ghi audit_logs"]
    SL["identity: suspension.listener<br/>xóa session"]
    CL["learning: completion.listener<br/>đánh dấu bài xong"]
    NL["notifications.listener<br/>gửi Discord"]
  end
  U --> AL
  U --> SL
  C --> AL
  C --> NL
  R --> AL
  R --> NL
  S --> CL
  S --> NL
```

Quy tắc ranh giới, có kiểm tra trong CI (`scripts/check-boundaries.mjs`, job `build-test`):
- Code trong `modules/*` không import từ `integrations/*`. Code nghiệp vụ chỉ dùng **port** (interface) khai báo trong module, ví dụ `ContentSource`, `IdentityProvider`, `NotificationChannel`.
- Chỉ file `*.module.ts` được nối adapter vào port.

| Port | Module sở hữu | Adapter đang dùng |
| --- | --- | --- |
| `ContentSource` | content | `RegistryContentSource` (production, `CONTENT_SOURCE=registry`), `GithubContentSource`, `LocalDirContentSource` (dev) |
| `IdentityProvider` | identity | `GithubIdentityProvider` |
| `NotificationChannel` | notifications | `DiscordWebhookChannel` |

## 4. Đăng nhập và duyệt thành viên

```mermaid
sequenceDiagram
  autonumber
  actor M as Thành viên
  participant W as web
  participant A as api: identity
  participant G as GitHub OAuth
  participant DB as postgres

  M->>W: Bấm "Đăng nhập bằng GitHub"
  W->>A: GET /api/v1/auth/github/login
  A-->>M: Chuyển tới GitHub, kèm state và PKCE
  M->>G: Đồng ý cấp read:user, user:email
  G-->>A: callback với code
  A->>G: Đổi code lấy token, đọc hồ sơ và email đã xác minh
  A->>DB: Tạo hoặc cập nhật user
  alt Email khớp approved_emails
    A->>DB: status = active
  else Không khớp
    A->>DB: status = pending
  end
  A->>DB: Tạo session, chỉ lưu token đã băm
  A-->>W: Cookie session httpOnly
  W-->>M: active thì vào /dashboard, pending thì vào /pending
```

- Admin thêm email vào danh sách duyệt ở `/admin/members`. User `pending` sẽ thành `active` ở lần đăng nhập sau.
- Khóa tài khoản phát `user.suspended`; listener của identity xóa mọi session của user đó ngay.

## 5. Publish nội dung qua registry

```mermaid
sequenceDiagram
  autonumber
  actor Mt as Mentor
  participant T as repo track
  participant R as learning-hub-content
  actor Ad as Admin
  participant A as api: content
  participant DB as postgres
  participant D as Discord

  Mt->>T: PR sửa bài, CI validate
  Mt->>T: Merge sau review
  Ad->>R: PR đổi commit của track trong tracks.yaml
  R->>R: CI validate mọi track tại commit đã ghim
  Ad->>R: Merge
  Ad->>A: Sync ở /admin/content
  A->>R: Đọc tracks.yaml ở main
  A->>T: Đọc file của từng track tại commit đã ghim
  A->>DB: content_versions draft, kèm diff và track_sources
  Ad->>A: Publish phiên bản draft
  A->>T: Đọc lại tại cùng commit, validate lại
  A->>DB: Upsert catalog trong một giao dịch, gỡ bài đã xóa thành archived
  A-->>D: content.published: bài thêm, sửa, gỡ và link so sánh của từng track
```

- Sync cùng một commit hai lần không tạo phiên bản mới. Mỗi lần chỉ có đúng một phiên bản ở trạng thái `published`.
- Không có bài học hay bài tập nào thay đổi thì không gửi Discord (learning-hub#26).
- Link "Sửa trên GitHub" trên trang bài học trỏ tới repo và commit của phiên bản đang publish (`sourceFor`, learning-hub#21).
- Thay vì bấm trên UI, có thể chạy lệnh: `pnpm --filter @hub/api content:sync [--sha <sha>] [--publish]`.

## 6. Nộp bài và review

```mermaid
sequenceDiagram
  autonumber
  actor M as Thành viên
  participant A as api
  participant DB as postgres
  participant D as Discord
  actor Mt as Mentor

  alt quiz
    M->>A: Gửi đáp án
    A->>DB: Chấm trên server, submission passed hoặc failed
  else self_check
    M->>A: Đánh dấu đã tự kiểm tra
    A->>DB: submission passed
  else project hoặc code
    M->>A: Nộp link GitHub
    A->>DB: submission awaiting_review
    A-->>D: Có bài cần review
    Mt->>A: Nhận bài ở /review
    A->>DB: review in_progress, chặn tự review
    Mt->>A: Nhận xét, rubric, chọn kết quả
    A->>DB: approved hoặc changes_requested
    A-->>D: Review xong
  end
  A->>DB: passed hoặc approved thì learning ghi bài đã xong
```

- Mỗi bài nộp chỉ có tối đa một review đang mở. Mentor có thể trả bài lại hàng chờ (`review.released`).
- Hàng chờ có bộ lọc theo track và số liệu cho mentor (learning-hub#30). Bộ lọc theo mentee chỉ hiện khi đã có ghép cặp (E8).
- Chấm tự động cho bài `code` (E6, [#4](https://github.com/hcmute-dsc/learning-hub/issues/4)) và cho SQL (E13, [#3](https://github.com/hcmute-dsc/learning-hub/issues/3)) chưa làm. Thiết kế ở ARCHITECTURE §7.

## 7. Từ PR tới production

```mermaid
flowchart LR
  Issue["Issue trên Project #1"] --> Branch["Nhánh feat/N-..."]
  Branch --> PR["Pull request"]

  subgraph Checks["Check bắt buộc trên main"]
    BT["build-test<br/>format, typecheck, test,<br/>boundaries, migrate, build, sync"]
    SE["secrets<br/>gitleaks"]
    TI["check<br/>tên PR"]
  end

  PR --> Checks
  Checks --> Review["Tự review, đăng lên PR<br/>maintainer xác nhận"]
  Review -->|squash merge| Main["main"]
  Main --> CI2["CI trên main"]
  CI2 --> Images["Deploy: build image<br/>api, web → GHCR, tag = commit"]
  Images --> SSH["SSH vào VPS<br/>chép compose và script"]
  SSH --> Up["compose pull, migrate, up -d"]
  Up --> Probe["Kiểm tra site trả lời<br/>lỗi thì job Deploy đỏ"]
  Uptime["Uptime: mỗi 15 phút"] -->|lỗi| Alert["Discord kênh vận hành"]
```

- `main` được bảo vệ từ 18/11/2026 ([WORKFLOW §7.5](WORKFLOW.md)):
  - bắt buộc PR, chỉ squash merge;
  - nhánh PR phải cập nhật theo `main` trước khi merge;
  - cấm force push.
- Deploy chạy lần lượt (`concurrency: deploy-production`). Image gắn tag theo commit, nên muốn quay lại bản cũ thì đặt `IMAGE_TAG` cũ (DEPLOYMENT).
- Nội dung không đi qua đường này: merge vào repo track không làm site đổi cho tới khi admin publish (§5).

## 8. Sao lưu

```mermaid
flowchart LR
  subgraph VPS["VPS"]
    Db[("postgres")]
    Daily["backup.sh<br/>19:30 UTC mỗi đêm<br/>giữ 14 ngày"]
    Week["offsite-export.sh<br/>Chủ nhật 20:00 UTC<br/>mã hóa age, giữ 4 bản"]
    Serve["offsite-serve.sh<br/>chỉ cho list và get file .age"]
  end
  subgraph Lead["Máy tech lead"]
    Pull["offsite-pull.ps1<br/>Task Scheduler thứ Hai 9:00<br/>giữ 12 bản"]
    Key["Khóa bí mật age<br/>có passphrase"]
    Check["offsite-restore-check.ps1<br/>thử khôi phục mỗi tháng"]
  end
  Db -->|pg_dump| Daily --> Week --> Serve
  Pull -->|SSH key riêng, bị giới hạn lệnh| Serve
  Key --> Check
  Pull --> Check
```

- VPS không giữ khóa giải mã. Lộ SSH key kéo bản sao thì người khác cũng chỉ lấy được file đã mã hóa.
- Cài đặt và bàn giao: `deploy/README.md` (learning-hub), DEPLOYMENT §5.

## 9. Trạng thái

Bài nộp (`submissions.status`):

```mermaid
stateDiagram-v2
  [*] --> passed: quiz đúng, self_check
  [*] --> failed: quiz sai
  [*] --> awaiting_review: nộp link project/code
  awaiting_review --> approved: mentor duyệt
  awaiting_review --> changes_requested: mentor yêu cầu sửa
  changes_requested --> [*]: nộp lại tạo submission mới
  failed --> [*]: làm lại tạo submission mới
  passed --> [*]
  approved --> [*]
```

Phiên bản nội dung (`content_versions.status`):

```mermaid
stateDiagram-v2
  [*] --> draft: sync hợp lệ
  [*] --> rejected: sync lỗi validate
  draft --> published: publish
  published --> archived: phiên bản khác được publish
```

Thành viên (`users.status`): `pending` → `active` (khớp email hoặc admin duyệt), rồi `active` ⇄ `suspended` (admin khóa và mở).

Bảng và quan hệ: [DATABASE](../product/DATABASE.md).

## 10. Đang làm dở, nhớ cập nhật sơ đồ

| Việc | Sơ đồ bị ảnh hưởng |
| --- | --- |
| Hint và lời giải [#46](https://github.com/hcmute-dsc/learning-hub/issues/46) | §6, §9 |
| Chứng nhận hoàn thành track [#47](https://github.com/hcmute-dsc/learning-hub/issues/47) | §3 (module mới), §9 |
| Huy hiệu và streak [#48](https://github.com/hcmute-dsc/learning-hub/issues/48) | §3 (listener mới) |
| Thông báo trong app và email [#49](https://github.com/hcmute-dsc/learning-hub/issues/49) | §1 (SMTP), §3, §6 (gửi qua `outbox_events` thay vì gửi nền trực tiếp) |
| Chấm SQL [#3](https://github.com/hcmute-dsc/learning-hub/issues/3), chấm qua checker [#4](https://github.com/hcmute-dsc/learning-hub/issues/4) | §2 (container mới), §6 |
| Ghép mentor [#7](https://github.com/hcmute-dsc/learning-hub/issues/7) | §3, §6 |

Hiện Discord được gửi nền ngay trong tiến trình API. Lỗi chỉ ghi log `Notification not sent` và không thử lại. Bảng `outbox_events` đã có nhưng chưa dùng. API cũng chưa có cron job nào; việc định kỳ chạy bằng cron trên VPS (§8) và GitHub Actions (§7).
