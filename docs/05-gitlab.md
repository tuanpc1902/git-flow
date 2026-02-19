# 05 - GitLab: Hướng dẫn chi tiết

## 1. Tổng quan GitLab

### GitLab là gì?

GitLab là **nền tảng DevOps hoàn chỉnh** cung cấp:

```
┌──────────────────────────────────────────────────────┐
│                     GitLab                           │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │   Plan   │  │  Create  │  │  Verify  │           │
│  │ Issues   │  │ Git Repo │  │  CI/CD   │           │
│  │ Boards   │  │ MR/CR    │  │ Testing  │           │
│  └──────────┘  └──────────┘  └──────────┘           │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │ Package  │  │ Release  │  │ Monitor  │           │
│  │ Registry │  │ Deploy   │  │ Logging  │           │
│  └──────────┘  └──────────┘  └──────────┘           │
└──────────────────────────────────────────────────────┘
```

### GitLab vs GitHub

| Tính năng | GitLab | GitHub |
|-----------|--------|--------|
| CI/CD tích hợp | ✅ Có sẵn | Cần GitHub Actions |
| Self-hosted | ✅ Miễn phí (CE) | Enterprise (trả phí) |
| Container Registry | ✅ Có sẵn | Có (GitHub Packages) |
| Issue Boards | ✅ Có sẵn | Projects |
| Thuật ngữ | Merge Request | Pull Request |
| Wiki | ✅ Có sẵn | ✅ Có sẵn |

---

## 2. Merge Request (MR) - Kiến thức quan trọng nhất

### 2.1 Merge Request là gì?

MR là yêu cầu **gộp code từ branch này vào branch khác**, kèm theo:
- Mô tả thay đổi
- Code review từ đồng nghiệp
- Kiểm tra tự động (CI/CD)
- Thảo luận & feedback

### 2.2 Tạo Merge Request

#### Cách 1: Từ command line
```bash
# Push branch lên remote
$ git push -u origin feature/login

# Git trả về link tạo MR
> remote: To create a merge request for feature/login, visit:
> remote:   https://gitlab.com/team/project/-/merge_requests/new?merge_request%5Bsource_branch%5D=feature/login
```

#### Cách 2: Trên GitLab Web

```
1. Vào Repository → Merge Requests → New merge request
2. Chọn:
   - Source branch: feature/login
   - Target branch: main (hoặc develop)
3. Click "Compare branches and continue"
4. Điền thông tin:
```

### 2.3 Cấu trúc Merge Request tốt

```markdown
## Title
feat: Thêm chức năng đăng nhập bằng email

## Description

### Mô tả
Thêm trang đăng nhập cho người dùng, hỗ trợ:
- Đăng nhập bằng email/password
- Remember me
- Forgot password link

### Thay đổi chính
- Thêm `LoginForm` component
- Thêm `authService` cho API calls
- Thêm validation cho email/password
- Thêm unit tests

### Screenshots (nếu có UI)
| Trước | Sau |
|-------|-----|
| [ảnh] | [ảnh] |

### Checklist
- [x] Code follows project guidelines
- [x] Tests passed
- [x] No console errors
- [ ] Documentation updated

### Related Issues
Closes #123
Related to #456
```

### 2.4 Các tùy chọn Merge Request

```
┌─ Merge Request Settings ──────────────────────────┐
│                                                     │
│  Assignee:       Tuan Pham (người thực hiện)        │
│  Reviewers:      Minh Tran, Hoa Nguyen              │
│  Labels:         feature, frontend                  │
│  Milestone:      Sprint 5                           │
│                                                      │
│  ☑ Delete source branch when MR is accepted         │
│  ☑ Squash commits when MR is accepted               │
│  ☐ Allow commits from members who can merge         │
│                                                     │
│  Merge options:                                     │
│  ● Merge commit          (tạo merge commit)         │
│  ○ Merge commit with     (merge + squash commits)   │
│    semi-linear history                              │
│  ○ Fast-forward merge    (không tạo merge commit)   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 2.5 Merge Strategies trên GitLab

| Strategy | Mô tả | Khi nào dùng |
|----------|--------|-------------|
| **Merge commit** | Tạo merge commit, giữ tất cả commits | Muốn giữ lịch sử đầy đủ |
| **Squash and merge** | Gộp tất cả commits thành 1 | Feature có nhiều WIP commits |
| **Fast-forward** | Không tạo merge commit | Lịch sử tuyến tính |

### 2.6 Code Review trong MR

```
Khi review MR của đồng nghiệp:

