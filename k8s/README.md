# Laravel Demo K8s Deployment

Đây là repository chứa các file Kubernetes manifests cho Laravel Demo App. ArgoCD sẽ theo dõi repository này để tự động sync các thay đổi về Kubernetes cluster.

## Cấu trúc

- `deployment.yaml`: Deployment cho Laravel app với Nginx + PHP-FPM
- `service.yaml`: Service để expose app ra ngoài
- `ingress.yaml`: Ingress để route traffic từ domain `laravel-demo.k8s`

## Cách sử dụng

### 1. Apply thủ công (nếu không dùng ArgoCD)

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

### 2. Sử dụng với ArgoCD

1. Tạo ArgoCD Application với:
   - **Source Repo**: `https://github.com/hieptvh18/laravel-devops.git`
   - **Path**: `k8s`
   - **Target Revision**: `HEAD`
   - **Destination**: `default` namespace

2. ArgoCD sẽ tự động sync khi có thay đổi trong repository này

## Cấu hình

### Image

Image được sử dụng: `hieptvh18/laravel-demo:latest`

Jenkins pipeline sẽ tự động update image tag trong `deployment.yaml` khi build mới.

### Ingress

App được expose qua Ingress với host: `laravel-demo.k8s`

Để truy cập từ local, thêm vào `/etc/hosts`:
```
127.0.0.1 laravel-demo.k8s
```

### Environment Variables

App chạy với các env vars:
- `APP_ENV=production`
- `APP_DEBUG=false`
- `LOG_CHANNEL=stderr`

## Troubleshooting

### Lỗi 502 Bad Gateway

1. Kiểm tra pods đang chạy:
   ```bash
   kubectl get pods -l app=laravel-demo
   ```

2. Kiểm tra logs:
   ```bash
   kubectl logs -l app=laravel-demo
   ```

3. Kiểm tra service:
   ```bash
   kubectl get svc laravel-demo-service
   ```

4. Kiểm tra ingress:
   ```bash
   kubectl get ingress laravel-demo-ingress
   kubectl describe ingress laravel-demo-ingress
   ```

### Storage permissions

Nếu gặp lỗi về permissions, initContainer sẽ tự động set permissions cho storage và cache directories.

