# 09 - Bài tập thực hành

> Thực hành trên máy tính của bạn. Mỗi bài tập có hướng dẫn chi tiết.

---

## Cấp độ 1: Cơ bản (Beginner)

### Bài 1: Khởi tạo & Commit đầu tiên

**Mục tiêu**: Hiểu `init`, `add`, `commit`, `status`, `log`

```bash
# Bước 1: Tạo thư mục project
$ mkdir git-practice
$ cd git-practice

# Bước 2: Khởi tạo Git
$ git init

# Bước 3: Tạo file
$ echo "# My Project" > README.md
$ echo "console.log('Hello World');" > app.js

# Bước 4: Kiểm tra trạng thái
$ git status
# → Quan sát: 2 file untracked (đỏ)

# Bước 5: Stage file
$ git add README.md
$ git status
# → Quan sát: README.md (xanh), app.js (đỏ)

# Bước 6: Stage tất cả
$ git add .
$ git status
# → Quan sát: cả 2 file đều xanh

# Bước 7: Commit
$ git commit -m "init: khởi tạo project"

# Bước 8: Xem log
$ git log --oneline
# → Quan sát: 1 commit xuất hiện
```

**Tự kiểm tra:**
- [ ] `git status` hiện "nothing to commit, working tree clean"
- [ ] `git log` hiện 1 commit

---

### Bài 2: Staging Area - Hiểu sâu hơn

**Mục tiêu**: Phân biệt Working Dir, Staging Area, Repository

```bash
# Bước 1: Sửa file
$ echo "console.log('Updated');" >> app.js

# Bước 2: Tạo file mới
$ echo "body { margin: 0; }" > style.css

# Bước 3: Kiểm tra
$ git status
# → app.js: modified (đỏ)
# → style.css: untracked (đỏ)

# Bước 4: Xem diff
$ git diff
# → Hiện thay đổi trong app.js

# Bước 5: Stage chỉ app.js
$ git add app.js
$ git status
# → app.js: staged (xanh)
# → style.css: vẫn untracked (đỏ)

# Bước 6: Xem staged diff
$ git diff --staged
# → Hiện thay đổi đã stage

# Bước 7: Sửa thêm app.js (SAU khi đã stage)
$ echo "// comment" >> app.js
$ git status
# → app.js xuất hiện CẢ trong staged (xanh) VÀ modified (đỏ)!
# → Điều này cho thấy Staging Area lưu snapshot tại thời điểm `git add`

# Bước 8: Commit (chỉ có phần đã staged)
$ git commit -m "feat: update app.js"

# Bước 9: Kiểm tra
$ git status
# → app.js: modified (biến đổi mới chưa stage)
# → style.css: untracked
```

**Câu hỏi tự kiểm tra:**
1. Tại sao app.js xuất hiện cả xanh lẫn đỏ ở bước 7?
2. Commit ở bước 8 chứa thay đổi nào của app.js?

<details>
<summary>Đáp án</summary>

1. Vì Git staging area lưu snapshot tại thời điểm `git add`. Thay đổi sau `git add` là thay đổi mới, chưa staged.
2. Chỉ chứa dòng `console.log('Updated');`, KHÔNG chứa `// comment` (vì bổ sung sau khi `git add`).

</details>

---

### Bài 3: .gitignore

**Mục tiêu**: Hiểu cách .gitignore hoạt động

```bash
# Bước 1: Tạo các file cần ignore
$ echo "SECRET_KEY=abc123" > .env
$ echo "debug info" > debug.log
$ mkdir node_modules
$ echo "{}" > node_modules/package.json

# Bước 2: Kiểm tra
$ git status
# → Tất cả đều hiện

# Bước 3: Tạo .gitignore
$ echo ".env" > .gitignore
$ echo "*.log" >> .gitignore
$ echo "node_modules/" >> .gitignore

# Bước 4: Kiểm tra lại
$ git status
# → .env, debug.log, node_modules/ BIẾN MẤT khỏi danh sách!
# → .gitignore xuất hiện (file mới)

# Bước 5: Commit
$ git add .
$ git commit -m "chore: add gitignore"

# Bước 6: Thử tạo thêm file .log
$ echo "new log" > error.log
$ git status
# → error.log không xuất hiện (đã bị ignore bởi *.log)
```