1. Xem tab "Changes" → đọc code thay đổi
2. Click vào dòng code để comment:

   ┌─ Comment ─────────────────────────────────────┐
   │ src/login.js:25                                │
   │                                                │
   │ 💬 "Nên validate email format ở client-side   │  
   │    trước khi gọi API để giảm request không    │
   │    cần thiết"                                  │
   │                                                │
   │ [Start a review] [Add comment now]             │
   └────────────────────────────────────────────────┘

3. "Start a review" = gom comments, submit cùng lúc (khuyến nghị)
4. "Add comment now" = gửi comment ngay lập tức

5. Khi review xong, Submit review với:
   - ✅ Approve: Đồng ý merge
   - 💬 Comment: Góp ý, không approve/reject
   - ❌ Request changes: Yêu cầu sửa
```

### 2.7 Xử lý feedback từ Code Review

```bash
# Đồng nghiệp yêu cầu sửa code trong MR

# 1. Đang ở branch feature/login
$ git checkout feature/login

# 2. Sửa code theo feedback

# 3. Commit & push
$ git add .
$ git commit -m "fix: address review feedback - validate email client-side"
$ git push origin feature/login

# → MR tự động cập nhật với commit mới
# → Reply comment trên GitLab: "Done, đã sửa ✅"
```

---

## 3. GitLab CI/CD

### 3.1 CI/CD là gì?

```
CI (Continuous Integration):  Tự động build + test mỗi khi push code
CD (Continuous Delivery):     Tự động deploy sau khi CI pass
CD (Continuous Deployment):   Tự động deploy lên production

Push code → Build → Test → Deploy (staging) → Deploy (production)
    │         │       │         │                    │
    │         CI──────┘         CD───────────────────┘
```

### 3.2 File .gitlab-ci.yml

GitLab CI/CD được cấu hình bằng file `.gitlab-ci.yml` ở **root của project**:

```yaml
# .gitlab-ci.yml - Ví dụ cơ bản

# Định nghĩa các stages (chạy tuần tự)
stages:
  - build
  - test
  - deploy

# Variables (biến môi trường)
variables:
  NODE_VERSION: "18"

# Job: Build
build:
  stage: build
  image: node:18                    # Docker image để chạy
  script:
    - npm ci                        # Cài dependencies
    - npm run build                 # Build project
  artifacts:                        # Lưu kết quả build
    paths:
      - dist/
    expire_in: 1 hour

# Job: Test
test:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm run test                  # Chạy tests
  coverage: '/Statements\s*:\s*(\d+\.?\d*)%/'   # Parse coverage

# Job: Lint
lint:
  stage: test                       # Chạy song song với test
  image: node:18
  script:
    - npm ci
    - npm run lint

# Job: Deploy Staging
deploy_staging:
  stage: deploy
  script:
    - echo "Deploying to staging..."
    - ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop                       # Chỉ chạy trên branch develop

# Job: Deploy Production
deploy_production:
  stage: deploy
  script:
    - echo "Deploying to production..."
    - ./deploy.sh production
  environment:
    name: production
    url: https://example.com
  only:
    - main                          # Chỉ chạy trên branch main
  when: manual                      # Phải click manual để deploy
```

### 3.3 Ví dụ CI/CD cho các loại project

#### Node.js (Frontend/Backend)
```yaml
stages:
  - install
  - quality
  - build
  - deploy

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/

