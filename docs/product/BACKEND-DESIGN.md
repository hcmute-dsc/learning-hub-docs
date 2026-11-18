# BACKEND-DESIGN — NestJS

> NestJS + TypeScript, REST, OpenAPI (`@nestjs/swagger`), Prisma, Zod (qua `nestjs-zod`) cho validation DTO, Pino, Jest/Vitest (đề xuất **Vitest** cho `packages/shared` & web; **Jest** cho Nest vì tích hợp sẵn — chọn một và ghi trong `CONTRIBUTING`).

## 1. Cấu trúc thư mục

```
learning-hub/                  # monorepo (pnpm workspace)
  apps/
    api/
      src/
        main.ts
        app.module.ts          # nối module + gắn adapter vào port
        modules/
          identity/            # đăng nhập, session, guard
          users/               # hồ sơ, vai trò
          catalog/             # đọc track/module/lesson/exercise (published)
          content/             # sync từ Git, validate, versions, publish
          learning/            # enrollment, lesson progress, dashboard tiến độ
          submissions/         # nộp bài, vòng đời submission
          assessments/         # test_runs, chấm quiz, nhận kết quả CI
          reviews/             # review + comment
          mentorship/          # mentor_assignments
          notifications/       # phát thông báo (port NotificationChannel)
          audit/               # ghi/đọc audit_logs
        integrations/
          github/              # OAuth, Octokit, webhook, ContentSource adapter
          discord/             # webhook sender (P2: bot discord.js)
          webpush/             # [MVP-Full] WebPushChannel (VAPID, thư viện web-push)
          sql-checker/         # [MVP-Full] HTTP client gọi service sql-checker qua mạng nội bộ
        common/                # guards, filters, pipes, decorators, errors, pagination
        infrastructure/        # prisma, config (zod env), logging, scheduler, clock
      prisma/
        schema.prisma
        migrations/
        seed.ts
      test/                    # e2e (supertest) + factories
    web/                       # Next.js (xem FRONTEND-DESIGN)
  packages/
    shared/                    # Zod schema + type dùng chung, error codes, client OpenAPI sinh tự động
    content/                   # @hub/content: Zod schema file nội dung, parser/validator, CLI hub-content-validate
  content-example/             # cây nội dung mẫu nhỏ cho dev/test (sync bằng LocalDirContentSource)
  infra/                       # docker-compose, Caddyfile, workflows mẫu
  docs/
```

Cấu trúc mỗi module:

```
modules/submissions/
  submissions.module.ts
  api/            controllers, dto (zod)           # lớp trình bày
  application/    services (use case), event handlers
  domain/         entities/value objects/policies, domain events, errors   # chỉ khi có quy tắc đáng kể
  infrastructure/ prisma repositories, mapper
  ports/          interfaces module cần (vd RepoReader)
  index.ts        chỉ export API công khai (service, event, type)
```

**Domain model chỉ ở nơi có luật nghiệp vụ**: `submissions` (vòng đời trạng thái, trạng thái exercise và điều kiện mở khóa bài), `learning` (tiến độ, hoàn thành track), `mentorship` (state machine assignment), `content` (validate toàn vẹn). `users`, `catalog`, `audit` là CRUD/đọc, không cần lớp domain.

## 2. Module

Ký hiệu: C = controllers, S = application services, R = repositories, E = domain events phát ra, D = phụ thuộc.

### identity
- **Trách nhiệm:** OAuth GitHub flow, tạo/đóng session, guard xác thực, đọc email đã xác minh (`GET /user/emails`) và khớp với `approved_emails` (kiểm tra org: tùy chọn).
- C: `AuthController` (`GET /auth/github/login`, `GET /auth/github/callback`, `POST /auth/logout`, `GET /auth/me`).
- S: `AuthService`, `SessionService`. R: `SessionRepository`. Port: `IdentityProvider`.
- E: `UserLoggedIn`. D: `users`, `common`.

### users
- **Trách nhiệm:** hồ sơ, vai trò, tìm người dùng.
- C: `UsersController` (`GET /users/me`, `PATCH /users/me`), `AdminUsersController` (`GET/PATCH /admin/users`, `PUT /admin/users/{id}/roles`), `AdminApprovedEmailsController` (`GET/PUT /admin/approved-emails`, chỉ admin).
- S: `UsersService`, `RolesService`. R: `UserRepository`, `UserRoleRepository`.
- E: `UserRegistered`, `RoleGranted`, `RoleRevoked`. D: `common`.

