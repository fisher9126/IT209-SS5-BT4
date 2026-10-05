# Bài 4: Mô phỏng quy trình Hotfix & Gitflow thực tế

## 1. Bối cảnh
- Nhánh `main` đang chạy production ổn định (v1.0.0) thì phát hiện lỗi nghiêm trọng lộ dữ liệu.
- Nhánh `develop` đang dở dang các tính năng mới, không deploy ngay được.
- Cần vá nóng: tách `hotfix/v1.0.1` từ `main`, sửa, gộp vào `main` + gắn tag `v1.0.1`,
  rồi gộp ngược về `develop` để lỗi không "trôi" lại ở bản phát triển.

## 2. Mô hình Gitflow sử dụng
- `main`: code production ổn định, mỗi release gắn tag.
- `develop`: code đang phát triển cho phiên bản tương lai.
- `hotfix/*`: nhánh vá lỗi khẩn, tách TỪ `main`, sau khi xong gộp vào CẢ `main` LẪN `develop`.

## 3. Các bước thực hiện

### B1. Khởi tạo main v1.0.0 (production)
```bash
git init
git branch -m main
echo 'console.log("App v1.0.0 - stable");' > app.js
git add app.js
git commit -m "release: v1.0.0 stable"
git tag -a v1.0.0 -m "Release v1.0.0"
```

### B2. Tạo develop và làm tính năng mới (dở dang)
```bash
git checkout -b develop
echo 'console.log("new feature WIP");' > feature.js
git add feature.js
git commit -m "feat: start new feature on develop"
```

### B3. Có lỗi khẩn -> tách hotfix TỪ main
```bash
git checkout main
git checkout -b hotfix/v1.0.1
echo 'console.log("App v1.0.1 - security fix (data leak patched)");' > app.js
git add app.js
git commit -m "fix: patch user data leak"
```

### B4. Gộp hotfix vào main + gắn tag release
```bash
git checkout main
git merge --no-ff hotfix/v1.0.1 -m "merge: hotfix v1.0.1 into main"
git tag -a v1.0.1 -m "Release Hotfix 1.0.1"
```
- Dùng `--no-ff` để luôn tạo **merge commit** rõ ràng (chuẩn Gitflow), tránh fast-forward.
- `git tag -a v1.0.1` đánh dấu điểm release trên main.

### B5. Gộp ngược hotfix vào develop (để lỗi không trôi lại)
```bash
git checkout develop
git merge --no-ff hotfix/v1.0.1 -m "merge: hotfix v1.0.1 into develop"
```

### B6. Dọn dẹp nhánh hotfix
```bash
git branch -d hotfix/v1.0.1
```

## 4. Kiểm tra
```bash
git branch -a
git tag
git log --graph --oneline --all
```
Kết quả mong đợi:
- `git tag` liệt kê `v1.0.0` và `v1.0.1`.
- Tag `v1.0.1` chỉ vào commit mới (merge hotfix) trên `main`.
- `develop` chứa commit vá lỗi `fix: patch user data leak`.

## 5. Sơ đồ phân nhánh Gitflow (ASCII)
```
          v1.0.0                         v1.0.1 (tag)
main    o----------------------------------M
         \                                / (merge --no-ff)
          \                 hotfix/v1.0.1 F
           \               /             (fix: patch data leak)
            \             / (tách từ main)
develop      o-----------W---------------M'
             (feat WIP)                  (merge hotfix vào develop)
```
- `o` : commit; `F` : commit vá lỗi trên hotfix; `M`,`M'` : merge commit.
- Hotfix tách từ `main`, gộp vào `main` (gắn tag) và gộp ngược vào `develop`.

## 6. Ảnh chụp `git log --graph --oneline --all`
(Dán ảnh/output thật vào đây)
