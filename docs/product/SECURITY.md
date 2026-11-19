# SECURITY

> Mô hình đe dọa: hệ thống nội bộ của CLB sinh viên, dữ liệu cá nhân mức vừa (tên, email, GitHub, bài làm), **không** xử lý thanh toán. Kẻ tấn công điển hình: người ngoài thử tài khoản, thành viên tò mò/gian lận, token bị lộ, nội dung/PR độc hại. Nguyên tắc: quyền tối thiểu, mặc định từ chối, ghi vết quản trị, đơn giản để duy trì.

## 1. Xác thực

- **GitHub OAuth** (authorization code + PKCE + `state` chống CSRF đăng nhập). Scope tối thiểu: `read:user`, `user:email` (bắt buộc: đọc email đã xác minh để duyệt theo danh sách), `read:org` chỉ khi bật kiểm tra org membership (tùy chọn). Không lưu token người dùng lâu dài (chỉ dùng trong lúc callback; nếu cần gọi API thay mặt user, lưu mã hóa AES-256-GCM, hết hạn ngắn).
- Không có mật khẩu → không có kho mật khẩu để lộ. Định danh bằng `github_id` (bất biến), không phải `login`.
- Session: ID ngẫu nhiên 256-bit, lưu **băm** trong DB; cookie `__Host-sid` (httpOnly, Secure, SameSite=Lax, Path=/); hết hạn tuyệt đối 14 ngày, trượt 7 ngày; đăng xuất & đổi vai trò làm mất hiệu lực; admin có thể thu hồi theo user.
- Tài khoản `pending`/`suspended` không có quyền truy cập dữ liệu (guard kiểm tra `status`).
- **Duyệt thành viên bằng danh sách email (đã chốt 16/11/2026).** Trong callback, API gọi `GET /user/emails` và lấy **mọi** email có `verified = true` (không chỉ email chính); chuẩn hóa chữ thường rồi so với bảng `approved_emails` (DATABASE §3.1b). Có email khớp → `active`; không khớp → `pending`, admin/mentor duyệt tay (`POST /admin/users/{id}/approve`). User `pending` đăng nhập lại sau khi email được thêm vào danh sách cũng thành `active`. Email chưa xác minh **không** bao giờ được dùng để khớp. Thành viên có email trường trong danh sách có thể thêm email đó vào GitHub làm email phụ đã xác minh. Membership org `hcmute-dsc` không còn là quy tắc tự duyệt; nếu bật thì chỉ là điều kiện bổ sung tùy chọn.
- Danh sách email là **dữ liệu cá nhân**: chỉ admin xem/sửa (nhập hoặc thay toàn bộ, vd dán từ form thành viên của CLB); mọi thay đổi ghi `audit_logs` (metadata chỉ ghi số lượng thêm/bớt, không chép danh sách); không có endpoint/xuất nào cho member hay mentor.
- Admin đầu tiên được bootstrap bằng biến môi trường `BOOTSTRAP_ADMIN_GITHUB_LOGIN` (luôn `active`), sau đó quản lý trong app.

## 2. RBAC & ranh giới phân quyền

Vai trò: `member`, `mentor`, `content_maintainer`, `admin` (PRD §5). Hai lớp:

1. **Vai trò** (guard `@Roles`).
2. **Quan hệ tài nguyên** (policy trong service): ví dụ — chỉ **chủ bài** hoặc **mentor đang được giao/nhận review** hoặc admin xem `submission`; mentor **không** review bài của chính mình; mentor chỉ thấy mentee của mình (hoặc hàng đợi chung nếu bật).