**Thử thách**: File `.env` đã bị commit trước khi tạo `.gitignore`. Làm thế nào để untrack nó?

<details>
<summary>Đáp án</summary>

```bash
$ git rm --cached .env
$ git commit -m "chore: untrack .env"
```

</details>

---

## Cấp độ 2: Trung bình (Intermediate)

### Bài 4: Branching & Merging

**Mục tiêu**: Tạo branch, merge, xử lý conflict

```bash
# === SETUP ===
$ mkdir branch-practice && cd branch-practice
$ git init
$ echo "Line 1: Original" > file.txt
$ git add . && git commit -m "init: first commit"

# === PHẦN A: Fast-forward Merge ===

# Tạo feature branch
$ git checkout -b feature/greeting

# Thêm nội dung
$ echo "Line 2: Hello from feature branch" >> file.txt
$ git add . && git commit -m "feat: add greeting"

# Merge vào main
$ git checkout main
$ git merge feature/greeting
# → Quan sát: Fast-forward merge

$ git log --oneline --graph --all

# === PHẦN B: 3-Way Merge ===

# Tạo branch mới
$ git checkout -b feature/footer

# Commit trên feature/footer
$ echo "Footer: Made with love" >> file.txt
$ git add . && git commit -m "feat: add footer"

# Quay lại main, commit thêm
$ git checkout main
$ echo "Line 3: Added on main" >> file.txt
$ git add . && git commit -m "feat: add content on main"

# Merge
$ git merge feature/footer
# → Quan sát: 3-way merge, tạo merge commit

$ git log --oneline --graph --all

# === PHẦN C: Merge Conflict ===

# Tạo 2 branch sửa cùng 1 dòng
$ git checkout -b branch-a
$ echo "Version A" > conflict.txt
$ git add . && git commit -m "branch-a: version A"

$ git checkout main
$ git checkout -b branch-b
$ echo "Version B" > conflict.txt
$ git add . && git commit -m "branch-b: version B"

# Merge branch-a vào branch-b
$ git merge branch-a
# → CONFLICT!

# Mở file conflict.txt → xem markers <<<, ===, >>>
$ cat conflict.txt

# Fix conflict (chọn version bạn muốn)
$ echo "Version A + B combined" > conflict.txt
$ git add conflict.txt
$ git commit -m "fix: resolve merge conflict"

$ git log --oneline --graph --all
```

**Tự kiểm tra:**
- [ ] Phân biệt được fast-forward vs 3-way merge trong log
- [ ] Xử lý được conflict

---

### Bài 5: Stash & Cherry-pick

**Mục tiêu**: Lưu tạm thay đổi, chọn commit cụ thể

```bash
# === STASH ===
$ mkdir stash-practice && cd stash-practice
$ git init
$ echo "main code" > app.js
$ git add . && git commit -m "init: project"

# Tạo feature branch và code dở
$ git checkout -b feature/wip
$ echo "work in progress..." >> app.js

# Đột nhiên cần fix bug trên main!
$ git stash push -m "WIP: feature đang làm dở"

# Chuyển sang main fix bug
$ git checkout main
$ echo "bugfix" >> app.js
$ git add . && git commit -m "hotfix: fix critical bug"

# Quay lại feature, lấy stash
$ git checkout feature/wip
$ git stash list
$ git stash pop
# → Code đang làm dở được khôi phục!

$ cat app.js    # Kiểm tra

# === CHERRY-PICK ===
# Tạo branch với nhiều commits
$ git checkout main
$ git checkout -b feature/utils
$ echo "function add() {}" > utils.js
$ git add . && git commit -m "feat: add function"
$ echo "function subtract() {}" >> utils.js
$ git add . && git commit -m "feat: subtract function"
$ echo "function multiply() {}" >> utils.js
$ git add . && git commit -m "feat: multiply function"

# Xem log
$ git log --oneline
# → Ghi nhớ hash của "feat: add function"

# Cherry-pick chỉ commit "add" vào main
$ git checkout main
$ git cherry-pick <hash-of-add-commit>

$ git log --oneline
# → main có commit "add" mà không có subtract, multiply
```

