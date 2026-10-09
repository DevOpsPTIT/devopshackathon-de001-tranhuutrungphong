# DevOps Hackathon – Đề 001: Quản lý phòng Lab

## 1. Thông tin sinh viên
| Họ và tên | Mã sinh viên | Lớp | Tài khoản Linux | GitHub | Cổng Nginx |
|---|---|---|---|---|---|
| Trần Hữu Trung Phong | B24DCCN001 | K24CNTT1 | trungphong-k24cntt1 | [phongptit0612](https://github.com/phongptit0612) | 8088 |

---

## 2. Môi trường triển khai
- **Hệ điều hành**: Ubuntu 24.04 LTS (x86_64)
- **Nơi chạy**: VPS Cloud (`103.38.237.21`)
- **Web Server**: Nginx 1.24+ (Ubuntu package)
- **Quản lý mã nguồn**: Git 2.43+
- **Tường lửa**: UFW (Uncomplicated Firewall)

---

## 3. Cấu trúc dự án
```text
devops-hackathon-de001-trungphong/
├── src/
│   └── index.html
├── nginx/
│   └── trungphong-k24cntt1.conf
├── screenshots/
│   ├── 
│   ├── 
│   ├── 
│   ├── 
│   ├── 
│   └── 
├── .gitignore
└── README.md
```

- **Nguyên tắc bảo mật**: Thư mục `src/` là Nginx Web Root duy nhất được cấp phát quyền phục vụ ra Internet. Các thư mục cấu hình `nginx/`, hình ảnh `screenshots/`, tệp `.gitignore` và `README.md` được lưu trữ an toàn nội bộ, hoàn toàn không bị lộ ra website bên ngoài.

---

## 4. Cấu hình Nginx
File cấu hình: `nginx/trungphong-k24cntt1.conf`

| Tham số trong template | Giá trị đã điền | Giải thích |
|---|---|---|
| `<PORT>` | `8088` | Số cổng Nginx riêng biệt được phân bổ (từ 8080 trở lên), hoàn toàn tránh xung đột cổng 80 chung |
| `<SERVER_NAME>` | `103.38.237.21` | Địa chỉ IP công khai của máy chủ VPS (lấy từ lệnh `hostname -I`) |
| `<WEB_ROOT>` | `/var/www/devops-hackathon-de001-trungphong/src` | Đường dẫn tuyệt đối trỏ trực tiếp vào thư mục chứa giao diện web tĩnh |
| `<INDEX_FILE>` | `index.html` | Tệp chỉ mục mặc định được trả về khi người dùng truy cập |
| `<TEN_TAI_KHOAN>` | `trungphong-k24cntt1` | Tên tài khoản Linux dùng làm tiền tố định danh cho log file `access.log` và `error.log` |
| `<ALLOW_DIRECTIVE>` | `allow all;` | Chỉ thị Nginx cho phép mọi request từ bên ngoài truy cập (tương đương `Require all granted` trong Apache) |

---

## 5. Tường lửa UFW
Các rule cấu hình tường lửa:
- Cổng SSH: `22/tcp` (cho phép trước khi kích hoạt UFW để tránh khóa quyền truy cập VPS)
- Cổng Web dịch vụ: `8088/tcp` từ mọi nguồn (`Anywhere` trên cả IPv4 và IPv6)

Kết quả lệnh `sudo ufw status verbose`:
```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                  
8088/tcp                   ALLOW IN    Anywhere                  
22/tcp (v6)                ALLOW IN    Anywhere (v6)             
8088/tcp (v6)              ALLOW IN    Anywhere (v6)             
```

---

## 6. Các bước triển khai

### Bước 1: Tạo tài khoản Linux và thiết lập quyền hạn
```bash
# Tạo user mới với thư mục home, shell mặc định /bin/bash, nhóm chính cùng tên và nhóm phụ sudo
sudo useradd -m -s /bin/bash -U -G sudo trungphong-k24cntt1

# Đặt mật khẩu cho user mới
sudo passwd trungphong-k24cntt1

# Chuyển sang phiên làm việc của user mới
su - trungphong-k24cntt1

# Kiểm tra danh tính người dùng
id
whoami
```

### Bước 2: Cài đặt phần mềm và cấu hình Git máy chủ
```bash
# Cập nhật danh sách gói và cài đặt nginx, git, ufw, curl
sudo apt update
sudo apt install -y nginx git ufw curl

# Khởi chạy Nginx và kích hoạt khởi động cùng OS
sudo systemctl enable --now nginx
sudo systemctl status nginx

# Cấu hình danh tính Git trên server đồng bộ với GitHub
git config --global user.name "Tran Huu Trung Phong"
git config --global user.email "ptran4109@gmail.com"
```

### Bước 3: Deploy mã nguồn từ GitHub lên máy chủ
```bash
# Clone repository về thư mục web theo chuẩn đề bài
sudo git clone https://github.com/phongptit0612/devops-hackathon-de001-trungphong.git /var/www/devops-hackathon-de001-trungphong

# Bàn giao quyền sở hữu thư mục về cho user trungphong-k24cntt1
sudo chown -R trungphong-k24cntt1:trungphong-k24cntt1 /var/www/devops-hackathon-de001-trungphong

# Phân quyền chuẩn: Thư mục 755, Tệp tin 644 (không dùng 777)
sudo find /var/www/devops-hackathon-de001-trungphong -type d -exec chmod 755 {} +
sudo find /var/www/devops-hackathon-de001-trungphong -type f -exec chmod 644 {} +
```

### Bước 4: Cấu hình Server Block Nginx
```bash
# Sao chép tệp cấu hình từ repo vào thư mục sites-available
sudo cp /var/www/devops-hackathon-de001-trungphong/nginx/trungphong-k24cntt1.conf /etc/nginx/sites-available/trungphong-k24cntt1.conf

# Tạo symbolic link sang sites-enabled để kích hoạt website
sudo ln -sf /etc/nginx/sites-available/trungphong-k24cntt1.conf /etc/nginx/sites-enabled/

# Gỡ bỏ site mặc định default để giải phóng cổng và tránh xung đột
sudo rm -f /etc/nginx/sites-enabled/default

# Kiểm tra cú pháp cấu hình Nginx
sudo nginx -t

# Nạp lại cấu hình dịch vụ Nginx
sudo systemctl reload nginx

# Kiểm tra mã trạng thái HTTP nội bộ
curl -I http://103.38.237.21:8088
```

### Bước 5: Cấu hình tường lửa UFW
```bash
# Mở cổng SSH 22 trước khi bật tường lửa
sudo ufw allow 22/tcp

# Mở cổng web cá nhân 8088
sudo ufw allow 8088/tcp

# Kích hoạt tường lửa UFW
sudo ufw --force enable

# Kiểm tra chi tiết trạng thái và các quy tắc tường lửa
sudo ufw status verbose
```

---

## 7. Kiểm tra & minh chứng
- **Địa chỉ truy cập trình duyệt**: `http://103.38.237.21:8088`
- **Ảnh minh chứng website**:
![Trang web](screenshots/04-website.png)

---

## 8. Quy trình cập nhật website
Quy trình chuẩn khi thay đổi nội dung trang web:
1. **Chỉnh sửa tại máy phát triển**: Cập nhật thông tin trong tệp `src/index.html`.
2. **Commit và đẩy lên GitHub**:
   ```bash
   git add src/index.html
   git commit -m "feat: update lab announcement in index.html"
   git push origin main
   ```
3. **Đồng bộ hóa trên máy chủ VPS**:
   ```bash
   cd /var/www/devops-hackathon-de001-trungphong
   git pull origin main
   ```
   *(Không cần sử dụng quyền `sudo` vì thư mục đã được bàn giao quyền sở hữu cho tài khoản `trungphong-k24cntt1`)*.
4. **Xác nhận**: Truy cập lại URL `http://103.38.237.21:8088` trên trình duyệt hoặc chạy `curl -I http://103.38.237.21:8088` để kiểm tra thay đổi mà không cần reload hay restart Nginx.

---

## 9. Sự cố gặp phải & cách khắc phục
- **Sự cố 1: Lỗi Permission Denied khi thực hiện `git pull` trên server**
  - *Nguyên nhân*: Do ban đầu thư mục `/var/www/...` được tạo hoặc clone bằng quyền `root`.
  - *Cách khắc phục*: Chạy lệnh `sudo chown -R trungphong-k24cntt1:trungphong-k24cntt1 /var/www/devops-hackathon-de001-trungphong` để user quản lý có toàn quyền cập nhật mã nguồn mà không cần sudo.
- **Sự cố 2: Ngắt kết nối SSH khi bật tường lửa UFW**
  - *Nguyên nhân*: Bật UFW trước khi cho phép rule cổng 22/tcp.
  - *Cách khắc phục*: Luôn thực hiện `sudo ufw allow 22/tcp` trước khi chạy lệnh `sudo ufw enable`.

