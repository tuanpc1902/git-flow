# 01 - Git Cơ Bản

## 1. Git là gì?

Git là **hệ thống quản lý phiên bản phân tán** (Distributed Version Control System - DVCS). Nó giúp bạn:

- **Theo dõi thay đổi** của source code theo thời gian
- **Làm việc nhóm** mà không ghi đè code của nhau
- **Quay lại phiên bản cũ** khi cần thiết
- **Phân nhánh** để phát triển tính năng song song

### So sánh: Có Git vs Không có Git

```
❌ Không có Git:
project_v1/
project_v2/
project_v2_final/
project_v2_final_real/
project_v2_final_real_THIS_ONE/

✅ Có Git:
project/          ← Chỉ 1 thư mục, Git quản lý toàn bộ lịch sử
```

---

## 2. Cài đặt Git

### Windows
```bash
# Tải từ https://git-scm.com/download/win
# Hoặc dùng winget
$ winget install --id Git.Git -e --source winget
```

### macOS
```bash
$ brew install git
```

### Linux (Ubuntu/Debian)
```bash
$ sudo apt-get update
$ sudo apt-get install git
```

### Kiểm tra cài đặt
```bash
$ git --version
> git version 2.43.0
```

---

## 3. Cấu hình Git (Quan trọng - Làm ngay sau khi cài)

### 3.1 Cấu hình thông tin cá nhân

```bash
# Tên hiển thị trong commit (BẮT BUỘC)
$ git config --global user.name "Tuan Pham"

# Email (BẮT BUỘC - nên dùng email GitLab)
$ git config --global user.email "tuan.pham@company.com"
```

### 3.2 Cấu hình hữu ích khác

```bash
# Đặt editor mặc định (VS Code)
$ git config --global core.editor "code --wait"

# Đặt branch mặc định là "main" (thay vì "master")
$ git config --global init.defaultBranch main

# Bật màu sắc cho output
$ git config --global color.ui auto

# Xử lý line ending (Windows)
$ git config --global core.autocrlf true

# Xử lý line ending (macOS/Linux)
$ git config --global core.autocrlf input

# Lưu credentials (không phải nhập password mỗi lần)
$ git config --global credential.helper store
```

### 3.3 Kiểm tra cấu hình

```bash
# Xem tất cả config
$ git config --list

# Xem một config cụ thể
$ git config user.name
> Tuan Pham

# Xem config ở đâu được set
$ git config --list --show-origin
```

### 📌 Ba cấp độ config

| Cấp độ | Flag | Áp dụng cho | File |
|--------|------|-------------|------|
| System | `--system` | Tất cả user trên máy | `/etc/gitconfig` |
| Global | `--global` | User hiện tại | `~/.gitconfig` |
| Local | `--local` | Repo hiện tại | `.git/config` |

> **Ưu tiên**: Local > Global > System

---

## 4. Khái niệm cốt lõi: 3 Khu vực của Git

```
┌─────────────────┐     git add      ┌─────────────────┐    git commit    ┌─────────────────┐
│  Working         │ ──────────────> │  Staging Area    │ ──────────────> │  Repository      │
│  Directory       │                 │  (Index)         │                 │  (.git)          │
│                  │ <────────────── │                  │                 │                  │
│  File đang chỉnh │  git restore    │  File sẵn sàng  │                 │  Lịch sử commit  │
│  sửa             │                 │  commit          │                 │  được lưu trữ    │
└─────────────────┘                  └─────────────────┘                 └─────────────────┘
```

### Giải thích:

1. **Working Directory**: Thư mục làm việc - nơi bạn chỉnh sửa file
2. **Staging Area (Index)**: Khu vực chuẩn bị - nơi bạn chọn những thay đổi muốn commit
3. **Repository (.git)**: Kho lưu trữ - nơi Git lưu toàn bộ lịch sử

### Ví dụ minh họa:

```
Tưởng tượng bạn đang đóng gói hàng để gửi:

1. Working Directory = Bàn làm việc (đang sắp xếp đồ)
2. Staging Area     = Hộp đóng gói (chọn đồ bỏ vào hộp)
3. Repository       = Kho hàng (hộp đã dán nhãn & lưu kho)

Bạn có thể chọn lọc đồ bỏ vào hộp (staging),
không nhất thiết phải gửi tất cả cùng lúc!
```

---

## 5. Các lệnh cơ bản

### 5.1 Khởi tạo Repository

```bash
# Tạo repo mới trong thư mục hiện tại
$ mkdir my-project
$ cd my-project
$ git init
> Initialized empty Git repository in /path/my-project/.git/

# Clone repo từ GitLab về máy
$ git clone https://gitlab.com/username/project.git
$ git clone git@gitlab.com:username/project.git    # SSH (khuyến nghị)
```

