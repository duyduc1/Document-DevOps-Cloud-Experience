# IAM User Admin và MFA
### Accout IAM Admin - MFA
1. Chúng ta sẽ có một devide bên ngoài, device sẽ nhận và cho ra một số id và dựa vào số id đó để nhập vào yêu cầu của AWS mỗi lần đăng nhập
Tránh trường hợp hacker có thể đăng nhập vào sử dụng resource

2. Nhấn đăng nhập vào bảng điều khiển -> chọn Sign in using root user email (điền tài khoản gmail -> nhấn next và điền mật khẩu)

3. Search (IAM) chọn IAM mở với tab mới -> sẽ thấy Dashboard -> bên thanh Side Bar sẽ thấy list danh sách -> chọn Users -> Create User -> User name (điền vào ô input Admin_User_DevOps và tích chọn Provide user access to the AWS Management Console -> chọn i want to create an IAM user và tích chọn Users must create a new password at next sign-in-Recommended để không tạo lại password) -> Create

4. Set permissions trong Permissions options có thể chọn Add user to group và gán quyền AdministratorAccess sau này có thể tích bỏ AdministratorAccess cũng được hoặc có thể chọn thẳng Attach policies directly -> tích chọn AdministratorAccess -> Next

5. Review and create -> tích chọn Create User 

6. Retrieve password -> sẽ thấy phần user và password và đường link console(lấy id trong phần đường link này) -> qua tab khác đăng nhập bằng tài khoản và id vừa được tạo

7. Sau khi đăng nhập xong sẽ thấy Console Home -> nếu muốn bật xác thực thì chọn Security Credential -> chọn asign MFA -> trong Device name tự đặt (ví dụ AWS-lab-AdminuserDevOps) -> chọn Authenticator app -> Next (Có thể không cần cũng được)
-------------------------------------------------------------------------------------------------------------------

# Quản lý chi phí với AWS Budgets
1. Đăng nhập với User Root

2. Tích chọn vào Billing and Cost Management và sẽ thấy Tích chọn vào Billing and Cost Management home

3. Click vào Budgets bên thanh sidebar -> create budgets -> click chọn use a template -> kéo xuống Enter your budgeted amount($) ví dụ là 10 đô -> kéo xuống Email recipients (điền mail của mình vào) -> Create budget

# Tổng quan  các loại dịch vụ trên AWS

# Khởi tạo EC2 Ubuntu với VPC default
1. Gần thanh Header click chọn vào EC2 -> click chọn vào instances -> tích chọn Launch instances bên thanh sidebar

2. Name đặt là EC2_Ubuntu

3. Quick Start (click chọn Ubuntu)

4. Key pair name - required (đặt tên instance_Key_DevOps_On_AWS) -> Create key pair -> trong thư mục được tải về

5. Security group name - required (sg_for_devops_on_aws)

6. configure storage (1x tăng lên 18) 

7. click launch instance để tạo máy chủ

8. vào visual code kéo thư mục key đã được download vào thư mục visualcode -> sau đó ssh -i instance_Key_DevOps_On_AWS.pem ubuntu@ip của server (truy cập thành công vào server)

# Triển khai một VPC mới (là network là nơi tạo các resource bên trong)

1. Trên thanh header tích chọn VPC

2. Vào VPCs để xem sơ đồ của network

3. click chọn vào Create VPC gần thanh header

4. ở Resources to create (chọn VPC and more)

5. Sửa Auto-generate thành (devops)

6. Click Create VPC 

7. Sau khi cài xong nhấn vào (View VPC)

# Khởi tạo EC2 tên VPC mới

1. Chọn launch instance trong dashboard 

2. Name and tags (EC2_Ubuntu)

3. Quick Start (click chọn Ubuntu)

4. Key pair name - required (đặt tên instance_Key_DevOps_On_AWS) -> Create key pair

5. VPC - required (thay đổi thành devops-vpc bài trước đã tạo)

6. Subnet chọn devops-subnet-public2

7. Auto-assign public IP (chọn Enable)

8. Security group name - required (devops_sg)

9. configure storage (1x tăng lên 18) 

10. click launch instance để tạo máy chủ

