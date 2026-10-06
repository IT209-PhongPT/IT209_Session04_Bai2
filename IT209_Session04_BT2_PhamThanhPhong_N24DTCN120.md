# Báo cáo Bài tập 2: Quản lý Nhánh và Giải quyết Xung đột (Merge Conflict)

**Thông tin sinh viên:**
- **Họ và tên:** Phạm Thanh Phong
- **Mã sinh viên / Lớp:** N24DTCN120
- **Môn học:** IT209 - Phát triển ứng dụng Web / Cloud Infrastructure
- **Session:** Session 04 - Quản lý phiên bản mã nguồn với Git & GitHub
- **Bài tập:** Bài tập 2 (ex2)

---

## 1. Mục tiêu & Bối cảnh bài toán

### 1.1. Bối cảnh (Context)
Trong mô hình phát triển phần mềm cộng tác phân tán (Git Collaborative Workflow), các thành viên trong đội ngũ thường tách nhánh độc lập (`feature branches`) để phát triển tính năng mới. Trong khi đó, nhánh chính (`main` / `master`) vẫn liên tục nhận các bản vá lỗi hoặc tính năng mới từ các thành viên khác. Khi cả hai nhánh cùng chỉnh sửa nội dung trên cùng một dòng trong cùng một tệp tin (`README.md`), thuật toán tự động gộp của Git không thể tự phán đoán phiên bản nào là chính xác và sẽ nảy sinh xung đột gộp (**Merge Conflict**).

### 1.2. Mục tiêu kỹ thuật
- Thành thạo thao tác khởi tạo nhánh mới (`git checkout -b <branch_name>`), chuyển đổi qua lại giữa các nhánh (`git checkout <branch_name>`).
- Chủ động tạo ra tình huống xung đột gộp nhánh (Merge Conflict) tại tệp tin `README.md`.
- Đọc hiểu cấu trúc các thẻ đánh dấu xung đột (`<<<<<<<`, `=======`, `>>>>>>>`).
- Thực hiện xử lý xung đột thủ công (**Manual Conflict Resolution**), hợp nhất dữ liệu từ cả hai nhánh và hoàn tất quá trình gộp nhánh bằng một **Merge Commit**.
- Phân tích cơ chế gộp 3 vùng (**3-Way Merge**) và cấu trúc đồ thị nhánh thông qua `git log --graph --oneline`.

### 1.3. Ràng buộc kỹ thuật
- Tuyệt đối không sử dụng công cụ auto-resolve hoặc cờ gộp ép buộc mà phải xử lý thủ công từng khối conflict markers.
- Tệp tin sau khi gộp phải hoàn toàn sạch thẻ đánh dấu xung đột và có cấu trúc nội dung hợp lý.
- Lịch sử Git phải thể hiện rõ đồ thị phân nhánh và hội tụ với commit gộp có 2 nhánh tổ tiên (parents).

---

## 2. Cơ sở lý thuyết: Cơ chế 3-Way Merge & Xung đột nhánh

### 2.1. Bản chất cơ chế 3-Way Merge (Gộp 3 vùng)
Khi thực hiện gộp 2 nhánh đã phân nhánh (diverged), Git tìm kiếm commit tổ tiên chung gần nhất (**Common Ancestor / Base**) và so sánh 3 điểm:
1. **Base Commit (B):** Trạng thái tổ tiên chung của 2 nhánh.
2. **Current Branch Commit (HEAD / Ours):** Trạng thái commit trên nhánh hiện tại (`main`).
3. **Incoming Branch Commit (Theirs):** Trạng thái commit trên nhánh cần gộp (`feature-update`).

```
                    ┌─── [Commit A: feature-update] ───┐
                    │                                   │ (git merge)
[Base Commit] ──────┤                                   ▼
                    └─── [Commit B: main] ────────► [Merge Commit C]
```

- Nếu một dòng chỉ thay đổi trên một nhánh so với Base, Git sẽ tự động chấp nhận thay đổi đó.
- Nếu cùng một dòng bị thay đổi khác nhau trên **cả hai nhánh** so với Base, Git sẽ dừng tiến trình và kích hoạt trạng thái **Merge Conflict**.

### 2.2. Cấu trúc thẻ đánh dấu xung đột (Conflict Markers)
```text
<<<<<<< HEAD (Current change / Nhánh hiện tại main)
Dự án đã tích hợp API Gateway và Authentication Service trực tiếp trên nhánh main.
=======
Dự án đã hoàn thành module quản lý Kanban Board và gán thẻ công việc từ nhánh feature-update.
>>>>>>> feature-update (Incoming change / Nhánh được gộp)
```

---

## 3. Quy trình thực hiện chi tiết

### Bước 1: Khởi tạo Repository và Commit cơ sở trên nhánh `main`
```bash
# Khởi tạo repository với nhánh chính là main
git init -b main

# Cấu hình danh tính cục bộ
git config --local user.name "Pham Thanh Phong"
git config --local user.email "thanhphong102006@gmail.com"

# Tạo tệp tin README.md cơ bản
cat << 'EOF' > README.md
# Dự án Quản lý Tác vụ - Task Management System

- **Sinh viên:** Phạm Thanh Phong
- **MSSV:** N24DTCN120
- **Phiên bản:** v1.0.0

## Giới thiệu
Hệ thống quản lý tác vụ hỗ trợ làm việc nhóm và phân công công việc.

## Trạng thái triển khai
Dự án đang trong giai đoạn phát triển ban đầu.
EOF

# Đưa vào Staging Area và tạo Commit đầu tiên
git add README.md
git commit -m "feat: Initial commit with base README"
```

