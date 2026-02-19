# 03 - Remote Repository (Làm việc với repo từ xa)

## 1. Remote là gì?

Remote repository là **bản sao của repo trên server** (GitLab, GitHub...), cho phép:

- **Chia sẻ code** với team
- **Backup** code trên cloud
- **Cộng tác** nhiều người cùng lúc

```
┌─────────────────┐          push           ┌─────────────────────┐
│   Máy của bạn   │ ─────────────────────>  │   GitLab Server     │
│   (Local Repo)  │                         │   (Remote Repo)     │
│                 │ <─────────────────────  │                     │
└─────────────────┘        pull/fetch        └─────────────────────┘
                                                     ↕
                                            ┌─────────────────────┐
                                            │  Máy đồng nghiệp    │
                                            │  (Local Repo)        │
                                            └─────────────────────┘
```

---

## 2. SSH Key (Quan trọng - Nên setup đầu tiên!)

### 2.1 Tại sao dùng SSH?

| HTTPS | SSH |
|-------|-----|
| Phải nhập username/password mỗi lần | Tự động xác thực |
| URL: `https://gitlab.com/...` | URL: `git@gitlab.com:...` |
| Đơn giản, dễ setup | Cần tạo key, nhưng tiện hơn về sau |

### 2.2 Tạo SSH Key

```bash
# Kiểm tra đã có SSH key chưa
$ ls ~/.ssh/
> id_rsa  id_rsa.pub    ← Nếu có, không cần tạo mới

# Tạo SSH key mới
$ ssh-keygen -t ed25519 -C "tuan.pham@company.com"
> Generating public/private ed25519 key pair.
> Enter file in which to save the key (/home/you/.ssh/id_ed25519): [Enter]
> Enter passphrase (empty for no passphrase): [Enter hoặc nhập password]

# Nếu hệ thống không hỗ trợ ed25519:
$ ssh-keygen -t rsa -b 4096 -C "tuan.pham@company.com"
```

### 2.3 Thêm SSH Key vào GitLab

```bash
# Bước 1: Copy public key
# Windows:
$ cat ~/.ssh/id_ed25519.pub | clip

# macOS:
$ cat ~/.ssh/id_ed25519.pub | pbcopy

# Linux:
$ cat ~/.ssh/id_ed25519.pub
# → Copy output thủ công

# Bước 2: Vào GitLab
# → User Settings → SSH Keys → Add new key
# → Paste key → Add key

# Bước 3: Test kết nối
$ ssh -T git@gitlab.com
> Welcome to GitLab, @your-username!
```

### 2.4 Chuyển repo từ HTTPS sang SSH

```bash
# Xem remote hiện tại
$ git remote -v
> origin  https://gitlab.com/user/project.git (fetch)
> origin  https://gitlab.com/user/project.git (push)

# Đổi sang SSH
$ git remote set-url origin git@gitlab.com:user/project.git

# Kiểm tra lại
$ git remote -v
> origin  git@gitlab.com:user/project.git (fetch)
> origin  git@gitlab.com:user/project.git (push)
```

---

## 3. Quản lý Remote

### 3.1 Xem Remote

```bash
# Xem tên remote
$ git remote
> origin

# Xem chi tiết URL
$ git remote -v
> origin  git@gitlab.com:team/project.git (fetch)
> origin  git@gitlab.com:team/project.git (push)

# Xem thông tin chi tiết remote
$ git remote show origin
> * remote origin
>   Fetch URL: git@gitlab.com:team/project.git
>   Push  URL: git@gitlab.com:team/project.git
>   HEAD branch: main
>   Remote branches:
>     main             tracked
>     feature/login    tracked
>     develop          tracked
```

### 3.2 Thêm / Xóa / Đổi tên Remote

```bash
# Thêm remote mới
$ git remote add origin git@gitlab.com:team/project.git

# Thêm remote phụ (VD: fork của bạn)
$ git remote add my-fork git@gitlab.com:tuan/project.git

# Đổi tên remote
$ git remote rename origin gitlab

# Xóa remote
$ git remote remove my-fork

# Đổi URL remote
$ git remote set-url origin git@gitlab.com:team/new-project.git
```

---

## 4. Clone (Tải repo về máy)

