# Báo cáo Bài 2: Tái cấu trúc lịch sử commit bằng Interactive Rebase

- **Môn học:** Java Fundamental - Session 05
- **Đường dẫn nộp bài trên GitHub:** `homework/session_05/ex2/`
- **Người thực hiện:** Huy-Tuan (sieuka1990@gmail.com)

---

## 1. Mục tiêu bài tập
- Làm chủ công cụ **Interactive Rebase** (`git rebase -i`) để chỉnh sửa, tổ chức lại lịch sử commit cục bộ trước khi đẩy (push) lên remote repository.
- Thực hiện **gộp nhiều commit nhỏ lẻ** (`squash`) thành một commit duy nhất có ý nghĩa, phản ánh đúng một đơn vị tính năng hoàn chỉnh.
- Đổi tên thông điệp commit (`reword`) để tuân thủ chuẩn commit message (Conventional Commits: `feat: ...`).
- Loại bỏ hoàn toàn các commit rác thử nghiệm (`drop`) không phục vụ sản phẩm thực tế (ví dụ: file log thử nghiệm, file tạm `temp.txt`).
- Nắm vững **The Golden Rule of Rebasing** và biết cách xử lý hoàn tác khi cần thiết (`git rebase --abort`, `git reflog`).

---

## 2. Kịch bản & Các bước thực hiện chi tiết

### Bước 1: Chuẩn bị 4 commit thử nghiệm ban đầu

Trong quá trình phát triển tính năng Module Authentication, nhánh làm việc cục bộ phát sinh 4 commit rời rạc, lộn xộn:

1. **Commit 1:** Tạo file `auth.js` với chức năng khởi tạo ban đầu:
   ```bash
   git add auth.js
   git commit -m "feat: khoi tao module auth"
   ```
2. **Commit 2:** Sửa một lỗi chính tả trong `auth.js`:
   ```bash
   git add auth.js
   git commit -m "fix typo"
   ```
3. **Commit 3:** Bổ sung các hàm tiện ích (`hashPassword`, `validateToken`) vào `auth.js`:
   ```bash
   git add auth.js
   git commit -m "adds utility functions"
   ```
4. **Commit 4:** Tạo file tạm debug `temp.txt` (commit rác):
   ```bash
   git add temp.txt
   git commit -m "add temp file for debug"
   ```

---

### Bước 2: Kiểm tra lịch sử commit TRƯỚC KHI REBASE

Kiểm tra bằng lệnh:
```bash
git log --oneline
```

**Kết quả màn hình Terminal:**
```text
834511d add temp file for debug
ed4bd2a adds utility functions
d12132e fix typo
1a868b0 feat: khoi tao module auth
```

> **Nhận xét:** Lịch sử gồm 4 commit rời rạc, có commit phụ (`fix typo`), commit rác (`temp.txt`). Cần tái cấu trúc lại trước khi tạo Pull Request để đảm bảo lịch sử mã nguồn chuyên nghiệp, dễ bảo trì.

---

### Bước 3: Thực hiện lệnh Interactive Rebase

Thực hiện rebase lùi về 4 commit trước đó:
```bash
# Trường hợp tính năng được rẽ nhánh từ commit trước:
git rebase -i HEAD~4

# Hoặc nếu 4 commit này tính từ commit gốc (root commit) của repo:
git rebase -i --root
```

---

### Bước 4: Giao diện soạn thảo cấu hình Interactive Rebase

Khi chạy lệnh trên, Git mở trình soạn thảo văn bản với danh sách các commit từ cũ nhất (trên) đến mới nhất (dưới).

#### 📸 Giao diện cấu hình thao tác (`git-rebase-todo`):

```text
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        GIT INTERACTIVE REBASE TODO LIST                                │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ pick 1a868b0 feat: khoi tao module auth                                                │
│ squash d12132e fix typo                                                                │
│ squash ed4bd2a adds utility functions                                                  │
│ drop 834511d add temp file for debug                                                   │
│                                                                                        │
│ # Rebase 5455fc1 onto 0000000 (4 commands)                                             │
│ #                                                                                      │
│ # Commands:                                                                            │
│ # p, pick <commit> = use commit                                                        │
│ # r, reword <commit> = use commit, but edit the commit message                         │
│ # e, edit <commit> = use commit, but stop for amending                                 │
│ # s, squash <commit> = use commit, but meld into previous commit                       │
│ # f, fixup [-C | -c] <commit> = like "squash" but discard only commit message          │
│ # x, exec <command> = run command (the rest of the line) using shell                   │
│ # b, break = stop here (continue rebase later with 'git rebase --continue')            │
│ # d, drop <commit> = remove commit                                                     │
│ # l, label <label> = label current HEAD with a name                                    │
│ # t, reset <label> = reset HEAD to a label                                             │
│ # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>] = create a merge commit   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

#### 🔍 Phân tích các lựa chọn thao tác:
- **Dòng 1 (`pick 1a868b0`):** Giữ lại commit đầu tiên làm commit nền tảng.
- **Dòng 2 (`squash d12132e`):** Đổi từ `pick` sang `squash` (hoặc viết tắt `s`) để gộp nội dung sửa typo vào commit 1a868b0 phía trước.
- **Dòng 3 (`squash ed4bd2a`):** Đổi từ `pick` sang `squash` để tiếp tục gộp các hàm tiện ích vào cùng commit.
- **Dòng 4 (`drop 834511d`):** Đổi từ `pick` sang `drop` (hoặc viết tắt `d`, hoặc xóa hẳn dòng này) để vứt bỏ hoàn toàn commit chứa file rác `temp.txt`.

Sau khi chỉnh sửa, lưu tệp và đóng trình soạn thảo.

---

### Bước 5: Soạn thảo thông điệp Commit mới sau khi gộp

Ngay sau khi lưu file cấu hình todo, Git tự động gộp các commit và mở màn hình soạn thảo commit message tiếp theo để người dùng đặt thông điệp tổng hợp.

#### 📸 Giao diện soạn thảo Commit Message:

```text
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        EDIT REBASE COMMIT MESSAGE                                      │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ feat: hoan thien module authentication                                                 │
│                                                                                        │
│ # This is a combination of 3 commits.                                                  │
│ # This is the 1st commit message:                                                      │
│ # feat: khoi tao module auth                                                           │
│ # This is the commit message #2:                                                       │
│ # fix typo                                                                             │
│ # This is the commit message #3:                                                       │
│ # adds utility functions                                                               │
│ # Please enter the commit message for your changes. Lines starting                     │
│ # with '#' will be ignored, and an empty message aborts the commit.                    │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

