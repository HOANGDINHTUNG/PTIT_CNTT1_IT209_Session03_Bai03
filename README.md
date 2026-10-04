# Bài 3: Bảo mật tài nguyên bằng HTTP Basic Authentication

## 1. Mục tiêu
- Cài đặt công cụ băm mật khẩu `apache2-utils`.
- Tạo tệp mật khẩu ẩn `.htpasswd` lưu trữ thông tin đăng nhập được mã hoá.
- Cấu hình Nginx yêu cầu xác thực bằng cơ chế **HTTP Basic Authentication** cho thư mục `/admin`.

## 2. Các tệp tin đính kèm
- `.htpasswd`: File mẫu chứa thông tin tài khoản và mật khẩu đã băm (ví dụ mã hoá MD5 apr1).
- `my-web.conf`: File cấu hình Nginx có chứa block location sử dụng `auth_basic`.

## 3. Các bước thực hiện trên Server

### Bước 1: Cài đặt công cụ và tạo file htpasswd
Cài đặt gói `apache2-utils` để sử dụng lệnh `htpasswd`:
```bash
sudo apt update
sudo apt install apache2-utils -y
```

Sử dụng lệnh `htpasswd` để tạo tệp mật khẩu mới. File được lưu ở `/etc/nginx/` để tránh bị lộ mã nguồn ra bên ngoài internet.
```bash
sudo htpasswd -c /etc/nginx/.htpasswd admin_user
```
*(Lệnh này sẽ nhắc bạn nhập và xác nhận mật khẩu 2 lần cho tài khoản `admin_user`)*.

### Bước 2: Bổ sung cấu hình vào Server Block Nginx
Mở file cấu hình Server Block của website hiện tại:
```bash
sudo nano /etc/nginx/sites-available/my-web.conf
```
Thêm chỉ thị cấu hình yêu cầu chứng thực cho location `/admin` như sau (hoặc tham khảo file `my-web.conf` đính kèm):

```nginx
location /admin {
    auth_basic "Restricted Admin Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
    try_files $uri $uri/ =404;
}
```
Lưu file và thoát.

### Bước 3: Kiểm tra cú pháp và áp dụng
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 4. Kiểm tra kết quả (Testing)

Từ máy tính cá nhân hoặc Terminal, hãy sử dụng `curl -I` để kiểm tra các response header.

**1. Gửi request thông thường tới thư mục bị khóa:**
```bash
curl -I http://<IP_ADDRESS_DROPLET>/admin
```
- **Kết quả mong đợi:** Nginx từ chối truy cập và trả về mã `HTTP/1.1 401 Unauthorized` kèm theo header `WWW-Authenticate: Basic realm="Restricted Admin Area"`.

**2. Gửi request kèm thông tin đăng nhập hợp lệ:**
*(Thay `PasswordCuaBan` bằng mật khẩu bạn đã thiết lập ở bước 1)*
```bash
curl -u admin_user:PasswordCuaBan -I http://<IP_ADDRESS_DROPLET>/admin
```
- **Kết quả mong đợi:** Đăng nhập thành công, không còn lỗi 401. Trả về mã `HTTP/1.1 200 OK` (nếu có sẵn file index trong thư mục) hoặc `HTTP/1.1 404 Not Found` (nếu chưa tạo thư mục/file).
