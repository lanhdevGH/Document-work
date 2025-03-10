# Quy tắc Code Convention C# ASP.NET với Clean Architecture

## 1. Cấu trúc Branch

### 1.1. Branch chính
- `main`: Chỉ chứa code ổn định, đã test kỹ và sẵn sàng triển khai.
  - **KHÔNG** commit trực tiếp vào `main`.
  - Chỉ merge qua Pull Request (PR) đã được review.
  
- `develop`: Branch phát triển chính, tích hợp các tính năng và sửa lỗi.
  - Merge các branch feature và bugfix vào đây trước khi đưa lên `main`.
  - Code trên branch này phải luôn ở trạng thái hoạt động được.

### 1.2. Branch tính năng
- `feature/`: Dùng cho phát triển tính năng mới.
  - Đặt tên rõ ràng theo mẫu: `feature/add-user-authentication`
  - Mỗi tính năng tách thành một branch riêng.
  - Ví dụ: `feature/implement-jwt-auth`, `feature/add-product-filtering`

### 1.3. Branch sửa lỗi
- `bugfix/`: Dùng cho sửa lỗi không khẩn cấp.
  - Mẫu: `bugfix/fix-login-validation`
  - Ví dụ: `bugfix/fix-incorrect-date-format`, `bugfix/fix-null-reference-in-order`

- `hotfix/`: Dùng cho sửa lỗi khẩn cấp trên môi trường production.
  - Tạo từ `main`, merge lại vào cả `main` và `develop`.
  - Ví dụ: `hotfix/fix-payment-processing-crash`, `hotfix/fix-critical-security-issue`

### 1.4. Quy tắc đặt tên branch
- Sử dụng chữ thường và dấu gạch ngang `-`.
- Tên ngắn gọn, tối đa 50 ký tự.
- Không dùng ký tự đặc biệt, khoảng trắng.
- Format: `<loại>/<mô tả ngắn gọn>`

### 1.5. Quản lý branch
- Xóa branch đã merge sau 1-2 tuần khi đã ổn định:
  - Lệnh: `git branch -d <branch-name>`
- Thường xuyên cập nhật branch của bạn với `develop`:
  - `git checkout develop`
  - `git pull origin develop`
  - `git checkout <your-branch>`
  - `git merge develop`

## 2. Quy trình Commit

### 2.1. Nội dung commit
- Mỗi commit chỉ chứa thay đổi cho một nhiệm vụ cụ thể.
- Không gộp nhiều thay đổi không liên quan vào một commit.
- Commit thường xuyên với các thay đổi nhỏ, dễ hiểu.

### 2.2. Cấu trúc message commit
- Dòng đầu: Tối đa 50 ký tự, mô tả ngắn gọn thay đổi.
- Để dòng thứ 2 trống.
- Từ dòng 3: Mô tả chi tiết (nếu cần).

```
Add user registration endpoint

- Implement API endpoint for user registration
- Add input validation using FluentValidation
- Create UserRegistrationDto with proper annotations
```

### 2.3. Quy tắc viết message commit
- Bắt đầu bằng động từ mệnh lệnh, viết hoa chữ cái đầu:
  - `Add`: Thêm chức năng, file mới
  - `Fix`: Sửa lỗi
  - `Update`: Cập nhật chức năng, không phải sửa lỗi
  - `Remove`: Xóa chức năng, file
  - `Refactor`: Tái cấu trúc code
  - `Docs`: Cập nhật tài liệu
  - `Test`: Thêm/sửa test
  - `Chore`: Công việc bảo trì, cập nhật dependency

### 2.4. Tích hợp mã ticket
- Thêm mã ticket vào đầu message (nếu có):
  - `[PROJ-123] Add user login endpoint`
  - `[PROJ-456] Fix payment calculation bug`

### 2.5. Kiểm tra trước khi commit
- Sử dụng pre-commit hook để kiểm tra:
  - Code style (sử dụng `.editorconfig`)
  - Linter warnings
  - Unit tests
- Không commit:
  - Files cấu hình cá nhân (`.vs/`, `.idea/`)
  - Files cấu hình môi trường local (`.env`, `appsettings.Development.json`)
  - Files build (`bin/`, `obj/`, `node_modules/`)

## 3. Pull Request (PR)

### 3.1. Tạo PR
- PR từ branch của bạn vào `develop` (hoặc `main` với hotfix).
- Đặt tiêu đề ngắn gọn, rõ ràng:
  - `Add user authentication endpoint`
  - `Fix data validation in payment process`

### 3.2. Mô tả PR
- Sử dụng mẫu sau:

