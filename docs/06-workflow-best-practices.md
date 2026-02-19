# 06 - Workflow & Best Practices

## 1. Các Git Workflow phổ biến

### 1.1 GitLab Flow (Khuyến nghị cho GitLab)

```
                    Production
                        ↑ merge
                    Pre-production (staging)
                        ↑ merge
┌──────────┐        ┌──────┐
│feature/A │───────>│ main │
└──────────┘  MR    │      │
┌──────────┐        │      │
│feature/B │───────>│      │
└──────────┘  MR    └──────┘
┌──────────┐           ↑
│hotfix/C  │───────────┘
└──────────┘    MR
```

**Quy tắc:**
1. `main` là branch chính, luôn ở trạng thái stable
2. Mỗi feature/fix tạo branch riêng từ `main`
3. Merge vào `main` qua Merge Request
4. Deploy từ `main` → staging → production

```bash
# Workflow hàng ngày:
$ git checkout main
$ git pull origin main
$ git checkout -b feature/JIRA-456-new-dashboard

# Code...
$ git add .
$ git commit -m "feat(dashboard): add chart component"
$ git push -u origin feature/JIRA-456-new-dashboard
# → Tạo MR trên GitLab
```

### 1.2 Git Flow (Phổ biến nhất)

```
         hotfix/fix
          /        \
─────●───●──────────●────────────── main (production)
     │                    ↑
     │              release/1.0
     │              /          \
─────●───●────●───●─────────────●── develop
         \   /         \       /
      feature/A     feature/B
```

**Các branch:**

| Branch | Mục đích | Tạo từ | Merge vào |
|--------|---------|--------|-----------|
| `main` | Production code | - | - |
| `develop` | Tích hợp features | `main` | `main` (qua release) |
| `feature/*` | Tính năng mới | `develop` | `develop` |
| `release/*` | Chuẩn bị release | `develop` | `main` + `develop` |
| `hotfix/*` | Fix bug production | `main` | `main` + `develop` |

```bash
# Feature workflow:
$ git checkout develop
$ git checkout -b feature/user-profile
# ... code ...
$ git push origin feature/user-profile
# → MR vào develop

# Release workflow:
$ git checkout develop
$ git checkout -b release/1.0
# ... fix bugs, bump version ...
# → MR vào main VÀ develop

# Hotfix workflow:
$ git checkout main
$ git checkout -b hotfix/login-crash
# ... fix ...
# → MR vào main VÀ develop
```

### 1.3 Trunk-Based Development (Đơn giản)

```
         feature/A (ngắn, < 1 ngày)
          /     \
─────●───●───●───●────●────── main (trunk)
              \  /
           feature/B (ngắn)
```

**Quy tắc:**
- Chỉ có 1 branch chính: `main`
- Feature branch rất ngắn (< 1-2 ngày)
- Merge thường xuyên vào `main`
- Dùng feature flags để ẩn/hiện tính năng

```bash
# Tạo branch ngắn
$ git checkout -b feat/add-button
# ... code nhanh ...
$ git commit -m "feat: add submit button"
$ git push origin feat/add-button
# → MR ngay, merge nhanh
```

### So sánh 3 Workflow

| | GitLab Flow | Git Flow | Trunk-Based |
|---|---|---|---|
| Độ phức tạp | Trung bình | Cao | Thấp |
| Số branch | Vừa phải | Nhiều | Ít |
| Phù hợp | Team vừa | Team lớn, release cycle | Team nhỏ, CI/CD mạnh |
| Release | Liên tục | Định kỳ | Liên tục |

---

## 2. Quy ước đặt tên Branch

### 2.1 Format chuẩn

```
<type>/<ticket-id>-<short-description>
```

### 2.2 Các type phổ biến

| Type | Khi nào | Ví dụ |
|------|---------|-------|
| `feature/` | Tính năng mới | `feature/JIRA-123-user-login` |
| `fix/` hoặc `bugfix/` | Sửa bug | `fix/JIRA-456-login-crash` |
| `hotfix/` | Fix bug khẩn cấp | `hotfix/JIRA-789-security-patch` |
| `refactor/` | Cải thiện code | `refactor/optimize-queries` |
| `chore/` | Task không phải code | `chore/update-dependencies` |
| `docs/` | Tài liệu | `docs/update-api-guide` |
| `test/` | Thêm/sửa test | `test/add-login-tests` |
| `release/` | Chuẩn bị release | `release/v2.1.0` |