### catalog
- **Trách nhiệm:** đọc nội dung đã publish (track/module/lesson/exercise) và quan hệ prerequisite.
- C: `TracksController`, `LessonsController`, `ExercisesController` (chỉ GET).
- S: `CatalogQueryService`. R: `CatalogRepository` (đọc; ghi do `content` thực hiện qua `CatalogWriter` port nội bộ của catalog).
- E: không. D: `common`.

### content
- **Trách nhiệm:** đọc repo nội dung qua `ContentSource`, validate (parser/validator dùng chung trong `packages/content` = `@hub/content`) + kiểm tra khóa tự nhiên so với DB, ghi `content_version`, diff, publish.
- **Sync** (`POST /admin/content/sync`, tuỳ chọn `commitSha`): đọc cây file tại commit SHA → validate → ghi `content_version` `draft` + `summary` diff (thêm/sửa/xóa/không đổi theo loại), hoặc `rejected` + danh sách lỗi (trả **422**). Idempotent theo `(source_repo, commit_sha)`. **Không ghi** bảng catalog.
- **Publish** (`POST .../{id}/publish`): đọc lại nguồn tại đúng commit SHA đã ghim, validate lại, rồi trong **một giao dịch**: version published trước → `superseded`; upsert chỉ item mới/đổi (theo `uid`, so `content_hash`) và gắn `content_version_id`; item biến mất → `archived` (không xóa cứng); thay `exercise_prerequisites`; version → `published`.
- C: `AdminContentController` (`POST /admin/content/sync`, `GET /admin/content/versions`, `GET .../{id}`, `POST .../{id}/publish`, `POST .../{id}/reject` [Full]). CLI cho MVP-Lite (chưa có UI admin): `pnpm --filter @hub/api content:sync [--sha <sha>] [--publish]`, gọi cùng service.
- S: `ContentSyncService`, `ContentPublisher`, `ContentDiffService` (validator là `parseContent()` của `@hub/content`). R: `ContentVersionRepository`. Port: `ContentSource` (`LocalDirContentSource` cho dev/test, `GithubContentSource`; chọn bằng env `CONTENT_SOURCE` — ARCHITECTURE §7).
- E: `ContentVersionDrafted`, `ContentVersionPublished`. D: `catalog` (qua `CatalogWriter`), `audit` (qua event).

### learning
- **Trách nhiệm:** enroll, lesson progress, tiến độ track, mục "Tiếp tục học", hoàn thành track, dashboard. Trạng thái khóa/mở của exercise **không** tính ở đây: lấy từ `ExerciseStateService` của `submissions`.
- C: `EnrollmentsController`, `MyEnrollmentsController` (`GET /me/enrollments`), `LessonsController` (`GET /lessons/{id}`, `PUT /lessons/{id}/progress`).
- S: `EnrollmentService`, `ProgressService`, `LessonService`; luật thuần trong `progress.ts`. R: `EnrollmentRepository`, `LessonProgressRepository`.
- E: `Enrolled`, `LessonCompleted`, `TrackCompleted`. D: `catalog`, `submissions` (chiều phụ thuộc: learning → submissions → catalog). Lắng nghe `SubmissionPassed` (sau này cả `SubmissionApproved`) để ghi hoàn thành track; khi đọc tiến độ cũng kiểm tra lại (lưới an toàn).

### submissions
- **Trách nhiệm:** tạo submission (idempotent), kiểm tra nguồn bài nộp, vòng đời trạng thái, danh sách/chi tiết.
- C: `SubmissionsController` (`POST /exercises/{id}/submissions`, `GET /submissions/{id}`, `GET /me/submissions`, `POST /submissions/{id}/rerun`).
- S: `SubmitSolutionService`, `SubmissionStateMachine` (domain). R: `SubmissionRepository`. Port: `RepoReader`.
- Trạng thái exercise của member (gồm khóa/mở theo prerequisite) là hàm thuần trong `submissions/exercise-state.ts` (`exerciseState`, `statesWithPrerequisites`); `ExerciseStateService` nạp dữ liệu và được export cho `learning`.
- E: `SubmissionCreated`, `SubmissionPassed`, `SubmissionFailed`, `SubmissionAwaitingReview`, `SubmissionApproved`, `SubmissionChangesRequested`. D: `catalog`, `common`. **Không bao giờ** import `learning` (learning phụ thuộc submissions, không ngược lại).