# Hướng dẫn Connect tới Database và thao tác

1. Vào Databases bên thanh Sidebar của Dashboard -> sẽ thấy database devops-2 đã được tạo -> click vào 

* Chú ý Endpoint trong phần Endpoint & port

* Bên cạnh sẽ thấy Security -> chọn vào default (sg-07bb862d092e3b2c2) ở VPC security groups -> nếu chưa có hãy mở bằng Edit inbound rules

2. SSH vào server EC2 // ssh -i tới thư mục chứac key ubuntu@ip

3. sudo apt update

4. sudo apt install postgresql postgresql-contrib

5. sudo systemctl start postgresql.service

6. PGPASSWORD=vuduyduc psql -h <endpoint> -p 5432 -U postgres -d postgres

## Vào Security Group đang gắn → Thêm Inbound Rule:

1. sudo apt update

2. sudo apt install mysql-client -y

3. mysql -h <RDS-endpoint> -u admin -p

# Làm quen với dịch vụ lưu trữ S3

1. Vào Dashboard Console Home -> nhấn vào S3 

2. Sau khi chuyển tới dashboard S3 

- nhấn vào create bucket

- Bucket name (đặt là lab2.12-devops) 

- tất cả còn lại để mặc định 
- Createa bucket

3. Sau khi tạo xong vào lại 
- lab2.12-devops 
- Create folder 
- đặt tên Folder(data) 
- Create Folder 
- sau khi tạo xong sẽ xuất hiện folder data 
- click upload 
- click vào Add files 
- chọn file 
- upload
- vào ảnh vừa upload sẽ thấy Object URL sẽ thấy đường link ảnh

4. vào lại lab2.12-devops 
- sẽ hấy Permissions 
- ở phần Block public access 
- nhấn Edit 
- bỏ tích chọn Block all public access 
- save changes 
- kéo xuống tìm Accees control list (ACL) chọn edit
- chọn ACLs enabled 
- save changes

5. Vào lại lab2.12-devops 
- vào thư mục data 
- tích chọn ảnh 
- chọn Actions phía trên (chọn Make public using ACL) 
- make public 
- truy cập lại URL ảnh

# Dùng git làm việc với source 

``` bash
git config --list
git clone <repository-url>  
git status                  # Kiểm tra trạng thái
git add .                   # Thêm file vào staging
git commit -m "Message"     # Commit thay đổi
git pull origin <branch>    # Lấy code mới nhất từ remote
git push origin <branch>    # Đẩy code lên remote
git checkout <branch-name>  # Chuyển sang nhánh 
git checkout -b <branch>    # Tạo và chuyển sang nhánh mới
git merge <branch>          # Hợp nhánh
git fetch                   # Lấy thông tin từ remote (không merge)
git log                     # Check log 
git branch -d <branch-name> # Xóa nhánh đã merge
git push origin --delete develop # push thay doi 
git branch -D <branch-name> # Xóa nhánh chưa merge
```

# Các thao tác cơ bản với Docker

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
docker run -d --name <container-name> <image-name>:<tag>

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
```

# Build images với Dockerfile

### Setup environment
``` bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
apt-get install nodejs -y
sudo npm install -g npm@latest
npm i -g @nestjs/cli
nest new nest-docker
cd nest-docker
nano Dockerfile
```

### Dockerfile
``` bash
# Use the official Node.js image as the base image
FROM node:20
	
# Set the working directory inside the container
WORKDIR /usr/src/app
	
# Copy package.json and package-lock.json to the working directory
COPY package*.json ./
	
# Install the application dependencies
RUN npm install
	
# Copy the rest of the application files
COPY . .
	
# Build the NestJS application
RUN npm run build
	
# Expose the application port
EXPOSE 3000
	
# Command to run the application
CMD ["node", "dist/main"]
```

### run docker container
``` bash
docker build -t nest-docker:v1 .
docker run -d -p 3000:3000 nest-docker 

# push lên dockerhub
docker login
docker tag nest-docker vuduyduc764/nest-docker
docker images
docker push vuduyduc764/nest-docker
```

# Quản lý container với docker-compose

- nano docker-compose.yml

``` bash 
version: '3.8'