install:
  stage: install
  image: node:18
  script:
    - npm ci

lint:
  stage: quality
  image: node:18
  script:
    - npm run lint

test:
  stage: quality
  image: node:18
  script:
    - npm run test -- --coverage
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  image: node:18
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
```

#### Docker Build & Push
```yaml
build_docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main
```

### 3.4 Các biến CI/CD có sẵn

| Biến | Mô tả | Ví dụ |
|------|--------|-------|
| `CI_COMMIT_SHA` | Hash đầy đủ | `abc123def456...` |
| `CI_COMMIT_SHORT_SHA` | Hash ngắn | `abc123d` |
| `CI_COMMIT_BRANCH` | Tên branch | `feature/login` |
| `CI_COMMIT_REF_NAME` | Branch hoặc tag | `main` |
| `CI_PIPELINE_ID` | ID pipeline | `12345` |
| `CI_PROJECT_NAME` | Tên project | `my-project` |
| `CI_MERGE_REQUEST_IID` | ID của MR | `42` |
| `CI_REGISTRY` | URL registry | `registry.gitlab.com` |
| `CI_REGISTRY_IMAGE` | Image path | `registry.gitlab.com/team/project` |

### 3.5 Xem CI/CD Pipeline trên GitLab

```
Vào: Project → CI/CD → Pipelines

┌─ Pipeline #1234 ─────────────────────────────────────┐
│  Status: ✅ Passed    Duration: 5 min 23 sec          │
│  Branch: feature/login   Commit: abc1234              │
│                                                       │
│  ┌────────┐    ┌────────┐    ┌──────────┐             │
│  │ Build  │ →  │  Test  │ →  │ Deploy   │             │
│  │  ✅    │    │  ✅    │    │   ⏸️     │             │
│  │ 1m 20s │    │ 3m 45s │    │ manual   │             │
│  └────────┘    └────────┘    └──────────┘             │
└───────────────────────────────────────────────────────┘
```

---

## 4. Protected Branches

### 4.1 Tại sao cần Protected Branch?

Ngăn chặn:
- Push trực tiếp vào `main` (phải qua MR)
- Force push (ghi đè lịch sử)
- Xóa branch quan trọng

### 4.2 Cấu hình Protected Branch

```
Vào: Settings → Repository → Protected branches

┌─ Protected Branches ──────────────────────────────┐
│                                                     │
│  Branch: main                                       │
│                                                     │
│  Allowed to merge:  Maintainers                     │
│  Allowed to push:   No one  (phải qua MR)          │
│  Allowed to force push: ❌                          │
│  Code owner approval: ✅                            │
│                                                     │
│  Branch: develop                                    │
│  Allowed to merge:  Developers + Maintainers        │
│  Allowed to push:   No one                          │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 5. GitLab Issues & Boards

### 5.1 Tạo Issue

```markdown
# Title: [BUG] Login form không validate email

## Mô tả
Khi nhập email sai format (VD: "abc"), form vẫn submit được.

## Steps to reproduce
1. Vào trang /login
2. Nhập email: "abc" 
3. Nhập password: "123456"
4. Click "Login"

## Expected behavior
Hiển thị lỗi "Email không hợp lệ"

## Actual behavior
Form submit bình thường, API trả về 400

## Environment
- Browser: Chrome 120
- OS: Windows 11

## Labels: bug, frontend, priority::high
## Assignee: @tuan.pham
## Milestone: Sprint 5
```

### 5.2 Liên kết Issue với MR

```bash
# Trong commit message hoặc MR description:

# Đóng issue khi MR merge
Closes #123
Fixes #123
Resolves #123

# Liên kết nhưng không đóng
Related to #123
See #123

# Đóng nhiều issues
Closes #123, #456, #789
```

### 5.3 Issue Board (Quản lý công việc)

