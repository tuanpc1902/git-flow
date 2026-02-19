# 02 - Branching & Merging

## 1. Branch (Nhánh) là gì?

Branch là **bản sao độc lập** của code, cho phép bạn phát triển tính năng mới mà **không ảnh hưởng** đến code chính.

```
Tưởng tượng branch như đường rẽ trên con đường:

          feature/login
         /              \
────●────●────●──────────●────●──── main
              \          /
               feature/signup

- Bạn rẽ ra để làm việc riêng
- Khi xong, quay lại và merge vào đường chính
```

### Tại sao cần Branch?

| Không dùng Branch | Dùng Branch |
|-------------------|-------------|
| Code trực tiếp trên main | Mỗi feature/fix có nhánh riêng |
| Dễ conflict khi nhiều người | Làm việc song song, ít conflict |
| Code lỗi → ảnh hưởng tất cả | Code lỗi chỉ ở nhánh đó |
| Khó rollback | Dễ dàng bỏ nhánh lỗi |

---

## 2. Các lệnh Branch cơ bản

### 2.1 Xem danh sách Branch

```bash
# Xem branch local
$ git branch
> * main                    # * = branch hiện tại
>   feature/login
>   feature/signup

# Xem branch remote
$ git branch -r
>   origin/main
>   origin/feature/login

# Xem tất cả (local + remote)
$ git branch -a

# Xem branch với commit cuối
$ git branch -v
> * main           a1b2c3d feat: thêm đăng nhập
>   feature/login  e4f5g6h wip: form đăng nhập
```

### 2.2 Tạo Branch mới

```bash
# Tạo branch mới (KHÔNG chuyển sang)
$ git branch feature/login

# Tạo branch mới VÀ chuyển sang luôn (HAY DÙNG NHẤT)
$ git checkout -b feature/login
# hoặc (Git 2.23+)
$ git switch -c feature/login

# Tạo branch từ 1 commit cụ thể
$ git checkout -b hotfix/bug-123 abc1234

# Tạo branch từ branch khác
$ git checkout -b feature/login-v2 feature/login
```

### 2.3 Chuyển Branch

```bash
# Chuyển sang branch khác
$ git checkout feature/login
# hoặc (Git 2.23+)
$ git switch feature/login

# Chuyển về branch trước đó
$ git checkout -
# hoặc
$ git switch -

# ⚠️ LƯU Ý: Phải commit hoặc stash thay đổi trước khi chuyển branch!
$ git status    # Kiểm tra trước
$ git stash     # Lưu tạm nếu chưa muốn commit
$ git checkout main
```

### 2.4 Đổi tên Branch

```bash
# Đổi tên branch hiện tại
$ git branch -m ten-moi

# Đổi tên branch khác
$ git branch -m ten-cu ten-moi

# Đổi tên branch trên remote
$ git push origin :ten-cu ten-moi
$ git push origin -u ten-moi
```

### 2.5 Xóa Branch

```bash
# Xóa branch local (đã merge)
$ git branch -d feature/login
> Deleted branch feature/login

# Xóa branch local (chưa merge - FORCE)
$ git branch -D feature/login
> Deleted branch feature/login (was e4f5g6h)

# Xóa branch remote
$ git push origin --delete feature/login
# hoặc
$ git push origin :feature/login

# Xóa tất cả branch local đã merge (trừ main)
$ git branch --merged main | grep -v "main" | xargs git branch -d
```

---

## 3. Merge (Gộp nhánh)

### 3.1 Merge cơ bản

```bash
# Gộp feature/login vào main
# Bước 1: Chuyển sang branch đích (main)
$ git checkout main

# Bước 2: Merge branch nguồn vào
$ git merge feature/login
```

### 3.2 Các kiểu Merge

#### Fast-Forward Merge

Khi nhánh đích **không có commit mới** kể từ khi tạo nhánh nguồn:

```
TRƯỚC merge:
main:          A ── B
                     \
feature/login:        C ── D

SAU merge (fast-forward):
main:          A ── B ── C ── D     ← main chỉ "tiến tới" D
```

```bash
$ git checkout main
$ git merge feature/login
> Updating abc1234..def5678
> Fast-forward
>  src/login.js | 25 +++++++++++++++++++++++++
>  1 file changed, 25 insertions(+)
```

#### 3-Way Merge (Merge Commit)

Khi **cả hai nhánh đều có commit mới**:

```
TRƯỚC merge:
main:          A ── B ── E ── F
                     \
feature/login:        C ── D

SAU merge (3-way):
main:          A ── B ── E ── F ── G      ← G là merge commit
                     \             /
feature/login:        C ── D ─────
```