services:
  app:
    build: .
    container_name: nest-docker
    ports:
      - "3000:3000"
    volumes:
      - /storage:/app
    environment:
      - NODE_ENV=development
      - DATABAE_URL=0
    depends_on:
      - db

  db:
    image: mongo:6
    container_name: mongodb
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
	
volumes:
  mongo_data:
```

### run docker-compose and down docker-compose
``` bash
docker-compose -f docker-compose.yaml up -d
docker-compose down
```

# Setup Terraform

### install terraform và hashicorp

``` bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common

wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt-get install terraform
```

### install aws 

``` bash 
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### configure aws

``` bash
aws configure

### điền thông tin key
Access Key ID: <you-access-key-id>
Secret Access Key: <your-secret-key-access>
region name: <your-region>
output format: json
```

# Làm quen với Terraform

### tạo thư mục làm việc 

``` bash
mkdir terraform
cd terraform
```

### Các câu lệnh để kiểm tra vpc, subnet
``` bash
aws ec2 describe-security-groups --query "SecurityGroups[*].[GroupId,GroupName]" --output table
aws ec2 describe-subnets --query "Subnets[*].[SubnetId,AvailabilityZone]" --output table
```

### Cấu trúc file bên trong

``` bash
nano variables.tf

# nội dung bên trong
variable "region" {
  description = "Region"
  type        = string
  default     = "ap-southeast-1"
}

variable "ami" {
  description = "ami"
  type        = string
  default     = "ami-0474ac020852b87a9"
}

variable "instance_type" {
  description = "instance_type"
  type        = string
  default     = "t2.micro"
}
```

``` bash
nano version.tf

### nội dung bên trong
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "5.86.0"
    }
  }
	
  required_version = ">= 1.0"
}

provider "aws" {
  region  = var.region
}
```

``` bash
nano main.tf

### nội dung bên trong
resource "aws_instance" "app_server" {
  ami           = var.ami
  instance_type = var.instance_type

  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```

``` bash
nano outputs.tf

# nội dung bên trong
output "public_ip" {
  value = aws_instance.app_server.public_ip
}
```

### các câu lệnh chạy

``` bash
terraform plan
terraform init
terraform apply --auto-approve

# dùng để destroy
terraform destroy --auto-approve
```

# set up ECR chứa các docker images 

### install docker

``` bash
mkdir tools 
cd tools 
mkdir install-docker
cd install-docker
nano install-docker.sh
```

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

``` bash
chmod +x install-docker.sh
bash install-docker.sh
```

### Setup repo docker

1. Tạo ECR Repository trên AWS

2. vafo AWS Console -> tìm ECR

3. Chọn create repostiory
* Đặt tên repo (ví dụ: my-service).
* Chọn Private (thường dùng).
* Các tuỳ chọn khác để mặc định.

4. Sau khi tạo xong, sẽ có 1 repo dạng <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-service

### install aws 

``` bash 
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

### configure aws

``` bash
aws configure

### điền thông tin key
Access Key ID: <you-access-key-id>
Secret Access Key: <your-secret-key-access>
region name: <your-region>
output format: json
```

### Đăng nhập từ ECR từ Docker
- aws ecr get-login-password --region us-east-1 \
| docker login --username AWS --password-stdin 919446726176.dkr.ecr.us-east-1.amazonaws.com

### build 1 docker images

- Phải có Dockerfile cho service 

``` bash
cd backend-folder
docker build -t backend-service . 

# rename docker images vừa build xong 
docker tag backend-service:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-service:latest