---

### Bài 6: Rebase & Interactive Rebase

**Mục tiêu**: Rebase branch, dọn dẹp commit history

```bash
$ mkdir rebase-practice && cd rebase-practice
$ git init
$ echo "v1" > app.js
$ git add . && git commit -m "init: v1"

# Tạo feature branch
$ git checkout -b feature/clean

# Tạo nhiều commits lộn xộn
$ echo "v2" > app.js && git add . && git commit -m "wip"
$ echo "v3" > app.js && git add . && git commit -m "fix typo"
$ echo "v4" > app.js && git add . && git commit -m "oops"
$ echo "v5" > app.js && git add . && git commit -m "feat: real feature"

# Xem log
$ git log --oneline
# → 4 commits lộn xộn + 1 init

# === INTERACTIVE REBASE: Dọn dẹp ===
$ git rebase -i HEAD~4

# Trong editor, sửa thành:
# pick <hash> feat: real feature    ← đưa lên đầu
# fixup <hash> wip                  ← gộp vào commit trên
# fixup <hash> fix typo             ← gộp
# fixup <hash> oops                 ← gộp

# Lưu & thoát

$ git log --oneline
# → Bây giờ chỉ còn 2 commits: init + feat: real feature
# → Lịch sử sạch đẹp!

# === REBASE lên main ===
# Quay lại main, thêm commit
$ git checkout main
$ echo "main update" >> main.txt
$ git add . && git commit -m "feat: main update"

# Rebase feature lên main
$ git checkout feature/clean
$ git rebase main

$ git log --oneline --graph --all
# → Feature branch "nằm trên" main, lịch sử tuyến tính
```

---

## Cấp độ 3: Nâng cao (Advanced)

### Bài 7: Remote Workflow (Cần tài khoản GitLab)

**Mục tiêu**: Push, pull, tạo Merge Request

```bash
# Bước 1: Tạo project mới trên GitLab
# → gitlab.com → New project → Create blank project
# → Tên: git-practice
# → Visibility: Private

# Bước 2: Liên kết local repo
$ cd git-practice    # (project từ bài 1)
$ git remote add origin git@gitlab.com:<username>/git-practice.git

# Bước 3: Push main
$ git push -u origin main

# Bước 4: Tạo feature branch
$ git checkout -b feature/about-page
$ echo "<h1>About</h1>" > about.html
$ git add . && git commit -m "feat: add about page"

# Bước 5: Push feature branch
$ git push -u origin feature/about-page

# Bước 6: Tạo Merge Request trên GitLab
# → Vào GitLab → Merge Requests → New
# → Source: feature/about-page
# → Target: main
# → Viết mô tả
# → Create merge request

# Bước 7: Merge trên GitLab (click Merge button)

# Bước 8: Cập nhật local
$ git checkout main
$ git pull origin main

# Bước 9: Dọn dẹp
$ git branch -d feature/about-page
$ git fetch --prune
```

---

### Bài 8: Xử lý sự cố thực tế

**Mục tiêu**: Thực hành khôi phục từ các sai lầm