### assessments
- **Trách nhiệm:** tạo/cập nhật `test_runs`; chấm quiz trong app; xử lý kết quả CI nhận từ adapter; đối soát.
- C: `QuizAttemptController` (`POST /exercises/{id}/quiz-attempts`). Webhook controller **ở `integrations/github`**, gọi `AssessmentsService.recordCiResult()` qua port `CiResultReceiver` do `assessments` sở hữu.
- S: `QuizGradingService`, `CiResultService`, `ReconcileJob`. R: `TestRunRepository`.
- [MVP-Full] Port `SqlChecker` (adapter `integrations/sql-checker`): gửi `{submissionId, exerciseUid, sql}` tới service `sql-checker`, ghi `test_runs` (`provider = sql_checker`) — DECISIONS ADR-07, ARCHITECTURE §7.
- E: `TestRunFinished`. D: `submissions`, `catalog`.

### reviews
- **Trách nhiệm:** hàng đợi review, mở review, comment, kết luận.
- C: `ReviewQueueController` (`GET /reviews/queue`), `ReviewsController` (`POST /submissions/{id}/reviews`, `POST /reviews/{id}/comments`, `POST /reviews/{id}/complete`).
- S: `ReviewService`, `ReviewPolicy`. R: `ReviewRepository`, `ReviewCommentRepository`.
- E: `ReviewCompleted`. D: `submissions`, `mentorship`.

### mentorship
- **Trách nhiệm:** yêu cầu mentor, gán, kết thúc, danh sách mentee.
- C: `MentorshipController` (`POST /mentorship/requests`, `GET /me/mentorship`, `GET /mentor/mentees`), `AdminMentorshipController` (`POST /admin/mentorship/{id}/assign`).
- S: `MentorshipService`, `AssignmentStateMachine` (domain). R: `MentorAssignmentRepository`.
- Cấu hình (chốt theo học kỳ, không hard-code): `MENTOR_MAX_MENTEES` (mặc định 5), `REVIEW_TARGET_HOURS` (mặc định 72, chỉ là mục tiêu để hiển thị/đo). MVP đọc từ env; sau có thể chuyển thành setting admin.
- E: `MentorRequested`, `MentorAssigned`, `MentorshipEnded`. D: `users`.

### notifications
- **Trách nhiệm:** nhận domain event, chọn kênh/mẫu tin, gửi qua port `NotificationChannel`; (P2) lưu `notifications`.
- C: (P2) `NotificationsController`; [MVP-Full] `PushSubscriptionsController`. S: `NotificationDispatcher`. Port: `NotificationChannel` (adapter Discord + [MVP-Full] `WebPushChannel`, bảng `push_subscriptions`).
- D: lắng nghe event, `users`.

### audit
- **Trách nhiệm:** lắng nghe event "có ý nghĩa quản trị" và ghi `audit_logs`; API đọc cho admin (`GET /admin/audit-logs`).
- D: `common` (không phụ thuộc module khác; nhận event).

### integrations/github
Adapter: `GithubIdentityProvider`, `GithubContentSource` (GitHub REST, token tuỳ chọn ở MVP-Lite), `GithubRepoReader`, `GithubWebhookController`, `GithubAppAuth` (JWT → installation token). Không chứa luật nghiệp vụ; chỉ dịch giữa GitHub và port.

### integrations/discord
Adapter: `DiscordWebhookChannel` implements `NotificationChannel`. (P2) `DiscordBot` dùng `discord.js`.

## 3. Quy ước API

### 3.1 Phiên bản
Tiền tố URL: `/api/v1/...`. Thay đổi phá vỡ → `/api/v2`; v1 giữ tối thiểu 1 học kỳ. Thêm trường mới là tương thích.

### 3.2 Phân trang
Mặc định **cursor-based** cho danh sách có thể lớn (submissions, audit, queue): `?limit=20&cursor=<opaque>`; trả `{ "data": [...], "page": { "nextCursor": "…", "limit": 20 } }`. Danh sách nhỏ (tracks) dùng offset: `?page=1&pageSize=20` → `page: { page, pageSize, total }`. Tối đa `limit/pageSize = 100`.

### 3.3 Lọc & sắp xếp
Lọc bằng query param tường minh: `?level=beginner&tag=sql&status=awaiting_review`. Sắp xếp: `?sort=-submittedAt,title` (tiền tố `-` giảm dần), chỉ cho phép danh sách trắng trường (index-backed). Không cho phép filter tuỳ ý trên cột không có index.

### 3.4 Validation
Zod schema ở `packages/shared` (dùng chung với web). DTO = `createZodDto(schema)`; `ZodValidationPipe` toàn cục: strip trường lạ, từ chối sai kiểu → 422 (xem 3.6). Biến môi trường cũng validate bằng Zod lúc khởi động.