Quy tắc bắt buộc:
- Mặc định `AuthGuard` toàn cục; endpoint công khai phải khai báo `@Public()` tường minh.
- Kiểm tra quyền ở **server** cho mọi thao tác; frontend chỉ ẩn/hiện UI.
- Trả 404 thay vì 403 khi lộ sự tồn tại tài nguyên là nhạy cảm (bài nộp của người khác).
- Hành động nhạy cảm (gán vai trò, publish content, đổi mentor, duyệt user, xóa/ẩn danh dữ liệu) ghi `audit_logs`; người thực hiện ≠ người bị tác động khi cấp `admin`.
- Test tự động cho ma trận quyền (mỗi endpoint × vai trò) trong e2e.
- Thông báo trong app (F13): mọi truy vấn lọc theo `user_id` của session; đánh dấu đã đọc id của người khác trả 404. Nội dung thông báo cho mentor không có tên hay email thành viên; `link` chỉ là đường dẫn trong site.
- Link "Tắt email tóm tắt" là endpoint `@Public()` dùng `unsubscribe_token` 256 bit ngẫu nhiên làm thông tin xác thực; chỉ tắt được email, trả cùng một kết quả cho token không tồn tại. Trang web bắt bấm nút, vì trình quét mail tự mở link.
- **Hint và lời giải tham khảo** [P2, learning-hub#46]: `GET /exercises/{id}` chỉ trả nội dung các hint member **đã mở**; hint mở theo thứ tự, sau đủ `cost` lần nộp, và không mở được khi bài đang khóa (`POST /exercises/{id}/hints/{position}`, 409 nếu sai thứ tự hoặc chưa đủ lần nộp). Lời giải chỉ trả cho member đã **hoàn thành** bài (trạng thái `completed`: đạt, hoặc được duyệt nếu bài cần review; nộp mà chưa đạt thì chưa thấy), và luôn trả cho `mentor`/`admin` để review; `content_maintainer` không có quyền thêm. Lời giải không bao giờ nằm trong repo nội dung công khai (validator từ chối), chỉ đọc từ repo checker riêng tư bằng token chỉ đọc (`SOLUTIONS_GITHUB_TOKEN`). Quy tắc là hàm thuần có unit test: `apps/api/src/modules/submissions/hints.ts`.

## 3. Validate input

- Zod ở biên (DTO) cho body/query/params; từ chối trường lạ; giới hạn độ dài/kích thước (body ≤ 1 MB, comment ≤ 10 KB).
- URL nộp bài: parse bằng `URL`, chỉ `https://github.com/{owner}/{repo}/pull/{n}` hoặc `/commit/{sha}`; owner phải khớp `github_login` của người nộp (so khớp qua GitHub API theo `github_id`, không tin chuỗi); chặn redirect/host khác.
- Markdown (lesson, comment): render qua sanitize (`rehype-sanitize`), tắt HTML thô, chỉ cho phép giao thức `http/https/mailto`, thêm `rel="noopener noreferrer"` cho link ngoài.
- Tham số sắp xếp/lọc theo **danh sách trắng**.

## 4. Rate limiting

- Reverse proxy (Caddy rate-limit hoặc Nginx `limit_req`) + `@nestjs/throttler` trong API.
- Mặc định 100 req/phút/IP; auth 5/phút; `POST submissions` 10/phút/user và **giới hạn nộp lại**: tối đa 5 lần chạy checker/giờ/bài/user (bảo vệ quota Actions); webhook không tính theo IP mà theo chữ ký hợp lệ.
- Phản hồi 429 + `Retry-After`.

## 5. CSRF, XSS, SQL injection

| Mối đe dọa | Biện pháp |
| --- | --- |
| CSRF | Cookie `SameSite=Lax`; request thay đổi dữ liệu phải có header `X-CSRF-Token` (double-submit) và `Origin` khớp; không dùng GET để thay đổi trạng thái. |
| XSS | React escape mặc định; sanitize Markdown; **CSP** nghiêm (`default-src 'self'`; `script-src 'self'` + nonce; `img-src 'self' data: https://avatars.githubusercontent.com`; `frame-ancestors 'none'`; `worker-src 'self'` cho service worker Web Push); không `dangerouslySetInnerHTML` trừ trình render đã sanitize; cookie httpOnly; header `X-Content-Type-Options`, `Referrer-Policy`, HSTS. |
| SQL injection | Prisma (tham số hóa); cấm `$queryRawUnsafe`; mọi raw query dùng tagged template `Prisma.sql`; lint rule cấm. |
| IDOR | Policy theo tài nguyên (§2) + test. |
| SSRF | Không có tính năng fetch URL tùy ý từ server ngoài GitHub API đã định; host GitHub cố định. Web Push (MVP-Full): `endpoint` do trình duyệt cung cấp → chỉ nhận `https` và host thuộc danh sách push service đã biết (vd `fcm.googleapis.com`, `updates.push.services.mozilla.com`, `*.push.apple.com`, `*.notify.windows.com`); từ chối IP/host nội bộ. |
| Open redirect | `next=` chỉ nhận đường dẫn nội bộ (bắt đầu bằng `/` và không `//`). |

## 6. Upload file

MVP: **không có upload** (bài nộp là liên kết). Ảnh trong lesson đến từ content repo (qua PR + review), giới hạn kích thước/định dạng (png/jpg/webp/svg đã làm sạch). **Phase 2** (avatar/đính kèm nếu có): MinIO với bucket riêng, signed URL ngắn hạn, whitelist MIME + kiểm tra magic bytes, giới hạn 5 MB, đổi tên ngẫu nhiên, phục vụ từ domain riêng với `Content-Disposition: attachment` cho file không phải ảnh, (tùy chọn) quét ClamAV.

## 7. Chạy code của thành viên

**Quy tắc bất di bất dịch: không chạy code thành viên trong tiến trình/container của API hoặc web.**

### MVP — GitHub Actions trong repo chấm do CLB sở hữu
- Workflow ở `learning-hub-checker` (repo riêng tư, quyền ghi chỉ Ban kỹ thuật; branch protection; CODEOWNERS). Kích hoạt bằng `workflow_dispatch` từ GitHub App của hub.
- Chạy trên **runner GitHub-hosted** (VM tạm, bị hủy sau job) — cô lập khỏi hạ tầng CLB. **Không** dùng self-hosted runner cho code không tin cậy.
- Workflow checkout bài thành viên **vào thư mục riêng**, **không** chạy workflow/script từ repo thành viên (`persist-credentials: false`, không `pull_request_target` kiểu nguy hiểm), `permissions: contents: read`, **không secret** nào trong job (secret của repo chấm chỉ dùng ở job tách riêng không chạm code thành viên), `timeout-minutes: 5`.
- Chỉ nhận **repo công khai**; giới hạn kích thước/đường dẫn file được phép (`allowedPaths`); từ chối submodule, symlink ra ngoài, file > 1 MB.
- Kết quả nhận lại qua webhook `workflow_run` **chỉ từ `learning-hub-checker`** (kiểm tra `repository.full_name` + chữ ký) và định danh bằng `submissionId` do hub tạo (thành viên không can thiệp được giá trị này).
- Test ẩn/lời giải ở repo riêng tư; log công khai của job **không in** test ẩn.
- Nhận biết gian lận hiển nhiên: so khớp `sha`, `repo` với submission; hash file nộp; mentor review lại các bài `project`.

### MVP-Full — `sql-checker` (chỉ SQL, trên VPS của CLB; ADR-07)
Chạy **truy vấn SQL** của thành viên (không chạy code ngôn ngữ khác), nên chấp nhận được trên cùng VPS với hub nếu tuân thủ đủ các yêu cầu sau. Thiết kế: ARCHITECTURE §7 (sql-checker).

| Hạng mục | Yêu cầu |
| --- | --- |
| Cô lập DB | **PostgreSQL riêng** cho `sql-checker`, chỉ chứa dataset DSC Shop; không chung instance/volume/mật khẩu với DB hub; `sql-checker` không biết `DATABASE_URL` của hub |
| Mạng | Mạng Docker riêng, chỉ `api` gọi được; không publish port; container `sql-checker` và DB của nó **không có đường ra Internet** |
| Role | Truy vấn chạy bằng role ít quyền: không superuser, không `CREATEROLE`/`CREATEDB`, chỉ `SELECT` trên schema bài; không quyền trên `pg_catalog` ngoài mặc định |
| Giao dịch | `BEGIN READ ONLY` … `ROLLBACK` cho mọi bài đọc; bài sửa dữ liệu chạy trên schema/database dùng một lần, xóa sau khi chấm; chỉ một câu lệnh mỗi lần nộp (từ chối nhiều câu, `SET ROLE`, `SET SESSION AUTHORIZATION`) |
| Thời gian | `statement_timeout` 2–5 s, `idle_in_transaction_session_timeout`, `lock_timeout`; hủy kết nối khi quá hạn |
| Kết quả | Giới hạn số dòng (vd 1.000) và kích thước kết quả (vd 1 MB); cắt bớt thông báo lỗi trả về |
| Tính năng nguy hiểm | Cấm `COPY … PROGRAM`, `COPY` ra/vào file, `dblink`, `postgres_fdw`, ngôn ngữ untrusted (`plpython3u`, …), `lo_import/lo_export`; chỉ cài extension trong danh sách cho phép |
| Tài nguyên | Giới hạn CPU/bộ nhớ container (`cpus`, `mem_limit`), `pids-limit`; giới hạn kết nối đồng thời; rate limit nộp theo user (§4) |
| Dữ liệu chấm | Kết quả kỳ vọng/lời giải lấy từ `learning-hub-checker` (riêng tư), không trả về cho member; log không in lời giải |

### Tương lai — `learning-hub-runner` (chỉ khi cần chấm nhiều ngôn ngữ)
Dịch vụ **riêng**, không chung mạng/DB với API, và **không bao giờ chạy trên cùng VPS với hub**: Judge0/Piston cần container đặc quyền (privileged), một lần thoát sandbox sẽ chạm tới DB và secret của hub → nếu cần thì dùng một VPS nhỏ riêng. Yêu cầu sandbox tối thiểu:

| Hạng mục | Yêu cầu |
| --- | --- |
| Cô lập | Mỗi lần chạy một container/microVM riêng (gVisor/Firecracker/nsjail hoặc Judge0/Piston đã gia cố); không tái sử dụng giữa các user; chạy rootless, user không đặc quyền; `no-new-privileges`, bỏ toàn bộ capabilities, seccomp/AppArmor mặc định chặn syscall nguy hiểm |
| CPU | giới hạn (vd 0,5–1 vCPU/job), `cgroup cpu.max` |
| Bộ nhớ | giới hạn cứng (vd 256–512 MB), OOM kill, không swap |
| Thời gian | timeout tường (vd 5–10 s/test, 30 s tổng), kill cả process group |
| Filesystem | rootfs chỉ đọc; `tmpfs` ghi tạm có giới hạn (vd 64 MB); không mount host/socket Docker; xóa sạch sau job |
| Mạng | **tắt hoàn toàn** (`--network none`) trừ khi bài cần; nếu cần, allowlist đầu ra; chặn metadata service & mạng nội bộ |
| Tiến trình | giới hạn số process (`pids-limit`, chống fork bomb), số file mở, kích thước output (cắt log ≤ 64 KB) |
| Dữ liệu | không secret/biến môi trường nhạy cảm; test và dữ liệu đưa vào chỉ đọc |
| Giao tiếp | Hàng đợi một chiều từ API; kết quả trả bằng callback ký HMAC; runner không có quyền vào DB |
| Vận hành | Máy chủ riêng/VM riêng (không phải VPS của hub), cập nhật vá, giám sát, quota theo user |

Runner nhiều ngôn ngữ: chỉ Future, chưa xây ở MVP (ADR-07, ADR-08).

## 8. Quản lý secret

- Không commit secret; `.env.example` chỉ có tên biến. `gitleaks` trong CI cả hai repo (code và content).
- Production: biến môi trường qua trình quản lý chung của CLB (vd GitHub Actions Secrets/Environments cho deploy, file `.env` quyền 600 trên VPS hoặc Docker secrets). Danh sách secret & chủ sở hữu (ghi theo **chức danh**; hiện do Chủ nhiệm CLB giữ, xem ARCHITECTURE §14) nằm trong runbook bàn giao (không chứa giá trị).
- Secret chính: `GITHUB_APP_PRIVATE_KEY`, `GITHUB_WEBHOOK_SECRET`, `GITHUB_OAUTH_CLIENT_SECRET`, `SESSION/ENCRYPTION_KEY`, `DATABASE_URL`, `DISCORD_WEBHOOK_URL`; từ MVP-Full: `VAPID_PRIVATE_KEY` (Web Push; `VAPID_PUBLIC_KEY` không bí mật) và mật khẩu DB của `sql-checker`; từ P2: `SMTP_PASSWORD` (email tóm tắt; `SMTP_HOST/PORT/USER/FROM` không bí mật). SMTP phải dùng TLS (cổng 465) hoặc STARTTLS bắt buộc (587).
- Xoay vòng: khi đổi người giữ/hằng năm/khi nghi ngờ lộ; quy trình thu hồi ghi sẵn trong runbook.
- Log không chứa secret/token (redaction Pino).

## 9. Webhook

- **GitHub:** xác thực `X-Hub-Signature-256` bằng HMAC-SHA256 trên **raw body** với so sánh hằng thời gian (`timingSafeEqual`); từ chối nếu sai/thiếu; kiểm tra `X-GitHub-Event` thuộc danh sách cho phép; chống phát lại bằng `X-GitHub-Delivery` (bảng `webhook_deliveries`); kiểm tra `repository.full_name` thuộc danh sách tin cậy; trả 2xx nhanh; giới hạn kích thước body.
- **Discord:** chỉ **gửi** (incoming webhook URL là secret). Nếu có bot (P2), xác thực chữ ký interaction (Ed25519).

## 10. Quyền của GitHub App / token

| Đối tượng | Quyền | Ghi chú |
| --- | --- | --- |
| GitHub App (hub) | `Metadata: read`, `Contents: read`, `Pull requests: read`, `Actions: write` (chỉ để dispatch workflow ở checker repo), `Checks: read` | Cài đặt chỉ trên `learning-hub-content` và `learning-hub-checker`; không cài trên toàn org |
| OAuth App (đăng nhập) | `read:user`, `user:email` (đọc email đã xác minh), (`read:org` chỉ khi bật kiểm tra org tùy chọn) | Có thể dùng chính GitHub App (user-to-server) để gộp |
| PAT | **Không dùng** | Nếu bắt buộc, fine-grained, hết hạn ≤ 90 ngày, gắn tài khoản tổ chức, không phải cá nhân. Ngoại lệ tạm thời (MVP-Lite): `CONTENT_GITHUB_TOKEN` **tuỳ chọn** cho `GithubContentSource`, chỉ để nâng rate limit khi đọc content repo công khai — fine-grained, read-only, theo điều kiện trên; bỏ khi có GitHub App |
| Deploy | Token GitHub Actions mặc định `contents: read`; deploy key riêng | Không dùng quyền admin |

Bật 2FA bắt buộc cho tất cả thành viên Ban kỹ thuật và chủ org; bật branch protection cho `main` ở các repo; bật Dependabot + secret scanning + push protection.

## 11. Audit logging

Ghi vào `audit_logs` (append-only): đăng nhập thất bại bất thường [P2], cấp/thu hồi vai trò, duyệt/khóa user, thu hồi chứng nhận (`certificate.revoke`, kèm lý do), thay đổi danh sách email được duyệt, gán/đổi mentor, sync/publish/reject content, chạy lại test do admin, xóa/ẩn danh dữ liệu, thay đổi cấu hình. Trường: ai, hành động, đối tượng, thời điểm, IP/User-Agent (băm hoặc rút gọn), metadata (không chứa secret). Chỉ admin đọc. Giữ 24 tháng (DATABASE §9).

## 12. Bảo mật hạ tầng & chuỗi cung ứng

- Container non-root, image tối giản, quét (`trivy`) trong CI; cập nhật gói hằng quý; khóa phiên bản bằng `pnpm-lock.yaml`; hạn chế số dependency.
- Postgres không mở ra Internet; chỉ mạng Docker nội bộ; mật khẩu mạnh; backup mã hóa.
- TLS tự động (Caddy); HSTS; bật `trust proxy` đúng cách để lấy IP thật.
- Hành động CI/CD ghim theo SHA; `permissions:` tối thiểu; secrets chỉ ở môi trường `production` có người duyệt.

## 13. Riêng tư & tuân thủ

Thu thập tối thiểu (tên hiển thị, GitHub id/login, email nếu cần, avatar). Trang chính sách riêng tư bằng tiếng Việt ngắn gọn; thành viên có thể yêu cầu xóa/ẩn danh (DATABASE §8). Không bán/chia sẻ dữ liệu. Danh sách email được duyệt chỉ admin truy cập, không xuất cho member. **Bảng xếp hạng (MVP-Full): chỉ opt-in** — `users.leaderboard_opt_in` mặc định `false`, chỉ người đã bật mới xuất hiện; tắt là biến mất ngay; chỉ thành viên đã đăng nhập xem được, không có trang công khai ra Internet; không xếp hạng toàn thời gian. **Chứng nhận (P2, F19):** trang `/certificates/[code]` là trang công khai duy nhất có dữ liệu thành viên — chỉ tên hiển thị lúc cấp, track, ngày hoàn thành/cấp, repo + commit nội dung; không email, không id, không lý do thu hồi; `noindex` để tên không vào kết quả tìm kiếm; mã 120 bit ngẫu nhiên nên không dò được link của người khác, mã sai định dạng trả 404 trước khi đọc DB. Ẩn khỏi hồ sơ không thu hồi link đã chia sẻ (ghi rõ ở `/settings`). Subscription Web Push là dữ liệu thiết bị: xóa khi người dùng tắt, đăng xuất khỏi thiết bị đó hoặc bị xóa tài khoản. **Đã chốt (16/11/2026):** không cần đăng ký với nhà trường; dữ liệu thành viên lưu riêng tư (chỉ admin truy cập, không công khai, không chia sẻ ra ngoài).

## 14. Quản lý sự cố

Runbook 1 trang: phát hiện (cảnh báo Discord) → khoanh vùng (thu hồi session/secret, tạm tắt webhook) → khắc phục → thông báo thành viên nếu lộ dữ liệu → ghi lại bài học (post-mortem ngắn trong repo docs).