```
## What
- Thêm API endpoint đăng nhập người dùng
- Implement JWT token generation
- Add caching cho user session

## Why
- Cho phép người dùng đăng nhập vào hệ thống
- Cải thiện trải nghiệm người dùng và bảo mật

## How to test
1. Call POST /api/auth/login với body:
   ```json
   {
     "email": "user@example.com",
     "password": "password123"
   }
   ```
2. Verify JWT token trả về có cấu trúc đúng
3. Kiểm tra token có thể dùng để gọi API khác
```

### 3.3. Quy trình review
- Chờ ít nhất 1 người review và approve PR.
- Yêu cầu reviewer thích hợp (người hiểu domain hoặc công nghệ).
- Phản hồi comments nhanh chóng và lịch sự.
- Không tự merge PR của chính mình.

### 3.4. Merge PR
- Sử dụng `Squash and Merge` để gộp tất cả commit thành một.
- Đảm bảo tất cả CI/CD pipeline đều pass.
- Viết message squash có ý nghĩa, không chỉ mặc định.

### 3.5. Tài liệu PR
- Thêm ảnh/GIF/video minh họa nếu thay đổi UI/UX.
- Link đến ticket/issue liên quan.
- Nêu rõ những thay đổi lớn về database, API, configuration.

## 4. Xử lý Conflict

### 4.1. Cập nhật branch
- Pull code mới nhất từ branch đích (thường là `develop`):
  ```
  git checkout develop
  git pull origin develop
  git checkout <your-branch>
  git merge develop
  ```

### 4.2. Giải quyết conflict
- Kiểm tra các file conflict: `git status`
- Mở các file này và xem các phần được đánh dấu:
  ```
  <<<<<<< HEAD
  // Code từ branch hiện tại
  =======
  // Code từ branch đang merge vào
  >>>>>>> branch-name
  ```
- Chỉnh sửa để giữ code phù hợp với yêu cầu.
- Thêm files đã sửa: `git add <file-name>`

### 4.3. Hoàn thành merge
- Commit với message: `git commit -m "Resolve merge conflicts"`
- Push branch: `git push origin <your-branch>`

### 4.4. Conflict phức tạp
- Với conflict phức tạp, thông báo cho team.
- Có thể dùng công cụ merge như:
  - Visual Studio's merge tool
  - Beyond Compare
  - VS Code với extension Git Lens

## 5. Quy tắc chung

### 5.1. Kiểm tra trước khi push
- Build code: `dotnet build`
- Chạy tests: `dotnet test`
- Kiểm tra code style: `dotnet format`

### 5.2. Cập nhật code
- Pull code từ `develop` ít nhất 1 lần/ngày.
- Đảm bảo luôn làm việc trên code mới nhất.

### 5.3. Quản lý branch
- Xóa branch sau khi merge thành công và kiểm tra ổn định.
- Lệnh: `git branch -d <branch-name>`

### 5.4. Cập nhật tài liệu
- Cập nhật tài liệu API khi thay đổi endpoint.
- Cập nhật README.md khi thay đổi cấu trúc dự án.
- Cập nhật migration guide khi có breaking changes.

### 5.5. Quy tắc bảo vệ branch
- Không push code vào `main`/`develop` trực tiếp.
- Luôn thông qua PR được review.
- Cấu hình branch protection trên GitHub/GitLab.

## 6. Luồng làm việc mẫu

### 6.1. Phát triển tính năng mới
```bash
# Cập nhật develop
git checkout develop
git pull origin develop

# Tạo branch mới
git checkout -b feature/add-user-authentication

# Lập trình và commit
git add .
git commit -m "Add user login endpoint"

# Push branch
git push origin feature/add-user-authentication

# Tạo PR và chờ review
# Sau khi PR được merge:
git checkout develop
git pull origin develop
git branch -d feature/add-user-authentication
```

### 6.2. Hotfix
```bash
# Tạo hotfix từ main
git checkout main
git pull origin main
git checkout -b hotfix/fix-payment-crash

# Sửa lỗi và commit
git add .
git commit -m "Fix payment processing null reference exception"
git push origin hotfix/fix-payment-crash

# Tạo PR vào main, sau khi merge:
git checkout develop
git merge main
git push origin develop
git branch -d hotfix/fix-payment-crash
```

### 6.3. Cập nhật và tag version
- Khi release lên production:
```bash
git checkout main
git pull origin main
git tag v1.0.0
git push origin v1.0.0
```

### 6.4. Cập nhật thông tin remote
```bash
git fetch --all --prune
```