```bash
# Clone bằng SSH (khuyến nghị)
$ git clone git@gitlab.com:team/project.git

# Clone bằng HTTPS
$ git clone https://gitlab.com/team/project.git

# Clone vào thư mục tên khác
$ git clone git@gitlab.com:team/project.git my-project

# Clone 1 branch cụ thể
$ git clone -b develop git@gitlab.com:team/project.git

# Clone nông (shallow) - chỉ lấy N commit gần nhất (nhanh hơn)
$ git clone --depth 1 git@gitlab.com:team/project.git
# Hữu ích khi repo lớn và bạn chỉ cần code mới nhất

# Clone với submodules
$ git clone --recurse-submodules git@gitlab.com:team/project.git
```

---

## 5. Fetch, Pull, Push

### 5.1 Fetch (Tải về nhưng KHÔNG merge)

```bash
# Fetch tất cả branch từ origin
$ git fetch origin

# Fetch 1 branch cụ thể
$ git fetch origin main

# Fetch tất cả remotes
$ git fetch --all

# Fetch và xóa branch remote đã bị xóa
$ git fetch --prune
# hoặc
$ git fetch -p
```

**Fetch vs Pull:**
```
fetch: Tải dữ liệu → Lưu ở origin/main → KHÔNG merge
pull:  Tải dữ liệu → Lưu ở origin/main → TỰ ĐỘNG merge

Fetch an toàn hơn vì bạn có thể review trước khi merge:
$ git fetch origin
$ git log origin/main --oneline     # Xem commit mới
$ git diff main origin/main         # So sánh khác biệt
$ git merge origin/main             # Merge khi sẵn sàng
```

### 5.2 Pull (Tải về VÀ merge)

```bash
# Pull branch hiện tại
$ git pull origin main
# Tương đương:
# git fetch origin main + git merge origin/main

# Pull với rebase (KHUYẾN NGHỊ - lịch sử sạch hơn)
$ git pull --rebase origin main
# Tương đương:
# git fetch origin main + git rebase origin/main

# Cấu hình pull rebase mặc định
$ git config --global pull.rebase true

# Pull tất cả branch
$ git pull --all
```

### ⚠️ Xử lý khi Pull bị Conflict

```bash
$ git pull origin main
> Auto-merging src/app.js
> CONFLICT (content): Merge conflict in src/app.js

# Giải quyết:
# 1. Mở file, fix conflict
# 2. git add src/app.js
# 3. git commit

# Hoặc nếu dùng pull --rebase:
# 1. Mở file, fix conflict
# 2. git add src/app.js
# 3. git rebase --continue
```

### 5.3 Push (Đẩy code lên remote)

```bash
# Push branch hiện tại lên remote
$ git push origin main

# Push và set upstream (lần đầu push branch mới)
$ git push -u origin feature/login
# Sau đó chỉ cần:
$ git push    # Không cần chỉ định origin và branch

# Push tất cả branch
$ git push --all origin

# Push tags
$ git push --tags

# Force push (⚠️ NGUY HIỂM - ghi đè remote)
$ git push --force origin feature/login
# An toàn hơn:
$ git push --force-with-lease origin feature/login
# → Chỉ force push nếu remote không có commit mới từ người khác
```

### ⚠️ Khi nào bị Push rejected?

```bash
$ git push origin main
> ! [rejected]        main -> main (non-fast-forward)
> error: failed to push some refs

# Nguyên nhân: Remote có commit mà local chưa có
# Giải quyết:
$ git pull origin main     # Pull trước
$ git push origin main     # Push lại

# Hoặc:
$ git pull --rebase origin main
$ git push origin main
```

---

## 6. Tracking Branch

### 6.1 Upstream Branch là gì?

Upstream (tracking) branch là **liên kết giữa branch local và branch remote**.

```bash
# Xem upstream của branch hiện tại
$ git branch -vv
> * main           a1b2c3d [origin/main] feat: thêm đăng nhập
>   feature/login  e4f5g6h [origin/feature/login] wip: form login
>   local-only     i7j8k9l commit message          ← Không có upstream

# Set upstream cho branch
$ git branch --set-upstream-to=origin/main main
# hoặc khi push lần đầu:
$ git push -u origin feature/login
```

### 6.2 Checkout branch từ remote

```bash
# Xem branch remote
$ git branch -r
> origin/main
> origin/develop
> origin/feature/new-api

# Checkout và track branch remote
$ git checkout feature/new-api
# Git tự động tạo local branch và track origin/feature/new-api

# Nếu có nhiều remote, chỉ định rõ:
$ git checkout -b feature/new-api origin/feature/new-api
```

---

## 7. Tags (Đánh dấu phiên bản)

### 7.1 Tại sao dùng Tag?

Tag đánh dấu các **mốc quan trọng** trong project (releases, versions):

```
v1.0.0 ──── v1.1.0 ──── v1.2.0 ──── v2.0.0
   │            │            │           │
 Release    Minor fix    Feature     Major update
```