```
┌─ Board: Sprint 5 ────────────────────────────────────────┐
│                                                           │
│  Open          In Progress      Review         Done       │
│  ┌─────────┐  ┌─────────────┐  ┌───────────┐  ┌──────┐  │
│  │ #130    │  │ #123        │  │ #120      │  │ #115 │  │
│  │ Fix CSS │  │ Login form  │  │ Dashboard │  │ API  │  │
│  │ @hoa    │  │ @tuan       │  │ @minh     │  │ @tuan│  │
│  ├─────────┤  └─────────────┘  └───────────┘  ├──────┤  │
│  │ #131    │                                   │ #118 │  │
│  │ Add API │                                   │ Menu │  │
│  │         │                                   │ @hoa │  │
│  └─────────┘                                   └──────┘  │
└───────────────────────────────────────────────────────────┘
```

---

## 6. GitLab Container Registry

```bash
# Login vào GitLab Container Registry
$ docker login registry.gitlab.com

# Build và push image
$ docker build -t registry.gitlab.com/team/project:latest .
$ docker push registry.gitlab.com/team/project:latest

# Pull image
$ docker pull registry.gitlab.com/team/project:latest
```

---

## 7. GitLab Wiki

Mỗi project có Wiki tích hợp:

```
Vào: Project → Wiki → Create page

Hữu ích cho:
- Tài liệu hướng dẫn setup
- API documentation
- Architecture decisions
- Team conventions
```

---

## 8. GitLab Snippets

Chia sẻ đoạn code nhỏ (như Gist của GitHub):

```
Vào: User menu → Snippets → New snippet

Dùng khi:
- Chia sẻ script nhỏ
- Lưu cấu hình mẫu
- Chia sẻ code snippet cho team
```

---

## 9. Các tính năng GitLab hữu ích khác

### 9.1 Merge Request Templates

Tạo file `.gitlab/merge_request_templates/Default.md`:

```markdown
## Mô tả
<!-- Mô tả ngắn gọn thay đổi -->

## Loại thay đổi
- [ ] Bug fix
- [ ] Feature mới
- [ ] Refactoring
- [ ] Documentation

## Checklist
- [ ] Code đã được self-review
- [ ] Tests đã pass
- [ ] Documentation đã cập nhật
- [ ] No breaking changes

## Screenshots (nếu có)

## Related Issues
Closes #
```

### 9.2 Issue Templates

Tạo file `.gitlab/issue_templates/Bug.md`:

```markdown
## Mô tả bug

## Steps to reproduce
1. 
2. 
3. 

## Expected behavior

## Actual behavior

## Screenshots

## Environment
- Browser: 
- OS: 
- Version: 

/label ~bug ~priority::medium
```

### 9.3 CODEOWNERS

Tạo file `CODEOWNERS` ở root:

```
# Syntax: path     @user hoặc @group

# Tất cả file
*                   @team-lead

# Frontend
src/components/     @frontend-team
src/styles/         @frontend-team

# Backend  
src/api/            @backend-team
src/models/         @backend-team

# DevOps
.gitlab-ci.yml      @devops-team
Dockerfile          @devops-team
```

### 9.4 Deploy Tokens

```
Settings → Repository → Deploy tokens

Dùng để:
- CI/CD pull images
- External services access repo
- Read-only access cho deployment
```

---

## 10. Tóm tắt GitLab

| Tính năng | Mô tả | Truy cập |
|-----------|--------|----------|
| Merge Request | Yêu cầu gộp code | Repository → MR |
| CI/CD | Tự động build/test/deploy | CI/CD → Pipelines |
| Protected Branch | Bảo vệ branch quan trọng | Settings → Repository |
| Issues | Quản lý task/bug | Issues |
| Board | Kanban board | Issues → Board |
| Wiki | Tài liệu | Wiki |
| Container Registry | Docker images | Packages → Container Registry |
| Snippets | Chia sẻ code nhỏ | Snippets |

---

← [04 - Git Nâng Cao](04-git-nang-cao.md) | **Tiếp theo**: [06 - Workflow & Best Practices](06-workflow-best-practices.md) →