# push lên ECR 
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-service:latest
```

# Deploy ứng dụng với ecs

1. Tạo Cluster ECS

* Vào AWS Console → ECS → Clusters → Create Cluster.

* Đặt tên cluster (vd: nestjs-cluster).

2. Tạo Task Definition

* Vào ECS → Task Definitions → Create new Task Definition

* Đặt tên task (vd: nestjs-task)

* Thêm Container definition
- Name: nestjs-service
- Image: <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-service:latest
- Port mappings: 3000 → 3000 (hoặc 80 → 3000 nếu muốn expose HTTP)

* Chọn CPU/RAM (vd: 0.5 vCPU, 1GB RAM)

3. Tạo Service để chạy Task

* Vào ECS → Clusters → Chọn cluster → Create service.

* Task Definition: chọn cái vừa tạo (nestjs-task)

* Number of tasks: ví dụ 1.

* Networking:
- Chọn VPC.
- Chọn Subnet.
- Security group: mở port 3000 (hoặc 80).

4. Triển khai

* Nhấn Create service → ECS sẽ tự pull image từ ECR và chạy container.

* Bạn có thể kiểm tra trong ECS → Cluster → Tasks.


# Deploy FE best practive
1. Build project Vue
   
2. Tạo s3 Bucket
- Create bucket
- Đặt tên (ví dụ: frontend-vue-app-duc)
- Bỏ tick "Block all public access"
- Create bucket

3. Enable Static Website Hosting
* Trong bucket
- Vào tab Properties
- Tìm Static website hosting
- Enable và cấu hình:

``` bash
Index document: index.html
Error document: index.html
```
4. Upload file build
* Vào tab Objects → Upload
- Upload toàn bộ file trong dist/ (kéo thả tất cả các file folder vào)
- Nhớ upload file bên trong, không upload cả folder dist

5. Set quyền public
- Vào tab Permissions → Bucket policy:

``` bash
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::frontend-vue-app-duc/*"]
    }
  ]
}
```

6. Lấy URL truy cập:
- lấy domain ở Bucket website endpoint bên trong tab Properties

7. Tạo CloudFront Distribution
- Vào CloudFront → Create distribution
- Origin domain (dán domain đã lấy ở bước 6 paste vào s3 origin)

8. Trỏ domain router53 vào cloudFront nếu muốn
- ở step 1 của bước Create distribution có bước add router 53 có thể add ở đây
- hoặc có thể vào router53
- AWS Console → Route53
- Chọn: Registered domains
- Bấm: Register domain
- Nhập tên domain
- Thanh toán
- Xác nhận email

9. Tạo SSL Certificate (sau khi tạo domain ở router53)
- Tạo certificate
- Chọn:

``` bash
Request certificate → Public certificate
```

- Nhập domain:

``` bash
yourdomain.com (domain vừa tạo ở Router53)
```
10. Gắn domain vào CloudFront
- Vào CloudFront → Distribution → Edi
* Thêm Alternate domain name (CNAME)
``` bash
app.yourdomain.com (domain của router53)
```

11. Trỏ domain bằng Route53
- Hosted Zones → chọn domain của bạn
- Tạo record
- Record type:
``` bash
A – IPv4 address
```
- Bật:
``` bash
Alias: ON
```

- Alias target:
* Chọn:
``` bash
CloudFront distribution
```

# Deploy be service với aws lambda (express)
1. Create Function

2. Function name (đặt tên cho function)
- Runtime (chọn môi trường chạy ví dụ nodejs24.x)
- Create function
  
3. ở tab Code
- Chọn Upload from
- upload source đã nén thành file zip lên

4. nhấn Test() hoặc Deploy()

5. Trỏ API Gateway

5.1 Create API

5.2 Choose an API type
- Chọn HTTP API (nhấn Build)
- Điền  API Name
- ở Integrations nhấn Add integration
- Chọn lamba
- ở Lambda function chọn lambda vừa deploy (nhấn Next)

5.3 Configure routes
- Nhấn Add route : Chọn Method (ví dụ: Get)
- Resource path : gõ path của API đó (ví dụ /users)
- Intergration target: chọn thư mục deploy lambda (ví dụ: test)
- Next

5.4 Define stages 
- Để mặc định auto deploy
- Next

5.5 Review and create
- Create

6. ở tab develop ở thanh sidebar

6.1 Configure

``` bash
Access-Control-Allow-Origin: * (hoặc có thể thay thế bằng domain fe)
Access-Control-Allow-Headers: Content-Type,X-Amz-Date,Authorization,X-Api-Key
Access-Control-Allow-Methods: GET,POST,PUT,DELETE,OPTIONS
```
- nhớ nhấn nút add ở mỗi row
- Sau đó Save

# Deploy BE Best practive với VPC

## Setup VPC

### Layer Public Subnet

1. Triển khai VPC:

1.1 Tạo VPC (Create VPC)

- Điền (Name tag - optional) : VPC-Lab

- Điền (IPv4 CIDR) : 10.0.0.0/16

- Create VPC

1.2 Tạo Internet Gateway (Create internet gateway)

- Đặt (Name tag) : IGW-Lab

- Trên góc bên phải chọn (Action) -> Attach to VPC (VPC-Lab)

1.3 Tạo 2 Nat-Gateway (Create NAT gateway)

1.3.1 Tạo Nat-1

- Đặt (Name - optional) : Nat-Lab-1

- chọn (VPC) : VPC-Lab

- Create NAT gateway

1.3.2 Tạo Nat-2

- Đặt (Name - optional) : Nat-Lab-2

- chọn (VPC) : VPC-Lab

- Create NAT gateway

1.4 Tạo Subnets cho 2 Nat 

1.4.1 Tạo Subnet cho Nat-1 (Create subnet)

- chọn VPC (VPC-Lab)

- Điền (Subnet name): Nat-Subnet-1

- chọn (Availability Zone) : us-east-1a (tuỳ theo vùng và propocal triển khai)

- Điền (IPv4 subnet CIDR block) : 10.0.0.0/24

- Create subnet

1.4.2 Tạo Subnet cho Nat-2 (Create subnet)

- chọn VPC (VPC-Lab)

- Điền (Subnet name): Nat-Subnet-2

- chọn (Availability Zone) : us-east-1b (tuỳ theo vùng và propocal triển khai)

- Điền (IPv4 subnet CIDR block) : 10.0.1.0/24

- Create subnet

1.5 Tạo route tables cho Internet Gateway

- Create route table

- Đặt (Name - optional) : IGW-Route-Table

- chọn (VPC) : VPC-Lab

- Create route table

- Chọn (Edit routes)

- Chọn (Add route) : ở Destination chọn (0.0.0.0/0) ở Target chọn (Internet Gateway và chọn (IGW-Lab) đã tạo ở bước trên)

- Save changes

1.6 Attach 2 Subnet của Nat vào Routes của Internet Gateway 

- Vào (Route tables) của Internet Gateway (IGW-Route-Table)

- ở tab (Subnet associations)

- Chọn (Edit subnet associations)

- ở (Available subnets) gắn Subnet của 2 Nat vào (Nat-Subnet-1, Nat-Subnet-2)

- Save associations

1.7 kiểm tra luồng 

- vào (VPC-Lab) sẽ thấy 1 biểu đồ 

- ở tab (Resource map) khi xem Subnet của 2 Nat sẽ thấy thông tới IGW (chứng tỏ Nat đã có internet)

### Layer Private Subnet (Application - Database)

1.8 Tạo 2 Subnet cho App:

1.8.1 Tạo Subnet cho App-Subnets-1 (Create subnet)

- chọn VPC (VPC-Lab)

- Điền (Subnet name): App-Subnet-1

- chọn (Availability Zone) : us-east-1a (tuỳ theo vùng và propocal triển khai)

- Điền (IPv4 subnet CIDR block) : 10.0.10.0/22

- Create subnet

1.8.2 Tạo Subnet cho App-Subnet-2 (Create subnet)

- chọn VPC (VPC-Lab)

- Điền (Subnet name): App-Subnet-2

- chọn (Availability Zone) : us-east-1b (tuỳ theo vùng và propocal triển khai)

- Điền (IPv4 subnet CIDR block) : 10.0.14.0/22

- Create subnet

1.8.3 Tạo Route Table cho App-Subnet-1 (Create route table) và Attach App-Subnet-1 vào Route table

- Đặt (Name - optional) : App-1-Route-Table

- chọn (VPC) : VPC-Lab

- Create route table

- Chọn (Edit routes)

- Chọn (Add route) : ở Destination chọn (0.0.0.0/0) ở Target chọn (Nat Gateway và ở dưới chọn (Nat-1) đã tạo ở bước trên)

- Save changes

- ở tab (Subnet associations)

- Edit subnet associations

- ở (Available subnets) chọn App-Subnet-1

- Save associations

1.8.4 Tạo Route Table cho App-Subnet-2 (Create route table) và Attach App-Subnet-2 vào Route table

- Đặt (Name - optional) : App-2-Route-Table

- chọn (VPC) : VPC-Lab

- Create route table

- Chọn (Edit routes)

- Chọn (Add route) : ở Destination chọn (0.0.0.0/0) ở Target chọn (Nat Gateway và ở dưới chọn (Nat-2) đã tạo ở bước trên)

- Save changes

- ở tab (Subnet associations)

- Edit subnet associations

- ở (Available subnets) chọn App-Subnet-2

- Save associations

1.9 Tạo 2 subnet cho DB ở 2 vùng:

1.9.1 Tạo Subnet cho DB-Subnets-1 (Create subnet)

- chọn VPC (VPC-Lab)

- Điền (Subnet name): DB-Subnet-1

- chọn (Availability Zone) : us-east-1a (tuỳ theo vùng và propocal triển khai)

- Điền (IPv4 subnet CIDR block) : 10.0.2.0/24

- Create subnet

1.9.2 Tạo Subnet cho DB-Subnet-2 (Create subnet)

- chọn VPC (VPC-Lab)

- Điền (Subnet name): DB-Subnet-2

- chọn (Availability Zone) : us-east-1b (tuỳ theo vùng và propocal triển khai)

- Điền (IPv4 subnet CIDR block) : 10.0.3.0/24

- Create subnet

1.9.3 Tạo Route Table 2 Subnet của DB và Attach cả 2 subnet vào Route table (DB-Subnet-1, DB-Subnet-2)

- Create route table

- Đặt (Name - optional) : DB-Route-Table

- chọn (VPC) : VPC-Lab

- Create route table

- ở tab (Subnet associations)

- Edit subnet associations

- ở (Available subnets) chọn (DB-Subnet-1 và DB-Subnet-2)

- Save associations

* Chú thích Database không cần phải có Internet 

1.10 Tạo Endpoints (Dành cho các máy EC2 không có public IP)

- Create endpoint

- Name tag - optional : (EndPoint-Lab)

- Type: chọn EC2 Instance Connect Endpoint

- VPC : Chọn (VPC-Lab)

- Subnet : Chọn subnet triển khai Application (App-Subnet-1 hoặc App-Subnet-2)

- Create endpoint

## Aurora and RDS

### Tạo DB MySQL RDS với DB Subnet 

1. Tạo (Subnet groups)

- Create DB subnet group

- Name (đặt: DB-Subnet)

- Description (đặt: Description hoặc tuỳ mô tả dự án)

- VPC (chọn: VPC-Lab)

- chọn (Availability Zones: us-east-1a và us-east-1b)

- chọn (Subnets: DB-Subnet-1 và DB-Subnet-2)

- Create

2. Tạo Database (cụm master)

- chọn Create database (Full configuration)

- Engine options (chọn Database MySQL có thể chọn các DB khác tuỳ dự án)

- Choose a database creation method (chọn Full configuration)

- DB instance identifier (đặt cụm DB: Database-lab)

- Master username (đặt tên đăng nhập DB: ví dụ root)

- Master password (tự đặt)

- Confirm master password (xác nhận mật khẩu vừa đặt)

- Virtual private cloud (chọn VPC-Lab)

- Availability Zone (chọn us-east-1) // chú thích theo desgin thì cụm master sẽ ở vùng AZ1

- Create Database

3. Tạo Database (cụm replicas)

- click chọn cụm (database-lab)

- click Actions (chọn Create read replica)

- DB instance identifier (đặt: database-lab-repicas)

- Availability Zone (chọn us-east-1b)

- Create read replica

## Application (ALB)

### Tạo Domain cho Application (Ở service EC2)

1. Tạo target group

- Create target group

- Target type (chọn Ip addresses vì triển khai App trên ECS)

- Target group name (đặt: tg-items hoặc tuỳ theo dự án)

- Protocol (HTTP)

- Port (3000 port của service khi chạy)

- VPC (chọn: VPC-Lab)

- Health check path (trỏ tới API mà nó luôn trả về response 200 ví dụ /api/v1/items/product)

- Next qua bước 2

- Create target group

2. Tạo Load Balancers

- Create load balancer

- Load balancer types (chọn : Application Load Balancer (ALB))

- Create

- Load balancer name (đặt : domain-alb hoặc tuỳ theo yêu cầu dự án)

- VPC (chọn VPC-Lab)

- Availability Zones and subnets (chọn cả 2 vùng us-east-1a và us-east-1b)

- Subnet chọn (Nat-subnet-1 và Nat-subnet-2)

- Default action (chọn: Return fixed response)

- Response body - optional (đặt: Not Found) khi không trỏ đúng API sẽ hiển thị Not found

- Create load balancer

3. Add rule

- Sau khi tạo xong ALB ở tab (Listeners and rules (1))

- tích chọn ALB vừa tạo

- chọn Manage rules (Add rule)

- tại Conditions (0 values)

- Add condition (chọn Path)

- điền path API (ví dụ: /api/v1/items/*)

- Target group (chọn target group vừa tạo (tg-items))

- Next

- Priority (đặt số: ví dụ 1)

- Next

- Add rule

4. Security Group

- Ở tab Security

- kick chọn Security Group (ví dụ: sg-0224501b84cc86f20)

- Chọn (Edit inbound rules)

- Add rule

- Type (HTTP)

- tích chọn 0.0.0.0/0

- Save changes

## ECR , ECS, EC2

### Triển khai dự án với EC2

### EC2

- Vào service EC2

- instances (Launch instances)

- Name (có thể đặt hoặc không ví dụ: Deployment-App)

- Quick Start (chọn Ubuntu hoặc tuỳ distro)

- Instance type (chọn type máy phù hợp ví dụ (c71-flex.large))

- Key pair name - required (chọn key đã tạo trước đó hoặc Create new key pair)

- Network settings (chọn Edit)

- VPC: (chọn : VPC-Lab)

- Subnet (chọn Subnet sẽ triển khai (App-subnet-1 hoặc App-subnet-2 miễn 1 trong 2 Subnet này có Endpoint đã tạo trong VPC))

- Auto-assign public IP (disable IP) chặn IP public

- Add security group rule

- Type: Custom TCP

- Port range: 3000 (Port Application)

- Source: (sg-0224501b84cc86f20) chỉ cho phép Security group của ALB được phép truy cập port 3000 này

- Launch Instances

- Vào instance vừa tạo (Deployment-App)

- chọn Connect

- Connection type (chọn : Connect using a Private IP)

- Connect

- Ở tab Security của EC2 Instance (copy SG của EC2 ví dụ : sg-0a99bb977084ec4af)
 
- Tiếp theo vào RDS
 
- chọn database-lab
 
- ở tab (Connectivity & security) kéo xuống (Security group rules)
 
- click vào Security-group (default (sg-0224501b84cc86f20))
 
- ở Security Group click vào (sg-0224501b84cc86f20)
 
- Edit inbound rules
 
- Add rule
 
- Type (Chọn Custom TCP) , Port range: (3306) và Source add (sg-0a99bb977084ec4af) SG của EC2 // Chú thích chỉ cho những instance hoặc container trong SG này truy cập DB
 
- Trong instance EC2
 
- Clone source code
 
## Cài đặt Docker
 
- nano install-docker.sh
 
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
 
- chmod +x install-docker.sh
 
- bash install-docker.sh
 
## Cài đặt và Setup Database MySQL
 
- sudo apt install mysql-server -y
 
- Vào RDS database (database-lab)
 
- ở Tab Connectivity & security
 
- copy lệnh (ví dụ: mysql -h database-lab.cyf6wcuq0j47.us-east-1.rds.amazonaws.com -P 3306 -u root -p)
 
- Paste vào instance ec2 : mysql -h database-lab.cyf6wcuq0j47.us-east-1.rds.amazonaws.com -P 3306 -u root -p
 
- Điền mật khẩu
 
``` sql
create database testdb;
show databases;
```
 
- Edit file .env hoặc các file có chứa config Database
 
- thay đổi thông tin bên trong file confi
 
``` env
RDS_ENDPOINT=database-lab.cyf6wcuq0j47.us-east-1.rds.amazonaws.com
RDS_PORT=3306
RDS_USERNAME=root
RDS_PASSWORD=M1aT4Tt7aeXc
RDS_DB_NAME=testdb
```
 
## Chạy chương trình với Docker
 
* Dùng các lệnh Docker để build Images
 
``` bash
docker-compose up -d --build
docker ps -a
docker images
```
## Cài đặt AWS
 
- nano install-aws.sh
 
``` bash
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```
 
### configure aws
 
- Security credentials (nhấn vào name trên góc phải màn hình)
 
- Tìm Create access key (nhấn Create access key) nhớ phải lưu thông tin access key
 
* Trong instance EC2
 
- aws configure
 
``` bash
aws configure
 
### điền thông tin key
Access Key ID: <you-access-key-id>
Secret Access Key: <your-secret-key-access>
region name: <your-region>
output format: json
```
 
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
 
## Setup ECR
 
- Create a repository (Craete)
 
- ở Repository name (đặt tên Repo: item-task)
 
- Create
 
- Copy lại Repository name URI (có dạng: 919446726176.dkr.ecr.us-east-1.amazonaws.com/items-task)
 
* Trong instance EC2
 
- Sau khi build xong images
 
``` bash
docker images
docker tag nest-item-api:latest 919446726176.dkr.ecr.us-east-1.amazonaws.com/items-task:latest
docker push 919446726176.dkr.ecr.us-east-1.amazonaws.com/items-task:latest
```
 
* Check lại trong Repository name của ECR
 
- Check trong items-task đã có images chưa nếu có sẽ thấy Image tags (latest)
 
- Vào ECR Repository Name (item-task)
 
- Click chọn tag latest copy URI (919446726176.dkr.ecr.us-east-1.amazonaws.com/items-task:latest)
 
## Deploy Setup ECS
 
### Tạo Cluster
 
- Vào Amazon Elastic Container Service
 
- Cluster (Create Cluster)
 
- Cluster name (ví dụ: Lab-Cluster)
 
- Create
 
### Tạo Task definitions
 
- Create new task definition
 
- Task definition family (ví dụ: items-task)
 
- Task role (chọn: ecsTaskExcutionRole)
 
- Name (đặt tên container: item-container)
 
* Trong ECS
 
- Image URI (dán : 919446726176.dkr.ecr.us-east-1.amazonaws.com/items-task:latest)
 
- Container port (port của container: 3000)
 
- Port name (ví dụ: port-item)
 
- CPU, Memory hard limit, Memory soft limit (để mặc định hoặc tùy chỉnh theo dự án)
 
- Add environment variable (phải add đủ thông tin ENV trong file env hoặc file config)
 
- điền Key và Value
 
``` bash
Key: RDS_ENDPOINT | Value: database-lab.cyf6wcuq0j47.us-east-1.rds.amazonaws.com
Key: RDS_PORT | Value: 3306
Key: RDS_USERNAME | Value: root
Key: RDS_PASSWORD | Value: M1aT4Tt7aeXc
Key: RDS_DB_NAME | Value: testdb
```

- Create
 
* Quay lại cụm cluster vừa tạo (Lab-Cluster)
 
- ở tab Services
 
- chọn Create
 
- Task definition family (chọn task definitions vừa tạo: items-task)
 
* Kéo xuống Networking
 
- VPC : (chọn VPC-Lab)
 
- Subnets : chỉ đề lại 2 Subnets (App-Subnet-1 và App-Subnet-2) theo chuẩn design task sẽ run ở 2 AZ này
 
- Security group name (chọn Security group trùng với EC2 Instance đã tạo ở trên (sg-0a99bb977084ec4af))
 
* Kéo xuống Load balancing
 
- tích chọn Use load balancing
 
- Application Load Balancer (tích chọn Use an existing load balancer)
 
- Load balancer (chọn ALB đã tạo trước đó : doamin-alb)
 
- Listener (tích chọn Use an existing listener)
 
- ở Listener chọn (HTTP:80)
 
- Target group (tích chọn Use an existing target group)
 
- Target group name (chọn tg-items đã tạo trước đó trong target group)
 
- Create