### 7.2 Tạo Tag

```bash
# Tag nhẹ (lightweight) - chỉ là pointer đến commit
$ git tag v1.0.0

# Tag có chú thích (annotated) - KHUYẾN NGHỊ
$ git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag 1 commit cụ thể
$ git tag -a v1.0.0 abc1234 -m "Release version 1.0.0"
```

### 7.3 Quản lý Tag

```bash
# Xem tất cả tag
$ git tag
> v1.0.0
> v1.1.0
> v2.0.0

# Xem tag theo pattern
$ git tag -l "v1.*"

# Xem chi tiết tag
$ git show v1.0.0

# Xóa tag local
$ git tag -d v1.0.0

# Xóa tag remote
$ git push origin --delete v1.0.0

# Push 1 tag lên remote
$ git push origin v1.0.0

# Push tất cả tags
$ git push origin --tags

# Checkout tại tag (detached HEAD)
$ git checkout v1.0.0

# Tạo branch từ tag
$ git checkout -b hotfix/v1.0.1 v1.0.0
```

---

## 8. Fork Workflow (Phổ biến trong open source & GitLab)

```
┌──────────────────┐          Fork           ┌──────────────────┐
│  Original Repo   │ ──────────────────────> │  Your Fork       │
│  (upstream)      │                         │  (origin)        │
│  team/project    │                         │  tuan/project    │
└──────────────────┘                         └──────────────────┘
         ↑                                           │
         │           Merge Request                   │
         └───────────────────────────────────────────┘
```

### Quy trình:

```bash
# 1. Fork repo trên GitLab (click nút Fork trên web)

# 2. Clone fork về máy
$ git clone git@gitlab.com:tuan/project.git
$ cd project

# 3. Thêm upstream remote (repo gốc)
$ git remote add upstream git@gitlab.com:team/project.git

# 4. Xác nhận remotes
$ git remote -v
> origin    git@gitlab.com:tuan/project.git (fetch)
> origin    git@gitlab.com:tuan/project.git (push)
> upstream  git@gitlab.com:team/project.git (fetch)
> upstream  git@gitlab.com:team/project.git (push)

# 5. Tạo branch & code
$ git checkout -b feature/awesome
# ... code ...
$ git commit -m "feat: awesome feature"

# 6. Đồng bộ với upstream trước khi push
$ git fetch upstream
$ git rebase upstream/main

# 7. Push lên fork
$ git push origin feature/awesome

# 8. Tạo Merge Request: fork → original repo
```

---

## 9. Ví dụ thực tế: Workflow hàng ngày với Remote

### Sáng đầu ngày
```bash
# Cập nhật code mới nhất
$ git checkout main
$ git pull origin main

# Tạo branch cho task mới
$ git checkout -b feature/JIRA-123-user-settings
```

### Trong ngày
```bash
# Code & commit thường xuyên
$ git add .
$ git commit -m "feat(settings): add notification preferences"

# Push lên remote (backup & chia sẻ tiến độ)
$ git push -u origin feature/JIRA-123-user-settings
```

### Cuối ngày / Khi hoàn thành
```bash
# Cập nhật main mới nhất
$ git fetch origin
$ git rebase origin/main

# Push (force-with-lease vì đã rebase)
$ git push --force-with-lease origin feature/JIRA-123-user-settings

# → Tạo Merge Request trên GitLab
```

### Sau khi MR được merge
```bash
# Cập nhật main
$ git checkout main
$ git pull origin main

# Xóa branch local
$ git branch -d feature/JIRA-123-user-settings

# Dọn dẹp remote branches đã xóa
$ git fetch --prune
```

---

## 10. Tóm tắt

| Lệnh | Mô tả |
|-------|--------|
| `git clone <url>` | Clone repo về máy |
| `git remote -v` | Xem danh sách remote |
| `git remote add <name> <url>` | Thêm remote |
| `git fetch origin` | Tải dữ liệu, không merge |
| `git pull origin main` | Tải + merge |
| `git pull --rebase` | Tải + rebase (gọn hơn) |
| `git push origin <branch>` | Đẩy code lên |
| `git push -u origin <branch>` | Push + set upstream |
| `git push --force-with-lease` | Force push an toàn |
| `git tag -a v1.0 -m "msg"` | Tạo tag |
| `git fetch --prune` | Xóa ref branch đã xóa trên remote |

---

← [02 - Branching & Merging](02-branching-merging.md) | **Tiếp theo**: [04 - Git Nâng Cao](04-git-nang-cao.md) →
