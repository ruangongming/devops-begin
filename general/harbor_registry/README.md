### Tạo thư mục cài đặt và thư mục chứa dữ liệu

Tạo thư mục lưu các cấu hình cài đặt tại `/home/sysadmin/open-sources/harbor_registry` và tạo riêng thư mục lưu data của registry tại `/data/harbor_data`. </br>
Lưu ý sẽ dùng user root để cài.
``` sh
sudo -s
mkdir -p /data/harbor_data
mkdir -p /root/devops/general/harbor
cd /root/devops/general/harbor
curl -s https://api.github.com/repos/goharbor/harbor/releases/latest | grep browser_download_url | cut -d '"' -f 4 | grep '\.tgz$' | wget -i -
tar xvzf harbor-offline-installer*.tgz
cd harbor
cp harbor.yml.tmpl harbor.yml
```

### Chuẩn bị certificate cho registry
Tạo thư mục chứa Cert:
``` sh
mkdir -p /root/devops/general/harbor/certs
cd /root/devops/general/harbor/certs
vi harbor.a.vn.key
vi harbor.a.vn.crt
```