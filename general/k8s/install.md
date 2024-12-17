## Hướng Dẫn Cài Đặt Kubekey

[KubeKey](https://github.com/kubesphere/kubekey) là một công cụ mã nguồn mở nhẹ giúp triển khai các cụm Kubernetes. Nó cung cấp một cách linh hoạt, ah chóng và thuận tiện để cài đặt Kubernetes/K3s, cả hai Kubernetes/K3s và KubeSphere, và các add-ons liên quan đến cloud-native. Nó cũng là một công cụ hiệu quả để mở rộng và nâng cấp cụm của bạn.
[Kubekey Github]

### 1. Kiến trúc K8s

| Loại máy chủ | Tên máy chủ | IP máy chủ |
| --- | --- | --- |
| Master,Worker | HNTV-K8S-CLUSTER-01 | 172.16.22.11 |
| Master,Worker | HNTV-K8S-CLUSTER-02 | 172.16.22.12 |
| Master,Worker | HNTV-K8S-CLUSTER-03 | 172.16.22.13 |

### 2. Thiết lập SSH từ node master tới tất cả các máy chủ

Thực hiện theo hướng dẫn của [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)

Tạo khóa SSH:
```sh
ssh-keygen
```

Sao chép khóa SSH đến các node:
```sh
ssh-copy-id root@192.168.1.24
ssh-copy-id root@192.168.1.25
ssh-copy-id root@192.168.1.26
```


ssh root@103.161.170.82 -p 2224
Mật khẩu: `aart@2024`

### 3. Cài đặt các phụ thuộc cho tất cả các node

```sh
dnf install -y tar.x86_64 socat conntrack ebtables ipset ipvsadm nfs-utils iscsi-initiator-utils git
```

### 4. Tải KubeKey mới nhất từ node master

```sh
mkdir kubesphere
cd kubesphere
curl -sfL https://get-kk.kubesphere.io | VERSION=v3.0.10 sh -
chmod +x kk
```

### 5. Tạo cấu hình Kubesphere

Bỏ qua bước này nếu đã có cấu hình sẵn:
```sh
./kk create config --with-kubesphere v3.3.1 -f config/config.yaml
```

Thay đổi cấu hình của bạn theo [Sample Config](./config/sample-config.yaml), để triển khai sản phẩm tốt hơn, bạn nên sử dụng HAProxy để tạo cụm HA K8s theo [Link](https://kubesphere.io/docs/v3.3/installing-on-linux/high-availability-configurations/set-up-ha-cluster-using-keepalived-haproxy/).

Vui lòng kích hoạt cài đặt cho: metrics_server, servicemesh và network (ippool calico).

### 6. Tạo cụm K8s

```sh
./kk create cluster -f ./config/config.yaml
```
Điều này sẽ cài đặt Docker cho tất cả các node và cài đặt K8s.

Hoặc bạn có thể nâng cấp cụm:

```sh
./kk upgrade -f ./config/config.yaml
```

### 7. Cài đặt K9s

[K9s](https://github.com/derailed/k9s) cung cấp giao diện UI terminal để tương tác với các cụm Kubernetes của bạn.

```sh
curl -sS https://webinstall.dev/k9s | bash
export PATH="/root/.local/bin:$PATH"
source ~/.config/envman/PATH.env
```

### 8. Cài đặt Istio

### Với Istioctl (Khuyên dùng)

Cài đặt với [Istioctl](https://istio.io/latest/docs/setup/getting-started/#download)

```sh
curl -L https://istio.io/downloadIstio | sh -
```

Sau đó export PATH:

```sh
export PATH="$PATH:/root/tek4tv-devops-internal/HNTV-K8s/istio-1.22.1/bin"
```

Cài đặt Istio với CNI:

```sh
istioctl install --set profile=default --set components.cni.enabled=true
```

Cấu hình Istio Gateway:

```sh
kubectl edit svc -n istio-system istio-ingressgateway
```

Thay đổi nodePort của "-name: http2" thành 30000 và lưu lại.

Tăng số lượng replicas của Istio Ingress Gateway HPA:

```sh
kubectl edit hpa -n istio-system istio-ingressgateway
```

Thay đổi minReplicas từ 1 thành 3 và lưu lại.

Tăng số lượng replicas của Istiod HPA:

```sh
kubectl edit hpa -n istio-system istiod
```

Thay đổi minReplicas từ 1 thành 3 và lưu lại.

Để xác minh:

```sh
kubectl get pod -n istio-system
```

### 9. Tự động xóa các pod bị evicted

```sh
kubectl apply -f ../utilities/delete-evicted-pods.yml
```

Trên đây là các bước hướng dẫn cài đặt và cấu hình cụm Kubernetes với KubeKey và các công cụ liên quan. Chúc bạn thành công!