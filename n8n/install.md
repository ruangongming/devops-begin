# Hướng dẫn cấu hình & cài đặt N8N với Podman

Dưới đây là tổng hợp đầy đủ các bước cài đặt và cấu hình N8N sử dụng Podman, bao gồm tất cả các file cấu hình và script đã tạo.

## 1. Cài đặt Podman và Podman Compose

```bash
# Cài đặt Podman
dnf install -y podman

# Cài đặt Podman Compose
pip3 install podman-compose

# Tạo symbolic link cho podman-compose
ln -s /usr/local/bin/podman-compose /usr/bin/podman-compose
```

## 2. Tạo thư mục cấu hình N8N

```bash
# Tạo thư mục
mkdir -p /etc/n8n
cd /etc/n8n
```

## 3. Tạo file cấu hình Podman Compose

```bash
# Tạo file podman-compose.yml
cat > /etc/n8n/podman-compose.yml << 'EOF'
version: '3'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_SECURE_COOKIE=false
      - N8N_PROTOCOL=http
      - N8N_HOST=172.16.0.35
      - N8N_PORT=5678
      - N8N_EDITOR_BASE_URL=http://172.16.0.35:5678
      # Thêm các biến môi trường bảo mật (tùy chọn)
      # - N8N_ENCRYPTION_KEY=một_khóa_mã_hóa_phức_tạp_tại_đây
      # - N8N_BASIC_AUTH_ACTIVE=true
      # - N8N_BASIC_AUTH_USER=admin
      # - N8N_BASIC_AUTH_PASSWORD=mật_khẩu_phức_tạp_tại_đây
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
EOF
```

## 4. Cấu hình tự động khởi động

### 4.1. Tạo systemd service cho Podman Compose

```bash
# Tạo file systemd service
cat > /etc/systemd/system/n8n-podman.service << 'EOF'
[Unit]
Description=N8N Podman Compose Service
After=network.target

[Service]
Type=simple
WorkingDirectory=/etc/n8n
ExecStart=/usr/bin/podman-compose up
ExecStop=/usr/bin/podman-compose down
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Cấp quyền cho file
chmod 644 /etc/systemd/system/n8n-podman.service

# Kích hoạt service để tự động khởi động cùng hệ thống
systemctl daemon-reload
systemctl enable n8n-podman.service
systemctl start n8n-podman.service
```

### 4.2. Cấu hình Quadlet (cho Podman 4.0+)

```bash
# Tạo thư mục nếu chưa tồn tại
mkdir -p /etc/containers/systemd

# Tạo file quadlet cho n8n
cat > /etc/containers/systemd/n8n.container << 'EOF'
[Container]
Image=docker.n8n.io/n8nio/n8n:latest
PublishPort=5678:5678
Volume=n8n_data:/home/node/.n8n
Environment=N8N_SECURE_COOKIE=false
Environment=N8N_PROTOCOL=http
Environment=N8N_HOST=172.16.0.35
Environment=N8N_PORT=5678
Environment=N8N_EDITOR_BASE_URL=http://172.16.0.35:5678

[Service]
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Kích hoạt systemd generator
systemctl daemon-reload
systemctl enable --now n8n.service
```

### 4.3. Thêm lệnh khởi động vào crontab (phương pháp dự phòng)

```bash
# Mở crontab
(crontab -l 2>/dev/null; echo "@reboot cd /etc/n8n && /usr/bin/podman-compose up -d") | crontab -
```

## 5. Tạo script sao lưu dữ liệu

```bash
# Tạo script sao lưu
cat > /usr/local/bin/backup-n8n.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/root/n8n-backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Tạo thư mục sao lưu nếu chưa tồn tại
mkdir -p $BACKUP_DIR

# Sao lưu volume
podman volume export n8n_data > $BACKUP_DIR/n8n_data_$DATE.tar

# Giữ lại 7 bản sao lưu gần nhất
ls -t $BACKUP_DIR/n8n_data_*.tar | tail -n +8 | xargs -r rm

echo "Đã sao lưu volume n8n_data vào $BACKUP_DIR/n8n_data_$DATE.tar"
EOF

# Cấp quyền thực thi
chmod +x /usr/local/bin/backup-n8n.sh

# Thiết lập cron job để sao lưu hàng ngày vào lúc 2 giờ sáng
(crontab -l 2>/dev/null; echo "0 2 * * * /usr/local/bin/backup-n8n.sh") | crontab -
```

## 6. Tạo script giám sát

