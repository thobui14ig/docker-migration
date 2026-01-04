CÁCH 1 (KHUYÊN DÙNG): GitHub Actions SSH vào VPS → pull image → docker compose up

👉 Không cần cài thêm tool phức tạp
👉 Phù hợp với Docker Compose hiện tại của bạn
👉 Rất phổ biến cho VPS cá nhân

🔁 Flow sau khi áp dụng
push code frontend-seo
→ GitHub Actions build image
→ push Docker Hub
→ SSH vào VPS
→ docker compose pull frontend-seo
→ docker compose up -d frontend-seo
→ DONE 🚀

1️⃣ Chuẩn bị trên VPS (chỉ làm 1 lần)
✔ Đảm bảo VPS đã:

Cài Docker

Cài docker-compose

Có repo docker-migration

Ví dụ:

/home/docker/docker-migration
  └── docker-compose.yml

2️⃣ Tạo SSH key cho GitHub Actions
Trên VPS:
ssh-keygen -t rsa -b 4096 -C "github-actions"


➡️ Copy public key:

cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

3️⃣ Thêm Secrets vào GitHub repo frontend-seo

Vào Settings → Secrets → Actions → New secret

Tên	Giá trị
VPS_HOST	IP VPS
VPS_USER	user (vd: root)
VPS_SSH_KEY	nội dung id_rsa (PRIVATE KEY)

⚠️ Copy toàn bộ private key, bao gồm:

-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----

4️⃣ Update GitHub Actions CI (QUAN TRỌNG)
👉 Thêm bước Deploy to VPS
name: Docker Image CI

on:
  push:
    branches: ["master"]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: Dockerfile
          push: true
          tags: |
            thobui1996/frontend-seo:latest
            thobui1996/frontend-seo:${{ github.sha }}

      # 🚀 AUTO DEPLOY
      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /home/docker/docker-migration
            docker compose pull frontend-seo
            docker compose up -d frontend-seo
            docker image prune -f