# FRONTEND-DESIGN — Next.js

> Cảm giác mong muốn: **hiện đại, hướng lập trình viên, thân thiện, hợp cộng đồng sinh viên**. Không giống LMS doanh nghiệp (bảng biểu dày, màu xám xanh), không gamification ồn ào. Gợi ý thị giác: nền sáng/tối theo hệ thống, font mono cho mã & nhãn, khối code nổi bật, nhiều khoảng trắng, màu nhấn theo nhận diện DSC. Có sẵn kinh nghiệm: `mentoring-program` đã có dark/light + vi/en + bố cục gọn, tái sử dụng token màu/logo.

## 1. Công nghệ

Next.js (App Router) + React + TypeScript; **Tailwind CSS** + bộ component **shadcn/ui (Radix)** — mã nằm trong repo, không khóa vendor, truy cập tốt. Không dùng Ant Design (nặng, giao diện "doanh nghiệp"; Study Jam UI dùng Ant — không tái dùng). Biểu tượng `lucide-react`. Markdown: `react-markdown` + `remark-gfm` + `rehype-sanitize` + `shiki` (tô màu code). Form: `react-hook-form` + Zod (dùng schema từ `packages/shared`). Dữ liệu client: `TanStack Query`. Test: Vitest + Testing Library; Playwright cho vài luồng chính.

## 2. Cấu trúc `apps/web`

```
apps/web/
  src/
    app/
      (public)/            # landing, đăng nhập, pending
      (app)/               # khu vực đã đăng nhập (layout có sidebar)
        dashboard/
        tracks/ [slug]/ …
        lessons/ [id]/
        exercises/ [id]/
        submissions/ [id]/
        mentorship/
        review/            # mentor
        admin/             # admin / content maintainer
      api/                 # route handler mỏng nếu cần (vd /api/auth/callback proxy)
      layout.tsx  error.tsx  not-found.tsx  loading.tsx
    components/
      ui/                  # shadcn primitives
      learn/               # TrackCard, LessonNav, ExerciseHeader, ProgressRing…
      submit/              # SubmitForm, TestRunPanel, AttemptHistory
      review/              # DiffViewer, CommentThread, ReviewForm
      layout/              # Sidebar, Topbar, Breadcrumbs
    features/              # logic theo miền: hooks + query keys + mappers
    lib/                   # api client (sinh từ OpenAPI), auth, i18n, utils
    styles/                # tokens, globals
  messages/                # vi.json, en.json (P2)
```

## 3. Route

Khớp INFORMATION-ARCHITECTURE. Đường dẫn dùng `slug` (đọc được) cho track/module, `id` cho lesson/exercise/submission (ổn định).