### 3.5 Xác thực & phân quyền
- Cookie session (`__Host-sid`) cho web; không có token trong localStorage. (Client không phải trình duyệt: P2, PAT riêng có scope.)
- `AuthGuard` toàn cục (mặc định yêu cầu đăng nhập, `@Public()` để mở), `RolesGuard` (`@Roles('mentor')`), và **policy theo tài nguyên** trong service (`ReviewPolicy.canReview(user, submission)`).
- CSRF: `SameSite=Lax` + kiểm tra header `Origin`/`X-CSRF-Token` cho request thay đổi dữ liệu.

### 3.6 Định dạng lỗi
Theo tinh thần RFC 9457 (problem+json):

```json
{
  "type": "https://hub.hcmute-dsc.io.vn/errors/exercise-locked",
  "title": "Bài tập đang bị khóa",
  "status": 409,
  "code": "EXERCISE_LOCKED",
  "detail": "Cần hoàn thành 'sql-select-basics' trước.",
  "errors": [{ "path": "exerciseId", "message": "prerequisite chưa đạt" }],
  "requestId": "01J…"
}
```

| HTTP | Dùng khi |
| --- | --- |
| 400 | JSON/tham số sai cú pháp |
| 401 | chưa đăng nhập |
| 403 | không đủ quyền |
| 404 | không tồn tại (hoặc ẩn vì quyền) |
| 409 | xung đột trạng thái (đã enroll, đã có review mở, bài bị khóa) |
| 422 | dữ liệu hợp cú pháp nhưng sai ràng buộc (Zod) |
| 429 | rate limit (kèm `Retry-After`) |
| 502/503 | lỗi GitHub/Discord hoặc DB |

### 3.7 Idempotency
- `POST /exercises/{id}/submissions` yêu cầu header `Idempotency-Key` (UUID); lưu vào `submissions.idempotency_key` (UQ theo user). Gửi lại cùng key → trả lại submission cũ (200), khác nội dung cùng key → 422.
- Enroll: idempotent theo UQ `(user, track)`.
- Webhook GitHub: idempotent theo `X-GitHub-Delivery` + UQ `(provider, external_run_id)`.
- Sync content: idempotent theo `(source_repo, commit_sha)` (sync lại cùng commit trả version đã có).

### 3.8 Khác
- Thời gian ISO-8601 UTC; id là UUID; trường JSON `camelCase` (map với cột `snake_case` ở repository).
- `ETag`/`If-None-Match` cho `GET` nội dung (lesson) để giảm tải.
- Tài liệu: Swagger UI `/api/docs` (chỉ non-production hoặc sau auth); `openapi.json` được CI xuất và sinh client TypeScript vào `packages/shared`.
- Rate limit: `@nestjs/throttler` — 100 req/phút/IP mặc định, 10/phút cho `POST submissions`, 5/phút cho auth endpoints.

## 4. Endpoint đại diện

