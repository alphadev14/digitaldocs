# 🚀 Lab 03: GitLab CI/CD Setup (React + Vite + Nginx)

Các bước triển khai **CI/CD với GitLab Runner để
build và deploy ứng dụng React (Vite) lên Nginx server** với 3 môi
trường:

- **Development** → `dev.alphasoft.com`
- **Staging** → `staging.alphasoft.com`
- **Production** → `alphasoft.com`

---

# 1. Kiến trúc hệ thống

Developer
│
│ git push
▼
GitLab Repository
│
│ Trigger Pipeline
▼
GitLab CI/CD
│
▼
GitLab Runner (Server)
│
│ Build React App
▼
dist/
│
▼
Deploy to Nginx

dev.alphasoft.com → /var/www/dev-alphasoft
staging.alphasoft.com → /var/www/staging-alphasoft
alphasoft.com → /var/www/alphasoft

---

# 2. Cài đặt GitLab Runner

## 2.1 Cài đặt runner trên Ubuntu

```bash
sudo apt update
sudo apt install gitlab-runner -y
```

Kiểm tra:

```bash
gitlab-runner --version
```

---

## 2.2 Register Runner

```bash
sudo gitlab-runner register
```

Nhập thông tin:

`GitLab URL: http://git.alphasoft.com
Registration token: (lấy từ GitLab project)
Description: react-runner
Tags: react
Executor: shell`

---

# 3. Cấu hình Nginx

## 3.1 Tạo thư mục deploy

```bash
sudo mkdir -p /var/www/dev-alphasoft
sudo mkdir -p /var/www/staging-alphasoft
sudo mkdir -p /var/www/alphasoft
```

---

## 3.2 Phân quyền

```bash
sudo chown -R gitlab-runner:gitlab-runner /var/www
```

---

# 4. Cấu hình Nginx Virtual Host

## Dev Environment

```nginx
server {
    listen 80;
    server_name dev.alphasoft.com;

    root /var/www/dev-alphasoft;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

## Staging Environment

```nginx
server {
    listen 80;
    server_name staging.alphasoft.com;

    root /var/www/staging-alphasoft;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

## Production Environment

```nginx
server {
    listen 80;
    server_name alphasoft.com;

    root /var/www/alphasoft;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

Enable site:

```bash
sudo ln -s /etc/nginx/sites-available/dev.alphasoft.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/staging.alphasoft.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/alphasoft.com /etc/nginx/sites-enabled/
```

Reload nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

# 5. Mapping domain (Local Lab)

Chỉnh file hosts:

    C:\Windows\System32\drivers\etc\hosts

Thêm:

- 192.168.229.102 dev.alphasoft.com
- 192.168.229.102 staging.alphasoft.com
- 192.168.229.102 alphasoft.com

Flush DNS:

```bash
ipconfig /flushdns
```

---

# 6. GitLab CI/CD Pipeline

File `.gitlab-ci.yml`

```yaml
stages:
  - build
  - deploy_dev
  - deploy_staging
  - deploy_prod

build:
  stage: build
  tags:
    - react
  script:
    - npm cache clean --force
    - rm -rf node_modules
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

deploy_dev:
  stage: deploy_dev
  tags:
    - react
  variables:
    GIT_STRATEGY: none
  dependencies:
    - build
  rules:
    - if: '$CI_COMMIT_BRANCH == "dev" && $CI_COMMIT_TITLE =~ /beta/i'
  script:
    - rm -rf /var/www/dev-alphasoft/*
    - cp -r dist/* /var/www/dev-alphasoft/
  environment:
    name: dev
    url: http://dev.alphasoft.com

deploy_staging:
  stage: deploy_staging
  tags:
    - react
  variables:
    GIT_STRATEGY: none
  dependencies:
    - build
  rules:
    - if: '$CI_COMMIT_BRANCH == "master" && $CI_COMMIT_TITLE =~ /prod/i'
  script:
    - rm -rf /var/www/staging-alphasoft/*
    - cp -r dist/* /var/www/staging-alphasoft/
  environment:
    name: staging
    url: http://staging.alphasoft.com

deploy_prod:
  stage: deploy_prod
  tags:
    - react
  variables:
    GIT_STRATEGY: none
  dependencies:
    - build
  rules:
    - if: '$CI_COMMIT_BRANCH == "master" && $CI_COMMIT_TITLE =~ /prod/i'
  when: manual
  script:
    - rm -rf /var/www/alphasoft/*
    - cp -r dist/* /var/www/alphasoft/
  environment:
    name: production
    url: http://alphasoft.com
```

---

# 7. Workflow

## Development

git checkout dev
git commit -m "beta: update ui"
git push

Pipeline:

build → deploy_dev

<p align="center">
  <img src="/digitaldocs/assets/develop.png" alt="gitlab server" width="600">
</p>

## Staging

    git checkout master
    git commit -m "prod: release staging"
    git push

Pipeline:

    build → deploy_staging

<p align="center">
  <img src="/digitaldocs/assets/staging.png" alt="gitlab server" width="600">
</p>

## Production

Manual trigger trong GitLab pipeline:

    deploy_prod

<p align="center">
  <img src="/digitaldocs/assets/production.png" alt="gitlab server" width="600">
</p>
---

# 8. Kết quả

Sau khi hoàn thành:

✔ CI/CD pipeline tự động

<p align="center">
  <img src="/digitaldocs/assets/gitlab-server.png" alt="gitlab server" width="600">
</p>
✔ Deploy React app bằng GitLab Runner
✔ Multi environment deploy (dev / staging / prod)
✔ Nginx serving static build
