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

## 2. Vòng đời một việc

1. **Tạo issue** trong repo đúng chỗ: code → `learning-hub`, bài học → `learning-hub-content`, tài liệu và việc của CLB → `learning-hub-docs`.
   - Tiêu đề có tiền tố: `[E13] …`, `[Content] …`, `[Ops] …`, `[Org] …`, `[Phase 0] …`, `[Bug] …`.
   - Thân issue có: bối cảnh (link tài liệu), checklist việc cần làm, **"Hoàn thành khi: …"**.
2. **Thêm vào project** (khung bên phải issue → Projects → DSC Learning Hub), điền Priority, Area, Epic, Size. Status = Backlog hoặc Ready.
3. **Nhận việc:** tự gán mình vào Assignees, chuyển Status = In progress, đặt Sprint.
4. **Mở PR:** trong mô tả ghi `Closes hcmute-dsc/<repo>#<số>` (hoặc `Closes #<số>` nếu cùng repo), rồi chuyển Status = In review.
5. **Merge:** issue tự đóng và tự sang Done (workflow có sẵn của project).

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

**Workflows** (⋯ → Workflows):
- *Item closed* và *Pull request merged* → Status = Done: bật sẵn, kiểm tra lại cho chắc.
- *Auto-add to project*: bật cho từng repo (`learning-hub`, `learning-hub-content`, `learning-hub-docs`), filter `is:issue,pr is:open`, để issue mới tự vào project.
- *Item added to project* → Status = Backlog.

**Quyền** (⋯ → Settings → Manage access): thêm team `mentors` / Ban chủ nhiệm với quyền *Write*. Thành viên chỉ cần *Read*.

## 5. Tạo lại bằng `gh` (khi cần)

Cần quyền `project` cho `gh`: `gh auth refresh -s project`.

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
