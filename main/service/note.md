## **Chạy Pod**
### **Khi tạo mới lần đầu:**
```sh
# 1. Đảm bảo xóa các tài nguyên cũ (nếu có)
kubectl delete service sso-hrm-service -n hrm-dev
kubectl delete deployment sso-hrm-deployment -n hrm-dev

# 2. Tạo mới với --save-config
kubectl create -f sso-hrm-service.yaml -n hrm-dev --save-config
```

### **Cho các lần cập nhật sau:**
```sh
kubectl apply -f sso-hrm-service.yaml -n hrm-dev
```

## **Xóa pod**
### **Xóa bằng file YAML:** 
```sh
kubectl delete -f sso-hrm-service.yaml -n hrm-dev
```

### **Hoặc xóa từng tài nguyên riêng biệt:**
```sh
# Xóa deployment
kubectl delete deployment sso-hrm-deployment -n hrm-dev

# Xóa service
kubectl delete service sso-hrm-service -n hrm-dev
```

### **Hiển thị pod:**
```sh
kubectl get pods -n hrm-dev
```

### **Check log:**
```sh
kubectl logs sso-hrm-deployment-6bd875d596-xkfvg  -n hrm-dev -f
```

### **Kiểm tra service:**
```sh
kubectl get svc -n hrm-dev
```

### **Kiểm tra pod:**
```sh
kubectl describe pod sso-hrm-deployment-88d4d7756-7f5g9 -n hrm-dev
```

103.161.170.82 harbor.aart.vn