| Route | Nội dung | Render |
| --- | --- | --- |
| `/` | Landing + liên kết (site CLB, mentoring) | static |
| `/login`, `/pending` | Đăng nhập, chờ duyệt | static |
| `/dashboard` | Tiếp tục học, tiến độ, việc cần làm | dynamic |
| `/tracks`, `/tracks/[slug]` | Catalog, chi tiết track | server component + revalidate ngắn |
| `/tracks/[slug]/setup`, `/tracks/[slug]/resources` | Trang "Cài đặt", "Tài liệu thêm" của track (tuỳ chọn, learning-hub#44) | server |
| `/tracks/[slug]/[module]` | Module | server |
| `/lessons/[id]` | Lesson | server, cache theo `ETag` |
| `/exercises/[id]` | Đề + trạng thái + nộp bài | hybrid |
| `/submissions/[id]` | Chi tiết bài nộp, test run, review | dynamic, polling |
| `/me/submissions`, `/me/mentorship` | Lịch sử, mentor của tôi | dynamic |
| `/me/settings` | Cài đặt: thông báo (Web Push theo thiết bị), hiện mình trên bảng xếp hạng (opt-in) — MVP-Full | dynamic |
| `/leaderboard` | Bảng xếp hạng theo kỳ/đợt: top 10 + hạng của mình — MVP-Full | dynamic |
| `/review`, `/review/[submissionId]` | Hàng đợi, màn hình review | dynamic (mentor) |
| `/admin/...` | Users, mentorship, content versions, audit | dynamic (admin) |

Bảo vệ route: `middleware.ts` kiểm tra cookie session + vai trò (chỉ để redirect nhanh); **quyền thật** do API quyết định.

## 4. Kiến trúc component

- **Server Components** mặc định cho trang đọc; **Client Components** chỉ ở nơi có tương tác (form nộp bài, review, copy code).
- Component theo 3 tầng: `ui` (không biết nghiệp vụ) → `learn/submit/review` (biết miền, nhận props đã định dạng) → trang (lấy dữ liệu, ghép).
- Mô hình hiển thị trạng thái bài tập thống nhất `ExerciseState`: `locked | not_started | attempted | testing | needs_changes | in_review | completed` (suy ra từ API, một nơi duy nhất: `features/progress/state.ts`).

## 5. Quản lý state

| Loại state | Công cụ |
| --- | --- |
| Dữ liệu server | TanStack Query (key theo tài nguyên, `staleTime` ngắn; invalidate sau mutation) |
| Trạng thái đọc trang tĩnh | Server Components (không cần client state) |
| Form | react-hook-form |
| UI cục bộ | `useState`/URL search params (bộ lọc, tab) — **URL là nguồn sự thật cho bộ lọc** |
| Toàn cục | Hầu như không cần; `AuthContext` (user + roles) và theme. **Không** dùng Redux. |
| Realtime | Polling (2→10 s lùi dần) khi `submission.status ∈ {submitted, testing}`; SSE ở P2 |

## 6. Tích hợp API & xác thực

- Client TypeScript **sinh từ `openapi.json`** (openapi-typescript + fetch wrapper) đặt trong `packages/shared`; mọi response/lỗi có type.
- Cookie session `__Host-sid` httpOnly; Next gọi API cùng site (reverse proxy `/api` → Nest) nên không cần CORS phức tạp. Server Components gọi API bằng cách chuyển tiếp cookie của request.
- Luồng đăng nhập: nút "Đăng nhập bằng GitHub" → `GET /api/v1/auth/github/login` → GitHub → callback API → set cookie → redirect `/dashboard` (hoặc `/pending`).
- 401 → redirect `/login?next=…`; 403 → trang "không có quyền".
- Mutation thay đổi dữ liệu gửi `X-CSRF-Token` (đọc từ endpoint `/auth/me`).

## 7. Validation form & xử lý lỗi

- Dùng chung Zod schema với backend ⇒ cùng thông báo; hiển thị lỗi ngay dưới trường, `aria-describedby`, focus vào lỗi đầu tiên.
- Lỗi API theo `problem+json`: `code` → thông điệp tiếng Việt qua bảng `errorMessages[code]`; trường `errors[].path` gắn vào field.
- Phân tầng: lỗi trường → inline; lỗi thao tác → toast (kèm nút Thử lại); lỗi trang → `error.tsx` (có `requestId` để báo cáo); mất mạng → banner offline.
- Nộp bài: nút bị khóa khi đang gửi; gửi lại dùng cùng `Idempotency-Key` để tránh nộp trùng.

## 8. Trạng thái tải, rỗng, lỗi

| Màn hình | Loading | Empty | Error |
| --- | --- | --- | --- |
| Catalog | skeleton thẻ track | "Chưa có track nào được publish" + liên kết cho Ban | thử lại |
| Dashboard | skeleton | "Bạn chưa tham gia track nào → Khám phá" (CTA lớn) | thử lại |
| Hàng đợi review | skeleton hàng | "🎉 Hết bài cần review" | thử lại |
| Submission | thanh tiến trình "đang chạy test…" + ước tính | — | "Không đọc được kết quả → Chạy lại / báo lỗi" |
| Lesson | skeleton đoạn văn | — | thử lại |

Mỗi empty state có câu hướng dẫn hành động kế tiếp; không để trang trống.

## 9. Khả năng truy cập (a11y)

- Mục tiêu WCAG 2.1 AA: tương phản ≥ 4.5:1 (kiểm tra cả light/dark), focus ring rõ, điều hướng bàn phím đầy đủ, `skip to content`, landmark (`header/nav/main`), nhãn cho mọi control, không truyền đạt thông tin chỉ bằng màu (trạng thái có icon + chữ).
- Khối code: có thể focus, cuộn được, nút "Copy" có nhãn; Markdown heading đúng thứ tự.
- Tôn trọng `prefers-reduced-motion`, `prefers-color-scheme`.
- Kiểm thử: `eslint-plugin-jsx-a11y`, `axe` trong Playwright cho các trang chính.

## 10. Responsive

Mobile-first. Breakpoint: `sm 640`, `md 768`, `lg 1024`, `xl 1280`.
- ≤ md: sidebar thành menu dưới/hamburger; trang lesson một cột; bảng chuyển thành danh sách thẻ; nộp bài dùng form dọc.
- ≥ lg: sidebar cố định; trang lesson có mục lục bên phải; màn hình review dùng 2 cột (diff | nhận xét).
- Đọc lesson trên điện thoại là kịch bản chính của sinh viên → ưu tiên cỡ chữ/độ rộng dòng; nhập code/PR chủ yếu trên desktop (ghi chú trong UI).

## 11. i18n & nội dung

Giao diện tiếng Việt trước; chuỗi tách ra `messages/vi.json` (next-intl hoặc tương đương) để thêm `en` ở P2. Nội dung lesson có trường `language`; hiển thị nhãn ngôn ngữ ở catalog.

## 12. Hiệu năng

Lesson/track render server + cache; tách code theo route; ảnh dùng `next/image` (nguồn từ content, kích thước giới hạn); không gọi GitHub từ trình duyệt; font tự host. Mục tiêu LCP < 2,5 s trên 4G.

## 13. Gamification nhẹ (tắt được) & thông báo push

**P2:** vòng tiến độ, "streak" tuỳ chọn, huy hiệu mốc (hoàn thành track, review đầu tiên). Cài đặt trong hồ sơ: "Ẩn huy hiệu/streak".

**Bảng xếp hạng opt-in (MVP-Full, DECISIONS ADR-08).** Trang `/leaderboard`, chỉ thành viên đã đăng nhập:
- Chỉ hiện người đã bật "Hiện tôi trên bảng xếp hạng" ở `/me/settings` (mặc định **tắt**); người chưa bật thấy lời mời bật kèm giải thích ngắn.
- Theo học kỳ/đợt (bộ chọn kỳ/`cohort`, mặc định kỳ hiện tại), reset mỗi kỳ; **không** có tab "mọi thời đại".
- Xếp theo số bài hoàn thành có trọng số (`self_check` thấp hơn bài chấm tự động/mentor duyệt); **không** hiển thị thời gian hay tốc độ; đồng hạng hiển thị cùng hạng.
- Hiện **top 10** + một dòng "Hạng của bạn" (nếu đã bật). Không dùng màu/hiệu ứng "thua cuộc"; có liên kết về tiến độ cá nhân.

**Web Push (MVP-Full).** `/me/settings` có công tắc "Nhận thông báo trên thiết bị này" (xin quyền `Notification` chỉ khi người dùng bấm, không hỏi lúc tải trang); service worker `public/sw.js` + `manifest.webmanifest`. Trên iOS/iPadOS hiển thị hướng dẫn "Thêm vào Màn hình chính" trước khi bật. Bấm thông báo mở đúng trang (submission, hàng đợi review, track).