```bash
$ mkdir rescue-practice && cd rescue-practice
$ git init

# Tạo lịch sử
$ echo "v1" > app.js && git add . && git commit -m "commit 1"
$ echo "v2" > app.js && git add . && git commit -m "commit 2"
$ echo "v3" > app.js && git add . && git commit -m "commit 3"
$ echo "v4" > app.js && git add . && git commit -m "commit 4: IMPORTANT"
$ echo "v5" > app.js && git add . && git commit -m "commit 5"

# === TÌNH HUỐNG 1: Reset nhầm ===

# "Oops!" - Reset mất 2 commit
$ git reset --hard HEAD~2
$ git log --oneline
# → commit 4 và 5 "biến mất"!

# Cứu lại bằng reflog:
$ git reflog
# → Tìm hash của "commit 5"
$ git reset --hard HEAD@{1}
$ git log --oneline
# → ✅ Đã quay lại!

# === TÌNH HUỐNG 2: Commit nhầm branch ===

$ echo "feature code" > feature.js
$ git add . && git commit -m "feat: new feature"
# Oops! Commit này nên ở feature branch, không phải main!

# Sửa:
$ git log --oneline -1    # Ghi hash
$ git checkout -b feature/oops
# → Branch mới đã có commit
$ git checkout main
$ git reset --hard HEAD~1
# → main quay lại trước commit nhầm

# === TÌNH HUỐNG 3: Khôi phục file đã xóa ===

# Xóa file
$ git rm app.js
$ git commit -m "chore: remove app.js"

# Cần lại file app.js!
$ git log --all -- app.js     # Tìm commit cuối chứa file
# → Ghi hash
$ git checkout <hash>^ -- app.js
$ git add . && git commit -m "feat: restore app.js"
# → ✅ File đã quay lại!
```

---

### Bài 9: Git Bisect - Tìm commit gây bug

**Mục tiêu**: Dùng bisect để tìm commit gây lỗi

```bash
$ mkdir bisect-practice && cd bisect-practice
$ git init

# Tạo 10 commits, commit thứ 6 gây bug
$ echo "function calc(x) { return x * 2; }" > calc.js
$ git add . && git commit -m "commit 1: init"

$ echo "function calc(x) { return x * 2; }" > calc.js
$ git add . && git commit -m "commit 2: refactor"

$ echo "function calc(x) { return x * 2; }" > calc.js
$ git add . && git commit -m "commit 3: style"

$ echo "function calc(x) { return x * 2; }" > calc.js
$ git add . && git commit -m "commit 4: docs"

$ echo "function calc(x) { return x * 2; }" > calc.js
$ git add . && git commit -m "commit 5: test"

# ⬇️ Commit này gây bug!
$ echo "function calc(x) { return x * 3; }" > calc.js
$ git add . && git commit -m "commit 6: optimize"

$ echo "function calc(x) { return x * 3; }" > calc.js
$ echo "// added logging" >> calc.js
$ git add . && git commit -m "commit 7: logging"

$ echo "function calc(x) { return x * 3; }" > calc.js
$ echo "// added logging" >> calc.js
$ echo "// more stuff" >> calc.js
$ git add . && git commit -m "commit 8: more"

# BÂY GIỜ: Bạn phát hiện calc(5) trả về 15 thay vì 10!
# Dùng bisect để tìm commit nào gây ra:

$ git bisect start
$ git bisect bad                    # Hiện tại có bug
$ git bisect good HEAD~7            # commit 1 không bug

# Git checkout commit giữa, bạn kiểm tra:
$ cat calc.js
# → Nếu thấy "x * 3" → bad, nếu "x * 2" → good

$ git bisect bad    # hoặc good (tùy kết quả kiểm tra)
# Lặp lại...

# Cuối cùng:
# → Git tìm ra "commit 6: optimize" là commit gây bug!

$ git bisect reset   # Quay lại HEAD
```

---

### Bài 10: Workflow hoàn chỉnh (Capstone)

**Mục tiêu**: Mô phỏng workflow thực tế trong team