### 2.3 Quy tắc đặt tên

```bash
# ✅ TỐT
feature/PROJ-123-add-user-authentication
fix/PROJ-456-fix-email-validation
hotfix/critical-security-vulnerability
refactor/simplify-payment-flow

# ❌ KHÔNG TỐT
my-branch                    # Không rõ mục đích
feature/login-page-redesign-with-new-ui-and-better-ux   # Quá dài
Feature/Login               # Viết hoa, có space
fix bug                     # Có space
```

---

## 3. Quy ước Commit Message (Conventional Commits)

### 3.1 Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 3.2 Chi tiết

```bash
# === TYPE (bắt buộc) ===
feat:     # Tính năng mới
fix:      # Sửa bug
docs:     # Thay đổi documentation
style:    # Format code (không thay đổi logic)
refactor: # Refactor code (không fix bug, không thêm feature)
perf:     # Cải thiện performance
test:     # Thêm hoặc sửa tests
build:    # Thay đổi build system (webpack, npm, etc.)
ci:       # Thay đổi CI/CD config
chore:    # Thay đổi khác (không ảnh hưởng src/test)
revert:   # Revert commit trước

# === SCOPE (tùy chọn) ===
# Phạm vi thay đổi
feat(auth):     # Liên quan đến authentication
fix(api):       # Liên quan đến API
docs(readme):   # Liên quan đến README

# === SUBJECT (bắt buộc) ===
# Mô tả ngắn gọn, viết thường, không dấu chấm cuối
```

### 3.3 Ví dụ commit message tốt

```bash
# Đơn giản
$ git commit -m "feat: thêm chức năng đăng nhập bằng Google"
$ git commit -m "fix: sửa lỗi crash khi email rỗng"
$ git commit -m "docs: cập nhật hướng dẫn cài đặt"
$ git commit -m "refactor: tách UserService thành module riêng"
$ git commit -m "test: thêm unit test cho PaymentService"
$ git commit -m "chore: update dependencies"

# Với scope
$ git commit -m "feat(auth): thêm OAuth2 login"
$ git commit -m "fix(cart): sửa tính sai tổng giá"
$ git commit -m "perf(api): cache response giảm 50% load time"

# Với body (nhiều dòng)
$ git commit -m "feat(auth): thêm chức năng 2FA

Thêm xác thực 2 yếu tố bằng TOTP:
- Tích hợp Google Authenticator
- Thêm trang setup 2FA trong Settings
- Thêm backup codes

Closes #234"

# Breaking change
$ git commit -m "feat(api)!: đổi response format

BREAKING CHANGE: API response không còn wrap trong 'data' field.
Trước: { data: { users: [] } }
Sau:   { users: [] }

Migration guide: xem docs/migration-v3.md"
```

### 3.4 So sánh commit message tốt vs xấu

```bash
# ❌ XẤU
"fix bug"                    # Fix bug gì?
"update"                     # Update gì?
"changes"                    # ???
"WIP"                        # Không nên push WIP
"asdfg"                      # ...
"Fix the thing that was broken in the last commit" # Quá dài, không theo format

# ✅ TỐT
"fix(auth): prevent crash when token is expired"
"feat(cart): add quantity selector to cart items"
"refactor(user): extract validation logic to utils"
"docs: add API endpoint documentation for v2"
"chore: upgrade React from 17 to 18"
```

---

## 4. Code Review Best Practices

### 4.1 Cho người tạo MR (Author)

```
✅ NÊN:
1. MR nhỏ, tập trung 1 mục đích (< 400 dòng thay đổi)
2. Self-review trước khi assign reviewer
3. Viết mô tả MR rõ ràng
4. Thêm screenshots cho UI changes
5. Đảm bảo CI/CD pass trước khi request review
6. Reply mọi comment của reviewer
7. Squash WIP commits trước khi merge

❌ KHÔNG NÊN:
1. MR khổng lồ (> 1000 dòng)
2. Mix nhiều features trong 1 MR
3. Force push sau khi đã có review comments
4. Merge mà chưa address hết comments
```