### 5.2 Kiểm tra trạng thái (Lệnh dùng nhiều nhất!)

```bash
$ git status

# Output mẫu:
> On branch main
> Changes not staged for commit:
>   (use "git add <file>..." to update what will be committed)
>         modified:   src/app.js          ← File đã sửa, chưa staging
>
> Untracked files:
>   (use "git add <file>..." to include in what will be committed)
>         src/utils.js                    ← File mới, Git chưa theo dõi
>
> no changes added to commit

# Dạng rút gọn
$ git status -s
>  M src/app.js        # M = Modified (đã sửa)
> ?? src/utils.js      # ?? = Untracked (chưa theo dõi)
```

### 5.3 Thêm file vào Staging Area

```bash
# Thêm 1 file cụ thể
$ git add src/app.js

# Thêm nhiều file
$ git add src/app.js src/utils.js

# Thêm tất cả file trong thư mục hiện tại
$ git add .

# Thêm tất cả file đã modified (không gồm file mới)
$ git add -u

# Thêm tất cả (kể cả file mới và đã xóa)
$ git add -A
# hoặc
$ git add --all

# Thêm từng phần của file (interactive)
$ git add -p src/app.js
# Git sẽ hỏi từng đoạn thay đổi:
# y = thêm đoạn này
# n = bỏ qua đoạn này
# s = chia nhỏ đoạn này hơn nữa
# q = thoát
```

### 5.4 Commit (Lưu thay đổi)

```bash
# Commit với message
$ git commit -m "feat: thêm chức năng đăng nhập"

# Commit với message nhiều dòng
$ git commit -m "feat: thêm chức năng đăng nhập

- Thêm form login
- Validate email và password
- Xử lý error message"

# Add + Commit cùng lúc (chỉ với file đã tracked)
$ git commit -am "fix: sửa lỗi validate email"

# Sửa commit cuối cùng (chưa push)
$ git commit --amend -m "feat: thêm chức năng đăng nhập v2"

# Thêm file vào commit cuối mà không đổi message
$ git add forgotten-file.js
$ git commit --amend --no-edit
```

### 5.5 Xem lịch sử commit

```bash
# Xem log đầy đủ
$ git log

# Log rút gọn 1 dòng
$ git log --oneline
> a1b2c3d feat: thêm chức năng đăng nhập
> e4f5g6h fix: sửa lỗi hiển thị
> i7j8k9l init: khởi tạo project

# Log với graph (hữu ích khi làm việc nhiều nhánh)
$ git log --oneline --graph --all
> * a1b2c3d (HEAD -> main) feat: thêm chức năng đăng nhập
> | * d4e5f6g (feature/signup) feat: thêm form đăng ký
> |/
> * e4f5g6h fix: sửa lỗi hiển thị

# Log của 1 file cụ thể
$ git log --oneline -- src/app.js

# Log với thay đổi chi tiết
$ git log -p

# Log N commit gần nhất
$ git log -5

# Log theo thời gian
$ git log --since="2024-01-01" --until="2024-12-31"

# Log theo author
$ git log --author="Tuan"

# Tìm commit chứa từ khóa trong message
$ git log --grep="đăng nhập"
```

### 5.6 Xem sự khác biệt (diff)

```bash
# Xem thay đổi chưa staging (Working Dir vs Staging)
$ git diff

# Xem thay đổi đã staging (Staging vs Last Commit)  
$ git diff --staged
# hoặc
$ git diff --cached

# So sánh 2 commit
$ git diff abc1234 def5678

# So sánh 2 branch  
$ git diff main feature/login

# Xem diff của 1 file cụ thể
$ git diff -- src/app.js

# Chỉ xem tên file thay đổi
$ git diff --name-only

# Xem thống kê tóm tắt
$ git diff --stat
> src/app.js  | 15 +++++++++------
> src/utils.js |  3 +++
> 2 files changed, 12 insertions(+), 6 deletions(-)
```

---

## 6. Hoàn tác (Undo) - Cực kỳ quan trọng!

### 6.1 Bỏ file khỏi Staging Area (unstage)

```bash
# Cách mới (khuyến nghị)
$ git restore --staged src/app.js

# Cách cũ
$ git reset HEAD src/app.js

# Unstage tất cả
$ git restore --staged .
```

### 6.2 Hủy thay đổi trong Working Directory

```bash
# ⚠️ CẢNH BÁO: Lệnh này KHÔNG THỂ hoàn tác!

# Khôi phục file về trạng thái commit cuối
$ git restore src/app.js

# Cách cũ  
$ git checkout -- src/app.js

# Khôi phục tất cả file
$ git restore .
```

### 6.3 Xóa file chưa tracked