---

### Bước 2: Tạo nhánh `feature-update` và chỉnh sửa tệp `README.md`
```bash
# Tạo và chuyển sang nhánh feature-update
git checkout -b feature-update

# Chỉnh sửa nội dung phiên bản và trạng thái triển khai
# (Thêm chức năng Kanban Board & gán thẻ công việc)

# Commit thay đổi trên nhánh feature-update
git add README.md
git commit -m "feat(kanban): Update status and version in README from feature branch"
```

---

### Bước 3: Chuyển về nhánh `main` và tạo thay đổi xung đột
```bash
# Chuyển về nhánh chính
git checkout main

# Chỉnh sửa nội dung cùng các dòng trên nhánh main
# (Tích hợp API Gateway & Authentication Service)

# Commit thay đổi trên nhánh main
git add README.md
git commit -m "feat(auth): Update status and version in README from main branch"
```

---

### Bước 4: Thực hiện gộp nhánh và ghi nhận xung đột
```bash
# Thực hiện gộp nhánh feature-update vào main
git merge feature-update
```
**Output thông báo xung đột từ Git:**
```text
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

---

### Bước 5: Xử lý xung đột thủ công (Manual Resolution)
Mở tệp `README.md`, xóa bỏ hoàn toàn các thẻ đánh dấu `<<<<<<<`, `=======`, `>>>>>>>` và hợp nhất thông tin từ cả 2 nhánh:

```markdown
# Dự án Quản lý Tác vụ - Task Management System

- **Sinh viên:** Phạm Thanh Phong
- **MSSV:** N24DTCN120
- **Phiên bản:** v1.1.0-unified

## Giới thiệu
Hệ thống quản lý tác vụ hỗ trợ làm việc nhóm và phân công công việc.

## Trạng thái triển khai
Dự án đã tích hợp thành công cả hai tính năng sau khi giải quyết xung đột gộp nhánh:
1. **API Gateway & Authentication Service:** Xác thực người dùng và bảo mật hệ thống (từ nhánh `main`).
2. **Kanban Board & Task Assignment:** Quản lý bảng công việc trực quan và phân công nhiệm vụ (từ nhánh `feature-update`).
```

---

### Bước 6: Đánh dấu hoàn tất và tạo Merge Commit
```bash
# Đưa tệp tin đã giải quyết xung đột vào Staging Area
git add README.md

# Tạo commit gộp nhánh hoàn chỉnh
git commit -m "merge: Resolve conflict between main and feature-update in README.md"
```

---

## 4. Nhật ký thực thi & Minh chứng nghiệm thu (Verification Logs)

### 4.1. Minh chứng lịch sử đồ thị nhánh (`git log --graph --oneline`)
```bash
git log --graph --oneline
```
**Output thực tế:**
```text
*   d317846 merge: Resolve conflict between main and feature-update in README.md
|\  
| * 2e7dfca feat(kanban): Update status and version in README from feature branch
* | 2045892 feat(auth): Update status and version in README from main branch
|/  
* 5384345 feat: Initial commit with base README
```

---

### 4.2. Minh chứng chi tiết các commit tổ tiên của Merge Commit
```bash
git log -1 --pretty=fuller
```
**Output thực tế:**
```text
commit d317846c4f03940ad9547d894bcf9326e102f1a6
Merge: 2045892 2e7dfca
Author:     Pham Thanh Phong <thanhphong102006@gmail.com>
AuthorDate: Tue Oct 6 09:07:11 2026 +0700
Commit:     Pham Thanh Phong <thanhphong102006@gmail.com>
CommitDate: Tue Oct 6 09:07:11 2026 +0700

    merge: Resolve conflict between main and feature-update in README.md
```
*(Trường `Merge: 2045892 2e7dfca` xác nhận commit gộp sở hữu đúng 2 nhánh cha).*

---

### 4.3. Minh chứng trạng thái sạch của Working Tree (`git status`)
```bash
git status
```
**Output thực tế:**
```text
On branch main
nothing to commit, working tree clean
```

---

## 5. Hướng dẫn nộp bài & Cấu trúc kho lưu trữ

- **Đường dẫn trên GitHub:** `homework/session_04/ex2/`
- **Tệp báo cáo:** `README.md` (hoặc `IT209_Session04_BT2_PhamThanhPhong_N24DTCN120.md`)
- **Cấu trúc nộp bài chuẩn:**
```text
homework/session_04/ex2/
├── .git/
├── IT209_Session04_BT2_PhamThanhPhong_N24DTCN120.md
└── README.md
```

---

## 6. Tổng kết & Bài học kinh nghiệm

1. **Bản chất của Merge Conflict:** Xung đột không phải là lỗi phần mềm mà là cơ chế bảo vệ dữ liệu chủ động của Git nhằm ngăn chặn việc vô tình ghi đè mã nguồn của các đồng nghiệp.
2. **Kỹ năng xử lý xung đột thủ công:** Luôn kiểm tra kỹ ngữ cảnh của cả nhánh hiện tại (`HEAD`) và nhánh cần gộp (`Incoming`), giữ lại các thay đổi có ý nghĩa từ cả hai phía trước khi xóa thẻ đánh dấu xung đột.
3. **Quy tắc làm việc an toàn:** Sau khi sửa conflict, luôn kiểm tra lại cú pháp tệp tin, chạy thử chương trình, sử dụng `git add` và thực hiện commit gộp với thông điệp rõ ràng, minh bạch.
