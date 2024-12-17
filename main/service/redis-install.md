1. Xóa StatefulSet hiện tại:
```bash
kubectl delete statefulset redis-hrm-service -n hrm-dev
```

2. Cập nhật ConfigMap:
```bash
kubectl apply -f redis-config.yaml -n hrm-dev
```

3. Tạo lại StatefulSet:
```bash
kubectl apply -f redis-hrm-service.yaml -n hrm-dev
```