```bash
# Tạo script kiểm tra
cat > /usr/local/bin/check-n8n.sh << 'EOF'
#!/bin/bash
if ! podman ps | grep -q n8n; then
  echo "CẢNH BÁO: Container n8n không hoạt động! Đang thử khởi động lại..."
  cd /etc/n8n && podman-compose up -d
  sleep 10
  if ! podman ps | grep -q n8n; then
    echo "LỖI: Không thể khởi động lại container n8n!"
    # Thêm lệnh gửi email hoặc thông báo ở đây
  fi
fi
EOF

# Cấp quyền thực thi
chmod +x /usr/local/bin/check-n8n.sh

# Thiết lập cron job để kiểm tra mỗi 15 phút
(crontab -l 2>/dev/null; echo "*/15 * * * * /usr/local/bin/check-n8n.sh") | crontab -
```

## 7. Tạo script cập nhật tự động

```bash
# Tạo script cập nhật
cat > /usr/local/bin/update-n8n.sh << 'EOF'
#!/bin/bash
set -e

# Đường dẫn đến thư mục chứa file podman-compose.yml
cd /etc/n8n/

# Sao lưu cấu hình hiện tại
cp podman-compose.yml podman-compose.yml.bak.$(date +%Y%m%d_%H%M%S)

# Sao lưu dữ liệu trước khi cập nhật
/usr/local/bin/backup-n8n.sh

# Kéo image mới nhất
podman-compose pull

# Áp dụng thay đổi mới
echo "Áp dụng cấu hình mới..."
podman-compose down
podman-compose up -d

# Kiểm tra trạng thái
if podman ps | grep -q n8n; then
  echo "Cập nhật thành công!"
else
  echo "Lỗi! Khôi phục cấu hình cũ..."
  LATEST_BACKUP=$(ls -t podman-compose.yml.bak.* | head -1)
  cp $LATEST_BACKUP podman-compose.yml
  podman-compose up -d
  echo "Khôi phục từ bản sao lưu: $LATEST_BACKUP"
fi
EOF

# Cấp quyền thực thi
chmod +x /usr/local/bin/update-n8n.sh

# Thiết lập cron job để cập nhật hàng tuần vào Chủ nhật lúc 3 giờ sáng
(crontab -l 2>/dev/null; echo "0 3 * * 0 /usr/local/bin/update-n8n.sh") | crontab -
```

## 8. Khôi phục dữ liệu (nếu cần)

```bash
# Khôi phục từ bản sao lưu
podman volume rm n8n_data || true
podman volume create n8n_data
podman volume import n8n_data /đường/dẫn/đến/file/sao-lưu.tar
```

## 9. Các lệnh hữu ích

```bash
# Khởi động container
cd /etc/n8n && podman-compose up -d

# Dừng container
cd /etc/n8n && podman-compose down

# Xem logs
podman logs n8n

# Kiểm tra trạng thái
podman ps | grep n8n

# Thực hiện sao lưu thủ công
/usr/local/bin/backup-n8n.sh

# Cập nhật n8n thủ công
/usr/local/bin/update-n8n.sh

# Kiểm tra các service đã được kích hoạt
systemctl list-unit-files | grep -E 'podman|n8n'

# Kiểm tra crontab
crontab -l
```

## 10. Cấu trúc thư mục và file

```
/etc/n8n/
  ├── podman-compose.yml                 # File cấu hình chính

/etc/systemd/system/
  ├── n8n-podman.service                 # Systemd service

/etc/containers/systemd/
  ├── n8n.container                      # Cấu hình Quadlet

/usr/local/bin/
  ├── backup-n8n.sh                      # Script sao lưu
  ├── check-n8n.sh                       # Script giám sát
  ├── update-n8n.sh                      # Script cập nhật
  ├── podman-compose                     # Symbolic link đến podman-compose

/root/n8n-backups/                       # Thư mục chứa các bản sao lưu
```

## 11. Truy cập N8N

Sau khi hoàn tất cài đặt, bạn có thể truy cập N8N qua trình duyệt web:

```
http://172.16.0.35:5678
```

## 12. Bảo mật (tùy chọn)

Để tăng cường bảo mật, bạn có thể chỉnh sửa file `podman-compose.yml` để thêm xác thực cơ bản và mã hóa:

```bash
cd /etc/n8n/
nano podman-compose.yml
```

Thêm các biến môi trường sau vào phần `environment`:
```yaml
- N8N_ENCRYPTION_KEY=một_khóa_mã_hóa_phức_tạp_tại_đây
- N8N_BASIC_AUTH_ACTIVE=true
- N8N_BASIC_AUTH_USER=admin
- N8N_BASIC_AUTH_PASSWORD=mật_khẩu_phức_tạp_tại_đây
```

Sau khi thay đổi, áp dụng cấu hình mới:
```bash
cd /etc/n8n/ && podman-compose down && podman-compose up -d
```

---

Với hướng dẫn tổng hợp này, bạn đã có một hệ thống N8N hoàn chỉnh, tự động khởi động, được sao lưu định kỳ, giám sát liên tục và cập nhật tự động. Hệ thống này phù hợp cho môi trường sản xuất với độ tin cậy cao.