| Method + Path | Quyền | Phase | Ghi chú |
| --- | --- | --- | --- |
| `GET /auth/github/login` | public | MVP | redirect GitHub |
| `GET /auth/me` | đã đăng nhập | MVP | user + roles |
| `GET /tracks?level=&tag=&page=` | đã đăng nhập | MVP | chỉ published |
| `GET /tracks/{slug}` | đã đăng nhập | MVP | gồm module + lesson + exercise tóm tắt, trạng thái mở khóa |
| `POST /tracks/{slug}/enrollments` | member | MVP | idempotent |
| `GET /me/enrollments` | member | MVP | kèm % tiến độ |
| `GET /lessons/{id}` | member | MVP | body Markdown |
| `PUT /lessons/{id}/progress` | member | MVP | `{status}` |
| `GET /exercises/{id}` | member | MVP | 403/409 nếu khóa |
| `POST /exercises/{id}/submissions` | member | MVP | header Idempotency-Key |
| `POST /exercises/{id}/quiz-attempts` | member | MVP | chấm tức thì |
| `GET /submissions/{id}` | chủ bài / mentor được giao / admin | MVP | + `testRuns[]`, `reviews[]` |
| `GET /me/submissions?exerciseId=` | member | MVP | cursor |
| `POST /mentorship/requests` | member | MVP | |
| `GET /reviews/queue?track=&mentee=` | mentor | MVP | cũ nhất trước, tối đa 200 dòng; `track` = slug, `mentee` = GitHub login (không phân biệt hoa thường); kèm `stats` (đang chờ, đã nhận, quá hạn, chờ lâu nhất, mentee không hoạt động ≥ 14 ngày) và `options` (track, mentee của mentor) |
| `POST /submissions/{id}/reviews` | mentor | MVP | mở review (409 nếu đã có) |
| `POST /reviews/{id}/comments` | reviewer | MVP | |
| `POST /reviews/{id}/complete` | reviewer | MVP | `{status: approved\|changes_requested, summary}` |
| `GET /mentor/mentees` | mentor | MVP | |
| `POST /admin/mentorship/{id}/assign` | admin | MVP | |
| `POST /admin/content/sync` | content_maintainer, admin | MVP | `{commitSha?}` → version `draft` + diff; 422 + danh sách lỗi (version `rejected`); không ghi catalog |
| `POST /admin/content/versions/{id}/publish` | admin | MVP | đọc lại tại commit SHA đã ghim, validate lại, ghi catalog trong 1 giao dịch; 422 nếu không còn hợp lệ |
| `PUT /admin/users/{id}/roles` | admin | MVP | ghi audit |
| `POST /admin/users/{id}/approve` | admin, mentor | MVP | `pending` → `active`, ghi audit |
| `GET /admin/approved-emails` | admin | MVP | danh sách email được duyệt; không mở cho vai trò khác |
| `PUT /admin/approved-emails` | admin | MVP | `{emails[], note?, mode: replace\|append}`; chuẩn hóa chữ thường, 1 giao dịch; audit chỉ ghi số lượng thêm/bớt |
| `POST /exercises/{id}/self-check` | member | MVP | `{completed:true}` cho bài `self_check` |
| `GET /admin/audit-logs` | admin | MVP | cursor |
| `POST /integrations/github/webhook` | chữ ký HMAC | MVP | không dùng session |
| `GET /me/achievements` | member | P2 | |
| `GET /me/notifications` | member | P2 | |
| `POST /me/push-subscriptions` | member | MVP-Full | `{endpoint, keys:{p256dh, auth}}`; upsert theo `endpoint`; 422 nếu host không thuộc danh sách push service |
| `DELETE /me/push-subscriptions/{id}` | member | MVP-Full | chỉ subscription của mình |
| `PATCH /me/settings` | member | MVP-Full | `{leaderboardOptIn}` |
| `GET /leaderboard` | member | MVP-Full | `?term=&cohort=`; top 10 + dòng của người xem; chỉ người opt-in; từ view `v_leaderboard_term` |

Ví dụ:

```http
POST /api/v1/exercises/0192…/submissions
Idempotency-Key: 6f1c0f5e-…
Content-Type: application/json

{ "sourceType": "github_pr", "url": "https://github.com/linh/dsc-git-ex1/pull/1", "note": "Em đã thêm .gitignore" }
```
```http
HTTP/1.1 202 Accepted
{ "data": { "id": "0192…", "attempt": 2, "status": "submitted", "exerciseId": "0192…" } }
```

## 5. Sự kiện domain (tóm tắt)

| Event | Phát bởi | Người nghe |
| --- | --- | --- |
| `UserRegistered` | users | audit |
| `SubmissionCreated` | submissions | assessments (khởi tạo `test_run` `queued`) |
| `TestRunFinished` | assessments | submissions (cập nhật trạng thái) |
| `SubmissionAwaitingReview` | submissions | notifications (kênh mentor) |
| `ReviewCompleted` | reviews | submissions (→ `approved`/`changes_requested`), notifications (member), learning |
| `SubmissionPassed/Approved` | submissions | learning (hoàn thành exercise/enrollment) |
| `Enrolled`, `LessonCompleted`, `TrackCompleted` | learning | audit, notifications (sau này) |
| `MentorRequested/Assigned` | mentorship | notifications |
| `ContentVersionPublished` | content | notifications, audit |
| `RoleGranted/Revoked` | users | audit |

Event in-process, đồng bộ trong cùng request cho phần phải nhất quán (cập nhật trạng thái), bất đồng bộ (sau commit) cho phản ứng phụ (thông báo).

## 6. Kiểm thử

| Tầng | Công cụ | Phạm vi |
| --- | --- | --- |
| Unit | Jest | state machine, policy, validator, grading |
| Integration | Jest + Postgres container (testcontainers hoặc service CI) | repository + Prisma + ràng buộc DB |
| E2E | supertest | luồng nộp bài, webhook (có chữ ký), review |
| Contract | OpenAPI snapshot | CI fail nếu thay đổi phá vỡ chưa bump version |
| Boundary | eslint-plugin-boundaries | vòng phụ thuộc |