Ta xóa các dòng thông điệp cũ và thay thế bằng thông điệp chuẩn theo yêu cầu:
```text
feat: hoan thien module authentication
```

Lưu và đóng trình soạn thảo. Git sẽ thông báo rebase thành công:
```text
[detached HEAD 5455fc1] feat: hoan thien module authentication
 1 file changed, 15 insertions(+)
 create mode 100644 auth.js
Successfully rebased and updated refs/heads/master.
```

---

## 3. Kiểm tra kết quả sau Rebase (Verification)

### 3.1. Kiểm tra log commit sạch sẽ
Chạy lệnh:
```bash
git log --oneline
```

**Kết quả thực tế:**
```text
5455fc1 feat: hoan thien module authentication
```

> **Đạt yêu cầu:** Toàn bộ lịch sử chỉ còn **duy nhất 1 commit** sạch sẽ, đúng thông điệp quy định, các commit vụn vặt đã được gộp thành công.

---

### 3.2. Kiểm tra trạng thái working tree
Chạy lệnh:
```bash
git status
```

**Kết quả:**
```text
On branch master
nothing to commit, working tree clean
```

---

### 3.3. Kiểm tra file `temp.txt` đã bị xóa hoàn toàn
Kiểm tra xem file `temp.txt` còn tồn tại trong thư mục hay không:
```powershell
Test-Path temp.txt
# Kết quả: False
```

```bash
git log --all --full-history -- temp.txt
# Kết quả: Trống (không còn bất kỳ vết tích nào trong lịch sử commit)
```

---

### 3.4. Kiểm tra nội dung mã nguồn file `auth.js`
Nội dung file `auth.js` hiện tại tích hợp đầy đủ mọi thay đổi của 3 commit đã gộp:

```javascript
// Module Authentication
function login(username, password) {
    console.log("Đăng nhập người dùng:", username);
    return true;
}

function hashPassword(password) {
    return "hashed_" + password;
}

function validateToken(token) {
    return token && token.length > 10;
}

module.exports = { login, hashPassword, validateToken };
```

---

## 4. Bảng tra cứu các lệnh Interactive Rebase thường dùng

| Lệnh | Viết tắt | Ý nghĩa | Ứng dụng thực tế |
| :--- | :---: | :--- | :--- |
| `pick` | `p` | Sử dụng commit này nguyên vẹn | Giữ lại commit gốc hoặc commit quan trọng |
| `reword` | `r` | Sử dụng commit nhưng cho phép sửa lại message | Đổi thông điệp commit sai chính tả hoặc sai quy chuẩn |
| `edit` | `e` | Tạm dừng tại commit này để sửa đổi mã nguồn (`amend`) | Bổ sung file thiếu, tách 1 commit lớn thành nhiều commit |
| `squash` | `s` | Gộp commit vào commit liền trước và tổng hợp message | Gom các commit thử nghiệm nhỏ lẻ lại làm một |
| `fixup` | `f` | Tương tự `squash` nhưng vứt bỏ message của commit này | Gộp các commit fix nhỏ mà không cần sửa message commit trước |
| `drop` | `d` | Xóa bỏ commit khỏi lịch sử | Loại bỏ commit chứa mã độc, file rác, file nhạy cảm |
| `exec` | `x` | Chạy lệnh shell tương ứng sau commit đó | Tự động chạy test (`npm test`) kiểm tra từng commit |

---

## 5. Các lưu ý quan trọng (Best Practices)

1. **The Golden Rule of Rebasing (Quy tắc vàng):**
   > *Tuyệt đối KHÔNG rebase trên các nhánh công khai đã được chia sẻ với nhóm (như `main`, `master`, `develop`, `staging`).*
   - Rebase thay đổi mã hash của các commit, dẫn đến việc viết lại lịch sử (`rewriting history`).
   - Nếu đã đẩy lên remote mà rebase, các thành viên khác kéo về sẽ bị xung đột lịch sử nghiêm trọng.
   - Chỉ rebase trên các nhánh tính năng cá nhân (`feature branch`) cục bộ trước khi mở Pull Request / Merge Request.

2. **Cách hủy bỏ rebase khi gặp sự cố:**
   - Nếu trong quá trình rebase xảy ra xung đột không mong muốn hoặc thao tác nhầm:
     ```bash
     git rebase --abort
     ```
   - Nhánh sẽ lập tức quay trở lại trạng thái ban đầu trước khi gõ lệnh rebase.

3. **Cứu dữ liệu nếu đã hoàn thành rebase nhưng muốn quay lại:**
   - Sử dụng `git reflog` để tìm lại mã hash commit trước khi rebase:
     ```bash
     git reflog
     git reset --hard HEAD@{n}
     ```

---
