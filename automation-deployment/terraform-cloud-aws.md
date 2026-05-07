
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