```bash
# === SETUP: Tạo "central repo" ===
$ mkdir team-project && cd team-project
$ git init
$ echo "# Team Project" > README.md
$ echo "v1.0" > version.txt
$ git add . && git commit -m "init: team project v1.0"

# === DEVELOPER A: Làm feature login ===

# Cập nhật main
$ git checkout main

# Tạo feature branch
$ git checkout -b feature/login

# Code...
$ echo "<form>Login</form>" > login.html
$ git add . && git commit -m "feat(auth): add login form"

$ echo "function validate() {}" > auth.js
$ git add . && git commit -m "feat(auth): add validation"

# === DEVELOPER B: Làm feature dashboard (trên main) ===
$ git checkout main
$ git checkout -b feature/dashboard

$ echo "<div>Dashboard</div>" > dashboard.html
$ git add . && git commit -m "feat(dashboard): add dashboard page"

# === CẬP NHẬT: Main có hotfix ===
$ git checkout main
$ echo "v1.0.1" > version.txt
$ git add . && git commit -m "hotfix: bump version"

# === DEVELOPER A: Cập nhật main vào feature ===
$ git checkout feature/login
$ git rebase main
# → Feature login giờ nằm trên hotfix

# === Dọn dẹp commits trước MR ===
$ git rebase -i HEAD~2
# Giữ nguyên hoặc squash tùy ý

# === MERGE: Giả lập merge vào main ===
$ git checkout main
$ git merge --no-ff feature/login -m "Merge feature/login into main"

# === MERGE: Dashboard ===
$ git checkout feature/dashboard
$ git rebase main    # Cập nhật
$ git checkout main
$ git merge --no-ff feature/dashboard -m "Merge feature/dashboard into main"

# === XEM KẾT QUẢ ===
$ git log --oneline --graph --all

# === DỌN DẸP ===
$ git branch -d feature/login
$ git branch -d feature/dashboard

# === TAG RELEASE ===
$ echo "v1.1.0" > version.txt
$ git add . && git commit -m "release: v1.1.0"
$ git tag -a v1.1.0 -m "Release v1.1.0 - Login & Dashboard"

$ git log --oneline --graph --all --decorate
```

**Kết quả mong đợi**: Log hiện graph với merge commits, branches, và tag.

---

## Checklist tổng kết

### Sau khi hoàn thành tất cả bài tập, bạn nên:

**Cơ bản:**
- [ ] Tự tin dùng `init`, `add`, `commit`, `status`, `log`
- [ ] Hiểu Staging Area và 3 khu vực của Git
- [ ] Biết tạo và dùng `.gitignore`

**Trung bình:**
- [ ] Tạo, chuyển, merge, xóa branch
- [ ] Xử lý merge conflict
- [ ] Dùng `stash` và `cherry-pick`
- [ ] Dùng `rebase` và interactive rebase

**Nâng cao:**
- [ ] Làm việc với remote (push, pull, fetch)
- [ ] Tạo Merge Request trên GitLab
- [ ] Khôi phục từ sai lầm (`reflog`, `reset`, `revert`)
- [ ] Dùng `bisect` tìm bug
- [ ] Áp dụng workflow thực tế

---

## Tài nguyên học thêm

| Nguồn | Link | Mô tả |
|-------|------|--------|
| Pro Git Book | https://git-scm.com/book/vi | Sách miễn phí (có tiếng Việt) |
| GitLab Docs | https://docs.gitlab.com | Tài liệu chính thức GitLab |
| Learn Git Branching | https://learngitbranching.js.org | Game học Git trực quan |
| Oh Shit, Git!? | https://ohshitgit.com | Xử lý sự cố Git |
| Git Cheatsheet | https://education.github.com/git-cheat-sheet-education.pdf | PDF cheatsheet |
| Conventional Commits | https://www.conventionalcommits.org | Quy ước commit message |

---

← [08 - Cheatsheet](08-cheatsheet.md) | **Quay lại**: [README](../README.md)
