# Kiến thức cơ bản DevOps
- Devops : là người đưa ra giải pháp giúp tôi ưu quy trình làm việc , tăng chất lượng làm việc và sản phẩm

- Linux : là 1 hệ điều hành : Tối ưu chi phí , bảo mật và ổn định , khả năng kiểm soát và linh hoạt , 
dễ phát triển và cập nhật , tương thích đa nền tảng , cộng đồng phát triển lớn

``` bash
# Lệnh reset IP
ssh-keygen -R 192.168.1.110
```

# Setup IP tĩnh cho môi trường On-pre

``` bash
sudo -i
nano /etc/netplan/ + tab

# Nội dung bên trong
dhcp4: false
addresses: [192.168.1.110/24]
gateway4: 192.168.1.1
nameservers:
  addresses: [8.8.8.8,8.8.4.4]

# Apply cấu hình
netplan apply
```

# Cách lệnh Linux cơ bản

``` bash
cat /etc/passwd # Xem tất cả user và quyền 
pwd # xem đang ở thư mục nào
whoami # xem đang là user nào
cd # để chuyển qua thư mục khác
ls # để xem tất cả các thư mục 
ls -lta # hiển thị tất cả các thư mục file sắp xếp mới đến cũ
ls -l # hiển thị nội dung dưới dạng danh sách 
mkdir # tạo một folder || mkdir -p để cấp thêm quyền tạo folder trong folder
touch # tạo một thư mục 
rm # xoá thư mục hoặc folder || sử dụng lệnh rm -r để cấp quyền xoá || sử dụng lệnh rm -rf 
cp -r folder/ /vitricanchuyentoi/ 
cp file/ /vitri/ 
adduser devops // tạo một user mới
mv /filehoacfoldercanchuyen/ denvitricanchuyen
tail -n folder hoặc file dự án
sudo usermod -aG group_name user_name # them user vao group
sudo deluser alice sudo # xoa user ra khoi group
sudo groupadd developers # tao 1 group moi
scp -r /thư mục cần di chuyển user@<ip-server>:/home/user # di chuyển 1 file vào bên trong server
```

# Triển khai dự án backend Springboot và frontend react dạng service

### Luôn giữ 1 tư duy rằng backend thì triển khai database và làm sao run dự án 

``` bash
### cài đặt theo dõi các tiến trình
apt install net-tools -y
netstat -tlpun
```

## 1. setup cho backend

### 1. setup thư mục, user và môi trường

``` bash
sudo -i
cd /home/user
mkdir project
cd project
git clone https://github.com/duyduc1/ticket-car.git

### setup user cho dự án
adduser backend
chown -R backend. /home/user/project/ticker-car
chmod -R 755 /home/user/project/ticker-car

### setup môi trường cho dự án
apt install openjdk-21-jdk openjdk-21-jre -y
apt install maven -y
```

### 2. setup database

``` bash
### cài đặt database
apt install mysql-server

### cấu hình database
systemctl stop mysql
nano /etc/mysql/mysql.conf.d/mysqld.cnf
# set up 127.0.0.1 thành 0.0.0.0
systemctl restart mysql
mysql -u root
```

- Cấu hình 1 user riêng cho database

``` sql
create database database-project;
create user 'user'@'%' identified by 'password';
grant all privileges on database-project.* to 'user'@'%';
flush privileges;
show databases;
```

- Thay đổi config bên source backend

``` bash
nano src/main/resources/application.properties

# setup ip database từ local thành IP Server
# Username và password thay đổi thành username và password vừa tạo trong database
```

### 3. Tiến hành build dự án

``` bash
mvn clean install -DskipTests=true
ls target/
java -jar target/SpringSecurity.JWT-0.0.1-SNAPSHOT.jar
mkdir -p /var/log/backend
```

### 4. tiến hành setup service backend

``` bash
nano /etc/systemd/system/backend.service

### nội dung bên trong file backend.service
[Service]
Type=simple
User=backend
Restart=Always
WorkingDirectory=/home/user/project/ticker-car
ExecStart= java -jar target/SpringSecurity.JWT-0.0.1-SNAPSHOT.jar
StandardOutput=append:/var/log/backend/backend.log     
StandardError=append:/var/log/backend/backend.err

### Chạy file service
systemctl daemon-reload
systemctl start backend.service
systemctl status backend.service
```
 

## 2. setup cho frontend

### 1. setup thư mục, user và môi trường

