# Troubleshooting Laravel Demo App trên K8s

## Kiểm tra Pod Status

```bash
# Xem pods
kubectl get pods -l app=laravel-demo

# Xem chi tiết pod
kubectl describe pod <pod-name>

# Xem logs
kubectl logs <pod-name> --tail=100

# Xem logs real-time
kubectl logs -f <pod-name>
```

## Kiểm tra Health Checks

```bash
# Test health endpoint từ trong pod
kubectl exec <pod-name> -- curl http://localhost/up

# Test từ bên ngoài
kubectl port-forward <pod-name> 8080:80
curl http://localhost:8080/up
```

## Các lỗi thường gặp

### 1. Pod Degraded - Liveness/Readiness Probe Failed

**Nguyên nhân:**
- Laravel chưa khởi động xong
- APP_KEY chưa được generate
- Storage permissions chưa đúng
- Nginx hoặc PHP-FPM chưa start

**Giải pháp:**
- Kiểm tra logs để xem lỗi cụ thể
- Tăng `initialDelaySeconds` trong probes
- Kiểm tra start.sh có chạy đúng không

### 2. 500 Internal Server Error

**Nguyên nhân:**
- APP_KEY chưa được set
- Storage permissions
- Database connection (nếu có)

**Giải pháp:**
```bash
# Vào trong pod
kubectl exec -it <pod-name> -- bash

# Kiểm tra .env
cat .env | grep APP_KEY

# Generate key nếu cần
php artisan key:generate --force

# Kiểm tra permissions
ls -la storage/
ls -la bootstrap/cache/
```

### 3. Health Endpoint `/up` không hoạt động

**Giải pháp:**
- Kiểm tra route có được register không: `php artisan route:list | grep up`
- Thử dùng root path `/` thay vì `/up` trong health checks
- Kiểm tra Nginx config có đúng không

## Debug Commands

```bash
# Xem tất cả events
kubectl get events --sort-by='.lastTimestamp'

# Xem deployment status
kubectl rollout status deployment/laravel-demo

# Xem service endpoints
kubectl get endpoints laravel-demo-service

# Test service từ trong cluster
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- curl http://laravel-demo-service/up
```

## Rebuild và Redeploy

```bash
# Xóa pod để force recreate
kubectl delete pod <pod-name>

# Restart deployment
kubectl rollout restart deployment/laravel-demo

# Xem rollout history
kubectl rollout history deployment/laravel-demo

# Rollback nếu cần
kubectl rollout undo deployment/laravel-demo
```