```bash
# Xem file nào sẽ bị xóa (dry run)
$ git clean -n
> Would remove src/temp.js
> Would remove debug.log

# Xóa file chưa tracked
$ git clean -f

# Xóa cả thư mục
$ git clean -fd

# Xóa cả file trong .gitignore
$ git clean -fdx
```

---

## 7. File .gitignore

### 7.1 Tạo file .gitignore

```bash
# Tạo file .gitignore ở root project
$ touch .gitignore
```

### 7.2 Cú pháp .gitignore

```gitignore
# Comment - dòng ghi chú

# Bỏ qua file cụ thể
debug.log
secrets.json

# Bỏ qua theo extension
*.log
*.tmp
*.env

# Bỏ qua thư mục
node_modules/
dist/
build/
.idea/
.vscode/

# Bỏ qua thư mục ở bất kỳ đâu
**/logs/

# Ngoại lệ - KHÔNG bỏ qua file này
!important.log

# Bỏ qua file trong thư mục cụ thể
doc/*.pdf

# Bỏ qua đệ quy
doc/**/*.pdf
```

### 7.3 .gitignore phổ biến cho các dự án

```gitignore
# ===== Node.js =====
node_modules/
npm-debug.log*
.env
.env.local

# ===== Python =====
__pycache__/
*.py[cod]
venv/
.env

# ===== Java =====
*.class
target/
*.jar

# ===== IDE =====
.idea/
.vscode/
*.swp
*.swo
*~

# ===== OS =====
.DS_Store
Thumbs.db
desktop.ini

# ===== Build =====
dist/
build/
out/
```

### 7.4 Xử lý file đã tracked nhưng muốn ignore

```bash
# File đã được commit trước đó, giờ muốn ignore
# Bước 1: Thêm vào .gitignore
# Bước 2: Xóa khỏi tracking (giữ file trên máy)
$ git rm --cached secrets.json
$ git commit -m "chore: remove secrets.json from tracking"

# Xóa cả thư mục khỏi tracking
$ git rm --cached -r node_modules/
```

---

## 8. Xem thông tin chi tiết

### 8.1 git show

```bash
# Xem chi tiết commit cuối
$ git show

# Xem chi tiết 1 commit cụ thể
$ git show abc1234

# Xem nội dung file tại 1 commit
$ git show abc1234:src/app.js
```

### 8.2 git blame

```bash
# Xem ai sửa dòng nào trong file
$ git blame src/app.js

# Output:
> a1b2c3d4 (Tuan Pham  2024-01-15 10:30:00 +0700  1) import React from 'react';
> e5f6g7h8 (Minh Tran  2024-01-16 14:20:00 +0700  2) import { useState } from 'react';
> a1b2c3d4 (Tuan Pham  2024-01-15 10:30:00 +0700  3)
> i9j0k1l2 (Hoa Nguyen 2024-02-01 09:15:00 +0700  4) function App() {

# Blame một phạm vi dòng
$ git blame -L 10,20 src/app.js
```

---

## 9. Ví dụ thực tế: Workflow hàng ngày

```bash
# === Sáng: Bắt đầu ngày làm việc ===

# 1. Cập nhật code mới nhất
$ git pull origin main

# 2. Tạo nhánh mới cho task
$ git checkout -b feature/user-profile

# === Trong ngày: Code & Commit ===

# 3. Code xong một phần, kiểm tra
$ git status
$ git diff

# 4. Stage & Commit  
$ git add src/components/UserProfile.jsx
$ git add src/styles/profile.css
$ git commit -m "feat: tạo component UserProfile"

# 5. Tiếp tục code...
$ git add .
$ git commit -m "feat: thêm API lấy thông tin user"

# === Cuối ngày: Push code lên ===

# 6. Push lên GitLab
$ git push origin feature/user-profile

# 7. Tạo Merge Request trên GitLab
```

---

## 10. Tóm tắt lệnh cơ bản

| Lệnh | Mô tả | Ví dụ |
|-------|--------|-------|
| `git init` | Khởi tạo repo | `git init` |
| `git clone` | Clone repo | `git clone <url>` |
| `git status` | Xem trạng thái | `git status -s` |
| `git add` | Thêm vào staging | `git add .` |
| `git commit` | Lưu thay đổi | `git commit -m "msg"` |
| `git log` | Xem lịch sử | `git log --oneline` |
| `git diff` | Xem khác biệt | `git diff --staged` |
| `git restore` | Hoàn tác thay đổi | `git restore file.js` |
| `git show` | Xem chi tiết commit | `git show abc123` |
| `git blame` | Xem ai sửa dòng nào | `git blame file.js` |

---

**Tiếp theo**: [02 - Branching & Merging](02-branching-merging.md) →