```bash
$ git checkout main
$ git merge feature/login
> Merge made by the 'ort' strategy.
>  src/login.js | 25 +++++++++++++++++++++++++
>  1 file changed, 25 insertions(+)

# Nếu muốn luôn tạo merge commit (kể cả fast-forward)
$ git merge --no-ff feature/login
```

### 3.3 Merge Request (GitLab) / Pull Request (GitHub)

Thay vì merge trực tiếp bằng command line, trong thực tế team thường:

```
1. Push branch lên remote
   $ git push origin feature/login

2. Tạo Merge Request trên GitLab
   - Vào GitLab → Repository → Merge Requests → New
   - Source: feature/login
   - Target: main
   - Viết mô tả thay đổi
   
3. Code Review
   - Đồng nghiệp review code
   - Comment, đề xuất chỉnh sửa
   
4. Merge trên GitLab
   - Sau khi được approve
   - Click "Merge" trên GitLab
```

---

## 4. Xử lý Conflict (Xung đột)

### 4.1 Conflict xảy ra khi nào?

Khi **2 người cùng sửa 1 dòng** trong cùng 1 file:

```bash
# Developer A (trên branch main):
function hello() {
    return "Hello World";    ← Sửa dòng này
}

# Developer B (trên branch feature/login):
function hello() {
    return "Xin chào";       ← Cũng sửa dòng này
}

# → CONFLICT khi merge!
```

### 4.2 Nhận biết Conflict

```bash
$ git merge feature/login
> Auto-merging src/app.js
> CONFLICT (content): Merge conflict in src/app.js
> Automatic merge failed; fix conflicts and then commit the result.

$ git status
> On branch main
> You have unmerged paths.
>   (fix conflicts and run "git commit")
>
> Unmerged paths:
>   both modified:   src/app.js
```

### 4.3 Conflict trông như thế nào?

```javascript
function hello() {
<<<<<<< HEAD
    return "Hello World";        // ← Code trên branch hiện tại (main)
=======
    return "Xin chào";           // ← Code trên branch đang merge (feature/login)
>>>>>>> feature/login
}
```

Giải thích:
- `<<<<<<< HEAD`: Bắt đầu code của branch hiện tại
- `=======`: Phân cách giữa 2 phiên bản
- `>>>>>>> feature/login`: Kết thúc code của branch đang merge

### 4.4 Cách giải quyết Conflict

```bash
# Bước 1: Mở file conflict và chọn code muốn giữ

# Giữ code của mình:
function hello() {
    return "Hello World";
}

# Hoặc giữ code của họ:
function hello() {
    return "Xin chào";
}

# Hoặc kết hợp cả hai:
function hello(lang) {
    if (lang === "vi") return "Xin chào";
    return "Hello World";
}

# Bước 2: Xóa hết markers (<<<<, ====, >>>>)

# Bước 3: Stage file đã fix
$ git add src/app.js

# Bước 4: Commit
$ git commit -m "fix: resolve merge conflict in app.js"
```

### 4.5 Dùng VS Code để giải quyết Conflict

VS Code hiển thị conflict rất trực quan:

```
┌──────────────────────────────────────────┐
│  Accept Current Change | Accept Incoming │
│  Change | Accept Both | Compare Changes  │
├──────────────────────────────────────────┤
│  <<<<<<< HEAD (Current Change)           │
│  return "Hello World";                   │
│  =======                                 │
│  return "Xin chào";                      │
│  >>>>>>> feature/login (Incoming Change) │
└──────────────────────────────────────────┘

Click:
- "Accept Current Change"  → giữ code nhánh hiện tại  
- "Accept Incoming Change" → giữ code nhánh đang merge
- "Accept Both Changes"    → giữ cả hai
- "Compare Changes"        → so sánh song song
```

### 4.6 Hủy merge khi bị Conflict

```bash
# Nếu chưa biết cách fix, có thể hủy merge
$ git merge --abort
# ← Quay lại trạng thái trước khi merge
```

---

## 5. Rebase

### 5.1 Rebase là gì?

Rebase **viết lại lịch sử** bằng cách đặt commits của bạn lên **đầu** branch đích:

```
TRƯỚC rebase:
main:          A ── B ── E ── F
                     \
feature/login:        C ── D

SAU rebase:
main:          A ── B ── E ── F
                                \
feature/login:                   C' ── D'    ← C', D' là bản copy mới
```

### 5.2 Cách dùng Rebase

```bash
# Đang ở branch feature/login
$ git checkout feature/login

# Rebase lên main
$ git rebase main

# Nếu có conflict → giải quyết từng commit:
# 1. Fix conflict trong file
# 2. git add <file>
# 3. git rebase --continue
# 4. Lặp lại nếu còn conflict

# Hủy rebase
$ git rebase --abort
```

