# Bài 3 — Thiết lập Cơ sở dữ liệu và Systemd cho Spring Boot

## 1. Mục tiêu

- Tạo cơ sở dữ liệu MySQL `springboot_db`.
- Tạo tài khoản MySQL `spring-admin` và cấp toàn quyền trên `springboot_db`.
- Sử dụng người dùng hệ thống Linux `spring-runner` (không có shell login) để chạy ứng dụng.
- Tạo dịch vụ `spring-app.service` để quản lý ứng dụng Spring Boot trên cổng `8082`.
- Tự khởi động lại sau **10 giây** khi tiến trình bị lỗi.

## 2. Thông tin cấu hình

| Thành phần                    | Giá trị                                  |
| ----------------------------- | ---------------------------------------- |
| Hệ điều hành                  | Ubuntu                                   |
| Database                      | `springboot_db`                          |
| MySQL user                    | `spring-admin`                           |
| Linux user                    | `spring-runner`                          |
| Ứng dụng                      | `/opt/spring-app/app.jar`                |
| File Systemd                  | `/etc/systemd/system/spring-app.service` |
| File biến môi trường trên VPS | `/etc/spring-app/spring-app.env`         |
| Cổng ứng dụng                 | `8082`                                   |
| Chính sách khởi động lại      | `Restart=on-failure`, `RestartSec=10`    |

## 3. Các bước triển khai

### 3.1. Tạo database và cấp quyền

Chạy trong terminal Ubuntu:

```bash
sudo mysql
```

Sau đó chạy trong MySQL:

```sql
CREATE DATABASE IF NOT EXISTS springboot_db;
CREATE USER IF NOT EXISTS 'spring-admin'@'localhost' IDENTIFIED BY '<MAT_KHAU_THEO_DE_BAI>';
GRANT ALL PRIVILEGES ON springboot_db.* TO 'spring-admin'@'localhost';
SHOW GRANTS FOR 'spring-admin'@'localhost';
EXIT;
```

> Khi triển khai thật, thay `<MAT_KHAU_THEO_DE_BAI>` bằng mật khẩu được giao. Không đăng mật khẩu thật lên GitHub.

### 3.2. Tạo tài khoản Linux chạy dịch vụ

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin spring-runner
id spring-runner
getent passwd spring-runner
```

> Nếu `spring-runner` đã tồn tại thì bỏ qua lệnh `useradd`.

### 3.3. Chuẩn bị file JAR và cấu hình kết nối MySQL

File `/opt/spring-app/app.jar` phải là ứng dụng Spring Boot **đã build thật**, không phải file rỗng. Có thể kiểm tra bằng:

```bash
ls -lh /opt/spring-app/app.jar
java -version
```

Tạo file `/etc/spring-app/spring-app.env` **trên VPS**, không đưa file này vào GitHub:

```ini
SPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/springboot_db
SPRING_DATASOURCE_USERNAME=spring-admin
SPRING_DATASOURCE_PASSWORD=<MAT_KHAU_THEO_DE_BAI>
```

Phân quyền cho file môi trường:

```bash
sudo chown root:root /etc/spring-app/spring-app.env
sudo chmod 600 /etc/spring-app/spring-app.env
```

### 3.4. Cài đặt và kích hoạt Systemd

Sao chép file `spring-app.service` của bài nộp vào `/etc/systemd/system/`, sau đó chạy:

```bash
sudo systemctl daemon-reload
sudo systemctl enable spring-app.service
sudo systemctl start spring-app.service
```

## 4. Kiểm tra và ảnh minh chứng

**Trạng thái kết quả:** Chưa điền — cần kiểm tra lại sau khi VPS được mở và ứng dụng chạy thành công. Không kết luận đạt khi chưa có kết quả thực tế.

### 4.1. Trạng thái dịch vụ

Lệnh:

```bash
sudo systemctl status spring-app.service --no-pager
```

Kết quả mong đợi: `Active: active (running)`.

**[CHÈN ẢNH 1 — KẾT QUẢ SYSTEMCTL STATUS TẠI ĐÂY]**

Gợi ý sau khi chụp: lưu ảnh vào `images/01-status.png` rồi thay dòng ở trên bằng:

c:\Users\HP\OneDrive\Desktop\IT 214\Session07\Ex03\homework\session_07\ex3\img\Screenshot 2026-10-08 210454.png

### 4.2. Kiểm tra cổng 8082

Lệnh:

```bash
sudo ss -tlnp | grep 8082
```

Kết quả mong đợi: xuất hiện `LISTEN` ở cổng `8082` với tiến trình Java.

**[CHÈN ẢNH 2 — KẾT QUẢ KIỂM TRA CỔNG 8082 TẠI ĐÂY]**

Gợi ý: lưu `images/02-port.png`, rồi thay dòng ở trên bằng:

c:\Users\HP\OneDrive\Desktop\IT 214\Session07\Ex03\homework\session_07\ex3\img\Screenshot 2026-10-08 212813.png

### 4.3. Kiểm tra tài khoản thực thi

Lệnh:

```bash
sudo systemctl show spring-app.service -p User
ps -o user,pid,args -C java
```

Kết quả mong đợi: `User=spring-runner`; tiến trình Java tương ứng chạy dưới quyền `spring-runner`.

**[CHÈN ẢNH 3 — USER SPRING-RUNNER TẠI ĐÂY]**

Gợi ý: lưu `images/03-user.png`, rồi thay dòng ở trên bằng:

c:\Users\HP\OneDrive\Desktop\IT 214\Session07\Ex03\homework\session_07\ex3\img\Screenshot 2026-10-08 212849.png

### 4.4. Kiểm tra MySQL (ảnh bổ sung, nếu có)

Lệnh:

```bash
sudo mysql -e "SHOW DATABASES LIKE 'springboot_db'; SHOW GRANTS FOR 'spring-admin'@'localhost';"
```

**[CHÈN ẢNH 4 — DATABASE VÀ QUYỀN MYSQL TẠI ĐÂY (TÙY CHỌN)]**

Gợi ý: lưu `images/04-mysql.png`, rồi thay dòng ở trên bằng:

`![Database và quyền MySQL](images/04-mysql.png)`

## 5. Kết luận

Sau khi bổ sung ảnh chụp **kết quả thật**, đối chiếu với tiêu chí:

- [ ] Dịch vụ hiển thị `active (running)`.
- [ ] Cổng `8082` đang ở trạng thái `LISTEN`.
- [ ] Tiến trình Java chạy bằng user `spring-runner`.
- [ ] File `spring-app.service` có `Restart=on-failure` và `RestartSec=10`.

> Nếu chưa đạt, kiểm tra lỗi bằng `sudo journalctl -u spring-app.service -n 50 --no-pager`.