``` bash
### setup thư mục dự án
sudo -i
cd /home/user
mkdir project
cd project
git clone https://github.com/duyduc1/frontend.git

### setup user cho dự án
adduser frontend
chown -R frontend. /home/user/project/frontend
chmod -R 755 /home/user/project/frontend

### setup môi trường cho dự án
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
apt-get install nodejs -y
sudo npm install -g npm@latest
```

### 2. tiến hành setup service frontend

``` bash
nano /etc/systemd/system/frontend.service

### nội dung bên trong file backend.service
[Service]
Type=simple
User=frontend
Restart=Always
WorkingDirectory=/home/user/project/frontend
ExecStart= npm run start -- --port=3000

### Chạy file service
systemctl daemon-reload
systemctl start frontend.service
systemctl status frontend.service
```

# Setup Nginx gắn domain

1. build dự án frontend React

``` bash
cd /home/user/project/frontend
npm run build
mv build /var/www/html
```

2. cài đặt nginx và setup domain

``` bash
sudo apt install nginx -y
nano /etc/nginx/conf.d/name-domain.vn.conf

server {
    listen 80;
    server_name name-domain.vn;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name name-domain.vn;

    root /var/www/html/build;
    index index.html;

    # Đường dẫn tới file cert
    ssl_certificate /etc/ssl/certs/name-domain.vn.crt;       # hoặc fullchain.pem
    ssl_certificate_key /etc/ssl/private/name-domain.vn.key; # hoặc privkey.pem

    location / {
        try_files $uri /index.html;
    }

    location /api/ {
        proxy_pass http://<IP-SERVER>:8080/api/; # port BE
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# kiểm tra syntax
sudo nginx -t

# Reload nginx
sudo systemctl restart nginx
```

- hoặc

``` bash
server {
    listen 80;
    server_name duc-domain.vn www.duc-domain.vn;

    root /var/www/html/build;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    # location /api/ {
    #     proxy_pass http://127.0.0.1:8080/api/;
    #     proxy_http_version 1.1;
    #     proxy_set_header Host $host;
    #     proxy_set_header X-Real-IP $remote_addr;
    #     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #     proxy_set_header X-Forwarded-Proto $scheme;
    # }
}

```

# Cài đặt gitlab và shell host

``` bash
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
sudo apt-get install gitlab-ee=18.0.2-ee.0
nano /etc/gitlab/gitlab.rb

# update lại thành domain
external_url 'https://gitlab-ctl.greenglobal.com.vn'
# Tắt HTTPS nội bộ (vì SSL nằm ở proxy ngoài)
nginx['listen_https'] = false
nginx['listen_port'] = 80
		
# Bắt buộc GitLab tin tưởng proxy
nginx['proxy_set_headers'] = {
  "X-Forwarded-Proto" => "https",
  "X-Forwarded-Ssl" => "on"
}

# reload gitlab
sudo gitlab-ctl reconfigure

# xem password
cat /etc/gitlab/initial_root_password

# Truy cập domain của git
# username là root , password lấy trong initial_root_password 
```

- Nếu là domain tự host hãy dùng lệnh sau

- Nhớ add <ip-server> <tên-domain-git> ví dụ 10.1.0.13 git.vuduyduc.vn
# Đẩy dự án lên git

1. Tạo một repo trước trên gitlab vừa setup

``` bash
git config --global user.name "Duc"
git config --global user.email "vuduyduc550@gmail.com"
git clone http://git.duyduc.tech/foodweb1/foodweb.git
git checkout -b main
git add .
git commit -m "push(project-demo)"
git push --set-upstream origin main
```

# Cài đặt và sử dụng Docker

1. Setup thư mục

``` bash
mkdir tools
cd tools
mkdir docker
cd docker/
nano install-docker.sh
```

2. Cài đặt bằng shellscript

``` bash
#!/bin/bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce
sudo systemctl start docker
sudo systemctl enable docker
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker --version
docker-compose --version
```

3. Run file script

``` bash
chmod +x install-docker.sh
bash install-docker.sh
``` 

4. Thao tác với docker

