# IE104 Project - Web bán hàng

Web bán hàng đơn giản: xem sản phẩm, giỏ hàng, đặt hàng, xem danh sách đơn.

| Thành phần | Công nghệ |
|---|---|
| Frontend | React 19 + TypeScript (Vite) |
| Backend | Laravel 13 (PHP 8.4), REST API |
| Database | MySQL 8 |
| Môi trường | Docker Compose |
---

## 1. Cài đặt lần đầu

### Yêu cầu

- **Docker Desktop** (đang chạy, góc dưới trái hiện "Engine running")
- **Git**
- **VS Code** (khuyến nghị)

Không cần cài PHP, Composer, Node hay MySQL trên máy. Tất cả chạy trong Docker.

### Các bước

Mở **PowerShell** (hoặc terminal trong VS Code) và chạy lần lượt:

```powershell
# 1. Tải code về (chọn thư mục ngắn, tránh Desktop/OneDrive)
git clone https://github.com/caodii1999/IE104_Project.git
cd IE104_Project
git checkout develop

# 2. Tạo file môi trường
Copy-Item backend\.env.example backend\.env
Copy-Item frontend\.env.example frontend\.env

# 3. Dựng và chạy toàn bộ dự án
docker compose up -d --build

# 4. Sinh khóa ứng dụng cho Laravel
docker compose exec backend php artisan key:generate

# 5. Tạo bảng trong database
docker compose exec backend php artisan migrate
```

> **Lần đầu chạy sẽ khá lâu** (vài phút đến 10 phút): Docker tải image, backend chạy `composer install`, frontend chạy `npm install`. Theo dõi tiến độ bằng:
> ```powershell
> docker compose logs -f backend
> ```
> Khi thấy dòng `Server running on [http://0.0.0.0:8000]` thì bấm `Ctrl + C` để thoát xem log (dự án vẫn chạy nền).

**Mac/Linux:** thay `Copy-Item a b` bằng `cp a b`, các lệnh còn lại giữ nguyên.

### Kiểm tra đã chạy đúng chưa

| Dịch vụ | Địa chỉ | Kết quả mong đợi |
|---|---|---|
| Frontend | http://localhost:5173 | Trang mặc định Vite + React |
| Backend | http://localhost:8000 | Trang welcome của Laravel |
| phpMyAdmin | http://localhost:8080 | Đăng nhập `root` / `root`, thấy database `doan_db` có các bảng |

Chạy `docker compose ps`: cả 4 service (`db`, `backend`, `frontend`, `phpmyadmin`) phải ở trạng thái **Up**, riêng `db` là **healthy**.

### Thông tin kết nối database

| Mục | Giá trị |
|---|---|
| Host (từ container backend) | `db` |
| Host (từ máy bạn, ví dụ dùng DBeaver) | `localhost` |
| Cổng (từ máy bạn) | `3307` |
| Database | `doan_db` |
| User / Password | `doan` / `secret` |
| Root password | `root` |

---

## 2. Làm việc hằng ngày

### Bật / tắt

```powershell
docker compose up -d        # bật (không cần build lại)
docker compose down         # tắt, GIỮ dữ liệu database
docker compose down -v      # tắt và XÓA SẠCH database (làm lại từ đầu)
docker compose ps           # xem trạng thái
docker compose logs -f backend    # xem log (đổi backend thành frontend, db...)
```

### Chạy lệnh trong container

```powershell
docker compose exec backend php artisan migrate               # chạy migration mới
docker compose exec backend php artisan migrate:fresh --seed  # xóa bảng, tạo lại, nạp dữ liệu mẫu
docker compose exec backend php artisan config:clear          # sau khi sửa .env
docker compose exec backend composer require <ten-thu-vien>   # thêm thư viện PHP
docker compose exec frontend npm install <ten-thu-vien>       # thêm thư viện JS
```

### Khi nào cần làm gì sau khi `git pull`

| Thay đổi trong code mới | Việc cần làm |
|---|---|
| Có migration mới | `docker compose exec backend php artisan migrate` |
| Đổi `backend/composer.json` | `docker compose exec backend composer install` |
| Đổi `frontend/package.json` | `docker compose restart frontend` |
| Đổi `docker-compose.yml` hoặc `Dockerfile` | `docker compose up -d --build` |
| Đổi file `.env.example` | Cập nhật file `.env` của bạn cho khớp, rồi `docker compose restart` |

Sửa code PHP và React thì **tự cập nhật**, không cần restart.

---

## 3. Quy trình git

Có 2 nhánh chính được bảo vệ, **không push thẳng**:

- `main`: bản ổn định, chỉ nhận code từ `develop` ở các mốc lớn
- `develop`: nhánh tích hợp, nơi mọi tính năng được gộp vào

Mỗi task làm trên **một nhánh riêng**:

```powershell
git checkout develop
git pull
git checkout -b feature/ten-viec     # ví dụ: feature/order-api

# ... làm việc, sửa file ...

git add .
git commit -m "feat: mo ta ngan"
git push -u origin feature/ten-viec
```

Sau đó vào GitHub, mở **Pull Request** từ nhánh của bạn vào `develop`, nhờ một người review và approve rồi mới merge.

### Quy ước đặt tên