### 5.3 Merge vs Rebase

| Merge | Rebase |
|-------|--------|
| Giữ nguyên lịch sử | Viết lại lịch sử |
| Tạo merge commit | Không tạo merge commit |
| Lịch sử phức tạp hơn | Lịch sử tuyến tính, sạch hơn |
| An toàn hơn | Cần cẩn thận hơn |
| Dùng cho merge vào main | Dùng để cập nhật feature branch |

### ⚠️ Quy tắc vàng của Rebase

> **KHÔNG BAO GIỜ rebase branch mà người khác đang dùng!**

```bash
# ❌ SAI - Đừng làm:
$ git checkout main
$ git rebase feature/login     # KHÔNG rebase main!

# ✅ ĐÚNG - Rebase branch của mình:
$ git checkout feature/login
$ git rebase main              # Rebase feature lên main
```

### 5.4 Interactive Rebase (Rất mạnh!)

```bash
# Chỉnh sửa N commit gần nhất
$ git rebase -i HEAD~3

# Editor mở ra:
pick a1b2c3d feat: thêm form login
pick e4f5g6h fix: sửa typo
pick i7j8k9l fix: sửa thêm typo

# Thay đổi thành:
pick a1b2c3d feat: thêm form login
squash e4f5g6h fix: sửa typo           # Gộp vào commit trước
squash i7j8k9l fix: sửa thêm typo      # Gộp vào commit trước

# Kết quả: 3 commits → 1 commit gọn gàng
```

**Các lệnh trong Interactive Rebase:**

| Lệnh | Viết tắt | Mô tả |
|-------|----------|--------|
| `pick` | `p` | Giữ commit nguyên vẹn |
| `reword` | `r` | Giữ commit, đổi message |
| `edit` | `e` | Dừng lại để sửa commit |
| `squash` | `s` | Gộp vào commit trước, giữ cả 2 message |
| `fixup` | `f` | Gộp vào commit trước, bỏ message này |
| `drop` | `d` | Xóa commit |
| `exec` | `x` | Chạy command |

---

## 6. Ví dụ thực tế

### Scenario 1: Phát triển tính năng mới

```bash
# 1. Cập nhật main
$ git checkout main
$ git pull origin main

# 2. Tạo feature branch
$ git checkout -b feature/user-profile

# 3. Code & commit
$ git add .
$ git commit -m "feat: tạo component UserProfile"
$ git add .
$ git commit -m "feat: thêm API user"

# 4. Cập nhật code mới từ main (rebase)
$ git fetch origin
$ git rebase origin/main

# 5. Push lên remote
$ git push origin feature/user-profile

# 6. Tạo Merge Request trên GitLab
```

### Scenario 2: Fix bug khẩn cấp (Hotfix)

```bash
# 1. Tạo hotfix từ main
$ git checkout main
$ git pull origin main
$ git checkout -b hotfix/critical-bug

# 2. Fix bug & commit
$ git add .
$ git commit -m "hotfix: fix critical login bug"

# 3. Push & tạo MR
$ git push origin hotfix/critical-bug
# → Tạo MR trên GitLab, merge ngay sau review
```

### Scenario 3: Dọn dẹp commit trước khi tạo MR

```bash
# Bạn có lịch sử commit lộn xộn:
$ git log --oneline
> a1b2c3d fix: oops
> e4f5g6h fix: typo again  
> i7j8k9l wip: forgot this
> m1n2o3p feat: add login form

# Dọn dẹp bằng interactive rebase:
$ git rebase -i HEAD~4

# Sắp xếp lại:
pick m1n2o3p feat: add login form
fixup i7j8k9l wip: forgot this
fixup e4f5g6h fix: typo again
fixup a1b2c3d fix: oops

# Kết quả: 1 commit sạch đẹp
$ git log --oneline
> x9y8z7w feat: add login form
```

---

## 7. Tóm tắt

| Lệnh | Mô tả |
|-------|--------|
| `git branch` | Xem danh sách branch |
| `git branch <name>` | Tạo branch mới |
| `git checkout -b <name>` | Tạo & chuyển sang branch mới |
| `git switch <name>` | Chuyển branch (Git 2.23+) |
| `git branch -d <name>` | Xóa branch đã merge |
| `git merge <branch>` | Merge branch vào branch hiện tại |
| `git merge --no-ff` | Merge luôn tạo merge commit |
| `git merge --abort` | Hủy merge |
| `git rebase <branch>` | Rebase lên branch |
| `git rebase -i HEAD~N` | Interactive rebase N commit |
| `git rebase --abort` | Hủy rebase |

---

← [01 - Git Cơ bản](01-git-co-ban.md) | **Tiếp theo**: [03 - Remote Repository](03-remote-repository.md) →