``` bash 
# Kiểm tra phiên bản Docker
docker --version

# Kiểm tra trạng thái Docker
docker info

# Danh sách tất cả các lệnh Docker
docker --help

## Làm việc với Docker Images
# Liệt kê các Docker Images
docker images

# Tải một image từ Docker Hub
docker pull <image-name>:<tag>

# Xóa một image
docker rmi <image-id>

# Xây dựng image từ Dockerfile
docker build -t <image-name>:<tag> .

# Gắn thẻ (tag) cho một image
docker tag <image-id> <new-image-name>:<new-tag>

# Đẩy image lên Docker Hub
docker push <image-name>:<tag>

## Làm việc với Containers
# Liệt kê các container đang chạy
docker ps

# Liệt kê tất cả container (bao gồm cả container đã dừng)
docker ps -a

# Chạy một container
docker run --name=<container-name> -dp <port-container>:<port-application> <image-name>:<tag>

# Dừng một container
docker stop <container-id>

# Khởi động lại container
docker restart <container-id>

# Xóa một container
docker rm <container-id>

# Truy cập vào một container đang chạy
docker exec -it <container-id> /bin/bash

## Lệnh về Mạng (Network)
# Liệt kê các mạng (network)
docker network ls

# Tạo một mạng mới
docker network create <network-name>

# Kết nối một container vào mạng
docker network connect <network-name> <container-id>

# Ngắt kết nối một container khỏi mạng
docker network disconnect <network-name> <container-id>

## Xem Logs và Debug
# Xem logs của container
docker logs <container-id>

# Theo dõi logs theo thời gian thực
docker logs -f <container-id>

# Xem chi tiết thông tin của container
docker inspect <container-id>

# Kiểm tra hiệu suất container
docker stats

## Dọn dẹp hệ thống
# Xóa các container đã dừng
docker container prune

# Xóa các image không sử dụng
docker image prune

# Xóa toàn bộ tài nguyên không sử dụng (container, image, network, volume)
docker system prune -a

## Các lệnh thường dùng hàng ngày
# Kiểm tra container đang chạy
docker ps

# Kiểm tra các image có sẵn
docker images

# Chạy một container
docker run -d --name <container> <image>

# Dừng một container
docker stop <container>

# Xóa một container
docker rm <container>

# Tải một image
docker pull <image>

# Xây dựng một image
docker build -t <image-name>:<tag> .

# Xem logs của container
docker logs <container>

# Truy cập vào container
docker exec -it <container> /bin/bash

# Khởi động docker-compose
docker-compose up -d || docker-compose -f file.yml up -d 

# tắt bằng docker compose
docker-compose down || docker-compose -f file.yml down
```

# Dockerfile cho dự án

### Dockerfile Springboot

``` Dockerfile
# Build stage
FROM maven:3.9.6-eclipse-temurin-21 AS build

WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jdk

WORKDIR /run
COPY --from=build /app/target/ecom-proj-0.0.1-SNAPSHOT.jar /run/ecom-proj-0.0.1-SNAPSHOT.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/run/ecom-proj-0.0.1-SNAPSHOT.jar"]
```

### Dockerfile React

``` Dockerfile
FROM node:18.18-alpine as build

WORKDIR /app
COPY . .

RUN npm install --force
RUN npm run build

FROM nginx:alpine

COPY --from-build /app/dist /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### docker-compose

``` yml
version: '3.8'
services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    depends_on:
      - mysql
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/vegetfood
      SPRING_DATASOURCE_USERNAME: vegetfood
      SPRING_DATASOURCE_PASSWORD: vegetfood
    container_name: vegetfood-backend
    restart: always

  mysql:
    image: mysql:8.0
    volumes:
      - /db/mysql-1:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: vegetfood
      MYSQL_DATABASE: vegetfood
      MYSQL_USER: vegetfood
      MYSQL_PASSWORD: vegetfood
    ports:
      - "3307:3306"
    container_name: mysql-1
    restart: always
```

# Push Image lên registry (DockerHub)

``` bash
cd
docker login
# Nhập username Dockerhub
# Nhập Password Dockerhub

docker images
docker tag <id-image> vuduyduc/frontend:v1
docker push vuduyduc/frontend:v1
```

# Thiết lập registry

``` bash
wget https://github.com/goharbor/harbor/releases/download/v2.9.3/harbor-online-installer-v2.9.3.tgz
tar -xvzf harbor-online-installer-v2.9.3.tgz
cd harbor
cp harbor.yml.tmpl harbor.yml
nano harbor.yml
# hostname: 192.168.230.132 // sửa hostname thành IP hoặc domain 
# https related config
# https: // tắt https nếu không dùng domain và SSL
# https port for harbor, default is 443
# port: 443
# The path of cert and key files for nginx
# certificate: /your/certificate/path
# private_key: /your/private/key/path
harbor_admin_password: admin123 # đổi mật khẩu nếu cần

sudo nano /etc/docker/daemon.json
{
  "insecure-registries": ["192.168.230.132"]
}

./install.sh

docker login 192.168.230.132
docker images
# nhớ phải tạo project demo ở harbor_web trước khi push lên
docker tag registry:2 192.168.230.132/demo/myapp:latest 
docker push 192.168.230.132/demo/myapp:latest
```