### 4.2 Cho người review (Reviewer)

```
✅ NÊN:
1. Review trong 24h (hoặc sớm hơn)
2. Góp ý constructive, có solution
3. Phân biệt: "Must fix" vs "Nice to have" vs "Nit:"
4. Approve khi satisfied, đừng block vì nitpick
5. Khen ngợi code tốt

❌ KHÔNG NÊN:
1. Để MR pending quá lâu
2. Chỉ comment "sửa đi" mà không nói sửa gì
3. Review style trong khi có linter
4. Quá khắt khe với approach khác mình
```

### 4.3 Comment Conventions

```
# Phân loại comment:

🔴 [MUST] - Bắt buộc sửa trước khi merge
"[MUST] SQL injection vulnerability ở dòng này. 
Cần dùng parameterized query."

🟡 [SHOULD] - Nên sửa
"[SHOULD] Nên extract logic này ra function riêng 
để dễ test hơn."

🟢 [NIT] - Không bắt buộc, góp ý nhỏ
"[NIT] Có thể dùng destructuring ở đây cho gọn."

💬 [QUESTION] - Hỏi để hiểu
"[QUESTION] Tại sao chọn approach này thay vì dùng 
built-in method?"

👍 [PRAISE] - Khen
"[PRAISE] Clean code! Cách handle error ở đây rất tốt."
```

---

## 5. Git Hooks (Tự động hóa)

### 5.1 Git Hooks là gì?

Scripts tự động chạy tại các thời điểm cụ thể:

```
pre-commit    → Trước khi commit (lint, format)
commit-msg    → Kiểm tra commit message
pre-push      → Trước khi push (run tests)
post-merge    → Sau khi merge (install dependencies)
```

### 5.2 Dùng Husky (Node.js projects)

```bash
# Cài đặt Husky
$ npm install --save-dev husky

# Khởi tạo
$ npx husky init

# Tạo pre-commit hook (lint trước khi commit)
$ echo "npm run lint" > .husky/pre-commit

# Tạo commit-msg hook (kiểm tra format commit message)
$ echo 'npx commitlint --edit "$1"' > .husky/commit-msg

# Cài commitlint
$ npm install --save-dev @commitlint/cli @commitlint/config-conventional
```

File `commitlint.config.js`:
```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', [
      'feat', 'fix', 'docs', 'style', 'refactor',
      'perf', 'test', 'build', 'ci', 'chore', 'revert'
    ]],
    'subject-max-length': [2, 'always', 72],
  },
};
```

### 5.3 Dùng lint-staged (Chỉ lint file đã staged)

```bash
$ npm install --save-dev lint-staged
```

File `package.json`:
```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{css,scss}": [
      "prettier --write"
    ],
    "*.md": [
      "prettier --write"
    ]
  }
}
```

Sửa `.husky/pre-commit`:
```bash
npx lint-staged
```

---

## 6. Semantic Versioning (SemVer)

### 6.1 Format: MAJOR.MINOR.PATCH

```
v2.4.1
│ │ │
│ │ └── PATCH: Fix bug, không thay đổi API
│ └──── MINOR: Thêm tính năng, backward compatible
└────── MAJOR: Breaking changes, không backward compatible

Ví dụ:
v1.0.0 → v1.0.1  (fix bug)
v1.0.1 → v1.1.0  (thêm feature)
v1.1.0 → v2.0.0  (breaking change)
```

### 6.2 Tạo Release trên GitLab

```bash
# Tạo tag
$ git tag -a v1.2.0 -m "Release v1.2.0 - Thêm dashboard"
$ git push origin v1.2.0

# Trên GitLab:
# Repository → Tags → v1.2.0 → Create release
# Hoặc: Deployments → Releases → New release
```

---

## 7. Quy trình làm việc thực tế (Real-world Workflow)

### 7.1 Sprint Workflow