| Loại | Mẫu | Ví dụ |
|---|---|---|
| Nhánh tính năng | `feature/<ten-viec>` | `feature/product-api` |
| Nhánh sửa lỗi | `fix/<ten-loi>` | `fix/cart-total` |
| Nhánh tài liệu | `docs/<ten>` | `docs/api-contract` |
| Commit | `loai: mo ta ngan` | `feat: add POST /orders` |

Các loại commit: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`.

### Trước khi commit, luôn kiểm tra

```powershell
git status
```

**Không được** thấy `.env`, `vendor/`, `node_modules/` trong danh sách file sắp commit. File `.env.example` thì được commit.

---

## 4. Cấu trúc thư mục

```
IE104_Project/
├── backend/             # Laravel (API, migration, seeder)
├── frontend/            # React + TypeScript (Vite)
├── db/                  # ERD, từ điển dữ liệu, mô tả dữ liệu mẫu
├── design/              # Link Figma, wireframe, mockup, design system
├── docs/                # Scope, API contract, quy ước, test case, báo cáo
├── postman/             # Postman collection
├── docker-compose.yml   # Môi trường chạy chung
└── README.md
```

> Thư mục `db/`, `design/`, `docs/`, `postman/` được thêm trong Pull Request riêng về tài liệu.

| Thư mục | Phụ trách |
|---|---|
| `frontend/`, `design/` | A |
| `backend/` (đơn hàng, hạ tầng API) | B |
| `backend/` (sản phẩm), `db/`, `postman/` | C |
| `docs/` | Cả nhóm |

**Lưu ý:** migration và seeder nằm ở `backend/database/`, không nằm ở `db/`. Thư mục `db/` chỉ chứa tài liệu thiết kế.

### Cổng sử dụng

| Cổng | Dịch vụ |
|---|---|
| 5173 | Frontend (Vite) |
| 8000 | Backend (Laravel) |
| 8080 | phpMyAdmin |
| 3307 | MySQL (từ máy bạn) |

---

## 5. Xử lý lỗi thường gặp

**Docker báo không kết nối được / `docker` lỗi**
Docker Desktop chưa chạy. Mở Docker Desktop, chờ "Engine running" rồi thử lại.

**Cổng bị chiếm** (lỗi `port is already allocated`)
Một chương trình khác đang dùng cổng 5173, 8000, 8080 hoặc 3307 (thường là XAMPP hoặc MySQL cài sẵn). Tắt chương trình đó, hoặc đổi số **bên trái** dấu `:` trong `ports` của `docker-compose.yml` (ví dụ `"8001:8000"`).

**Backend cứ khởi động lại liên tục** (`Restarting` trong `docker compose ps`)
Xem nguyên nhân:
```powershell
docker compose logs backend --tail 50
```
Nếu thấy `requires php >=8.4`, kiểm tra `backend/Dockerfile` dòng đầu phải là `FROM php:8.4-cli` rồi `docker compose up -d --build backend`.

**Backend log dừng ở `Generating optimized autoload files` rất lâu**
Bình thường trên Windows ở lần chạy đầu, chờ 3 đến 5 phút. Từ lần sau sẽ nhanh.

**`SQLSTATE[HY000] [2002] Connection refused` khi migrate**
Kiểm tra `backend/.env`: `DB_CONNECTION=mysql`, `DB_HOST=db`, và các dòng `DB_*` **không có dấu `#`** ở đầu. Sửa xong chạy:
```powershell
docker compose exec backend php artisan config:clear
```

**`Access denied for user` hoặc `Unknown database`**
Thông tin DB trong `backend/.env` phải khớp với `docker-compose.yml` (`doan_db`, `doan`, `secret`). Nếu vẫn lỗi, xóa volume cũ và chạy lại:
```powershell
docker compose down -v
docker compose up -d
docker compose exec backend php artisan migrate
```

**`localhost:8000` báo lỗi 500 "No application encryption key" hoặc "table doesn't exist"**
Chưa chạy `key:generate` hoặc `migrate`. Chạy lại bước 4 và 5 của phần cài đặt.

**Frontend không tự cập nhật khi sửa code**
Kiểm tra `frontend/vite.config.ts` có khối `server` với `watch: { usePolling: true }`.

**Chạy chậm**
Bình thường trên Windows khi code nằm ở ổ thường. Giữ thư mục dự án ở chỗ ngắn (ví dụ `D:\dev\IE104_Project`), tránh Desktop và OneDrive. Có thể tăng RAM, CPU cho Docker trong Settings, Resources.

**Lỗi `env: 'php\r': No such file or directory` hoặc file bị lỗi xuống dòng**
Do Git chuyển xuống dòng kiểu Windows. Chạy một lần rồi clone lại:
```powershell
git config --global core.autocrlf input
```

**Gặp lỗi khác**
Chụp màn hình hoặc copy đoạn log của `docker compose logs <service> --tail 50` và gửi vào nhóm chat kèm hệ điều hành đang dùng.

---

## 6. Tài liệu liên quan

- Phạm vi và deadline: `docs/00-scope-and-deadline.md`
- API contract: `docs/01-api-contract.md`
- Quy ước làm việc: `docs/02-conventions.md`
- Test case: `docs/03-test-cases.md`
- ERD và từ điển dữ liệu: `db/`
- Thiết kế giao diện: `design/`