```bash
# ─── SPRINT BẮT ĐẦU ───

# 1. Nhận task từ Board (VD: PROJ-123)
# 2. Tạo branch
$ git checkout main
$ git pull origin main
$ git checkout -b feature/PROJ-123-user-settings

# ─── HÀNG NGÀY ───

# 3. Code & commit thường xuyên
$ git add .
$ git commit -m "feat(settings): add notification preferences UI"

$ git add .
$ git commit -m "feat(settings): add API integration"

$ git add .
$ git commit -m "test(settings): add unit tests"

# 4. Push hàng ngày (backup)
$ git push origin feature/PROJ-123-user-settings

# 5. Cập nhật từ main (hàng ngày hoặc trước MR)
$ git fetch origin
$ git rebase origin/main
# Fix conflict nếu có...
$ git push --force-with-lease

# ─── HOÀN THÀNH TASK ───

# 6. Dọn dẹp commits (tùy team)
$ git rebase -i HEAD~5   # Squash WIP commits
$ git push --force-with-lease

# 7. Tạo MR trên GitLab
#    - Title: "feat(settings): add user notification settings"
#    - Description: mô tả, screenshots
#    - Assign reviewer
#    - Label: feature, frontend

# 8. Address review feedback
$ git add .
$ git commit -m "fix(settings): address review - add input validation"
$ git push

# 9. MR được approve → Merge (squash)

# 10. Dọn dẹp
$ git checkout main
$ git pull origin main
$ git branch -d feature/PROJ-123-user-settings
$ git fetch --prune
```

### 7.2 Flow Chart

```
  Nhận task
     │
     ▼
  Tạo branch ← git checkout -b feature/PROJ-XXX
     │
     ▼
  Code + Commit ← git commit -m "..."  (loop)
     │
     ▼
  Push ← git push origin feature/PROJ-XXX
     │
     ▼
  Rebase main ← git rebase origin/main
     │
     ├─ Conflict? → Fix → git add → git rebase --continue
     │
     ▼
  Tạo MR trên GitLab
     │
     ▼
  Code Review ←──── Feedback? → Sửa → Push → (loop)
     │
     ▼
  CI/CD Pass?
     │
     ├─ Fail? → Fix → Push → (loop)
     │
     ▼
  Merge ✅
     │
     ▼
  Xóa branch + Pull main
```

---

## 8. Best Practices tổng hợp

### 8.1 Commit

```
✅ Commit thường xuyên (nhỏ, atomic)
✅ Mỗi commit = 1 thay đổi logic
✅ Viết commit message theo quy ước
✅ Không commit file generated (dist/, build/)
✅ Không commit secrets (passwords, API keys)
✅ Review `git diff` trước khi commit

❌ Commit lớn chứa nhiều thay đổi
❌ Commit message "fix", "update", "WIP"
❌ Commit node_modules/
❌ Commit .env files
```

### 8.2 Branch

```
✅ Tạo branch cho mỗi task
✅ Đặt tên branch có ý nghĩa
✅ Giữ branch up-to-date với main
✅ Xóa branch sau khi merge
✅ Branch ngắn hạn (< 1 tuần)

❌ Code trực tiếp trên main
❌ Branch sống quá lâu (> 2 tuần)
❌ Tên branch vô nghĩa
```

### 8.3 Merge Request

```
✅ MR nhỏ, dễ review (< 400 dòng)
✅ Mô tả rõ ràng thay đổi
✅ Self-review trước
✅ Đảm bảo CI pass
✅ Reply tất cả review comments

❌ MR khổng lồ
❌ Merge mà chưa review
❌ Skip CI/CD
```

### 8.4 An toàn

```
✅ Dùng --force-with-lease thay vì --force
✅ Pull trước khi push
✅ Backup bằng stash trước khi thao tác nguy hiểm
✅ Dùng revert cho shared branches
✅ Test trên staging trước production

❌ Force push lên main/develop
❌ Reset --hard trên shared branches
❌ Deploy thẳng lên production
```

---

← [05 - GitLab](05-gitlab.md) | **Tiếp theo**: [07 - Xử lý sự cố](07-xu-ly-su-co.md) →
