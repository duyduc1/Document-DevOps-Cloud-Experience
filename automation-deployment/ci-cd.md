
# Setup gitlab-runner và triển khai CI/CD

1. triển khai gitlab-runner

``` bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | bash
apt-get install gitlab-runner
gitlab-runner register

# Enter the gitlab instance URL ( - bỏ link của gitlab vào trong này)
# Enter the registration Token ( - Vào CI/CD -> gitlab-runner -> lấy token vào dán vào)
# Enter the description for the runner ( - có thể bỏ tên của server vào ví dụ mô tả là deploy )
# Enter the tags for the runner ( - có thể bỏ tên của server vào ví dụ tags là deploy )
# Enter option maintenance note for the runner ( - gõ shell ) 
nano /etc/gitlab-runner/config.toml 
nohup gitlab-runner run --working-directory /home/gitlab-runner --config /etc/gitlab-runner/config.toml --service gitlab-runner --user gitlab-runner 2>&1 &
ps -ef| grep gitlab-runner
```

2. Cấp quyền cho gitlab-runner không sử dụng password

``` bash
visudo
gitlab-runner ALL=(ALL) NOPASSWD: ALL
gitlab-runner ALL=(ALL) NOPASSWD: /bin/mkdir*
gitlab-runner ALL=(ALL) NOPASSWD: /bin/mv*
gitlab-runner ALL=(ALL) NOPASSWD: /bin/cp*
gitlab-runner ALL=(ALL) NOPASSWD: /bin/chown*
gitlab-runner ALL=(ALL) NOPASSWD: /bin/rm*
gitlab-runner ALL=(ALL) NOPASSWD: /sbin/runuser
gitlab-runner ALL=(ALL) NOPASSWD: /usr/sbin/nginx*
gitlab-runner ALL=(ALL) NOPASSWD: /bin/su backend*
```

3. Setup key env cho project nếu không muốn public keyy trong repo

- Vào dự án Gitlab 

- Truy cập: Settings → CI/CD → Variables

- Bấm "Add variable"

- Nhập: 

> Key: ENV_CONTENT

> Value:

``` env
DB_HOST=192.168.230.140
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=postgres
DB_DATABASE=mydb
```

4. Tạo file (.gitlab-ci.yml)

## Springboot

``` yml
stages:
  - build
  - deploy
  - showlog

variables:
  DEPLOY_DIR: /opt/backend-data
  DATA_LOG: /var/log/java
  JAR_NAME: <file-build-sau-khi-build-trong-du-an-springboot>.jar
  JAR_PATH: target/<file-build-sau-khi-build-trong-du-an-springboot>.jar
  PORT: 8080

build:
  stage: build
  variables:
    GIT_STRATEGY: clone
  script:
    - mvn clean install
  tags:
    - deploy

deploy:
  stage: deploy
  variables:
    GIT_STRATEGY: none
  script:
    - sudo mkdir -p $DATA_LOG
    - sudo chown -R gitlab-runner:gitlab-runner $DATA_LOG
    - sudo fuser -k $PORT/tcp || true
    - sudo mkdir -p $DEPLOY_DIR
    - sudo cp $JAR_PATH $DEPLOY_DIR
    - sudo chown -R gitlab-runner:gitlab-runner $DEPLOY_DIR
    - cd $DEPLOY_DIR
    - nohup java -jar $JAR_NAME > $DATA_LOG/backend.log 2>&1 &
  tags:
    - deploy

showlog:
  stage: showlog
  variables:
    GIT_STRATEGY: none
  script:
    - echo "Showing latest log lines..."
    - cat $DATA_LOG/backend.log
  tags:
    - deploy
```

## React

``` yml
stages:
  - build
  - deploy
  - showlog

build:
  stage: build
  variables:
    GIT_STRATEGY: clone
  script:
    - npm install
    - npm run build || true
  artifacts:
    paths:
      - build/
  tags:
    - deploy

deploy:
  stage: deploy
  variables:
    GIT_STRATEGY: none
  script:
    - sudo rm -rf /var/www/html/*
    - sudo cp -r build/* /var/www/html/
    - sudo chown -R www-data:www-data /var/www/html/
    - sudo nginx -t
    - sudo nginx -s reload
  tags:
    - deploy

showlog:
  stage: showlog
  variables:
    GIT_STRATEGY: none
  script:
    - echo "Nginx Access Log:"
    - sudo tail -n 100 /var/log/nginx/access.log || true
    - echo "Nginx Error Log:"
    - sudo tail -n 100 /var/log/nginx/error.log || true
  tags:
    - deploy
```

## Nest

``` yml
stages:
  - build
  - deploy
  - showlog

variables:
  DEPLOY_DIR: /opt/backend-data
  APP_NAME: main.js
  DIST_DIR: dist
  PORT: 3000

build:
  stage: build
  variables:
    GIT_STRATEGY: clone
  script:
    - npm install
    - npm run build 
  tags:
    - deploy

deploy:
  stage: deploy
  variables:
    GIT_STRATEGY: none
  script:
    - sudo fuser -k $PORT/tcp || true
    - sudo mkdir -p $DEPLOY_DIR
    - sudo cp -r $DIST_DIR/* $DEPLOY_DIR/
    - sudo cp package*.json $DEPLOY_DIR/
    # Tạo file .env từ GitLab CI variable
    - echo "$ENV_CONTENT" > .env
    - sudo cp .env $DEPLOY_DIR/  
    - sudo chown -R gitlab-runner:gitlab-runner $DEPLOY_DIR
    - cd $DEPLOY_DIR
    - export $(cat .env | xargs)
    - npm install --omit=dev
    - nohup node $APP_NAME > nohup.out 2>&1 &
  tags:
    - deploy

showlog:
  stage: showlog
  variables:
    GIT_STRATEGY: none
  script:
    - tail -n 100 $DEPLOY_DIR/nohup.out
  tags:
    - deploy
```

## Next

``` yml
stages:
  - build
  - deploy
  - showlog

variables:
  DEPLOY_DIR: /opt/frontend-data
  PORT: 4000

build:
  stage: build
  variables:
    GIT_STRATEGY: clone
  script:
    - npm install
    - npm run build      
  artifacts:
    paths:
      - .next
      - public
      - package*.json
  tags:
    - deploy

deploy:
  stage: deploy
  variables:
    GIT_STRATEGY: none
  script:
    - sudo fuser -k $PORT/tcp || true
    - sudo mkdir -p $DEPLOY_DIR
    - sudo cp -r .next public package*.json $DEPLOY_DIR/
    # Copy file .env từ biến CI
    - echo "$ENV_CONTENT" > .env
    - sudo cp .env $DEPLOY_DIR/
    - sudo chown -R gitlab-runner:gitlab-runner $DEPLOY_DIR
    - cd $DEPLOY_DIR
    - export $(cat .env | xargs)
    - npm install --omit=dev
    - PORT=$PORT nohup npm run start > nohup.out 2>&1 &
  tags:
    - deploy

showlog:
  stage: showlog
  variables:
    GIT_STRATEGY: none
  script:
    - sleep 20
    - tail -n 1000 $DEPLOY_DIR/nohup.out
  tags:
    - deploy
```


# Docker ci/cd

1. Chuyển gitlab-runner vào môi trường docker

``` bash
usermod -aG docker gitlab-runner
su gitlab-runner
docker login
```

2. Tiến hành setting env

- Vào setting -> CI/CD -> variables

- Điền vào thông tin

* REGISTRY_URL (điền ở phần key) : docker.io (điền ở phần value) // nhớ bỏ đi tích chọn protect variables

* REGISTRY_PROJECT (điền ở phần key) : vuduyduc764 (điền ở phần value) // nhớ bỏ đi tích chọn protect variables

* REGISTRY_USER (điền ở phần key) : vuduyduc764 (điền ở phần value) // nhớ bỏ đi tích chọn protect variables
	  
* REGISTRY_PASSWORD (điền ở phần key) : Vuduyduc28042002@ (điền ở phần value) // nhớ bỏ đi tích chọn protect variables 

3. Setup file .gitlab-ci.yml

### Dựng Docker CI/CD Dockerfile push lên dockerhub

``` yaml
variables:
   DOCKER_IMAGE: ${REGISTRY_URL}/${REGISTRY_PROJECT}/${CI_PROJECT_NAME}:${CI_COMMIT_TAG}_${CI_COMMIT_SHORT_SHA}
   DOCKER_CONTAINER: ${CI_PROJECT_NAME}
stages:
    - buildandpush
    - deploy
    - showlog

build:
    stage: buildandpush
    before_script:
        - echo "$REGISTRY_PASSWORD" | docker login -u "$REGISTRY_USER" --password-stdin
    variables:
        GIT_STRATEGY: clone
    script:
        - docker build -t $DOCKER_IMAGE .
        - docker push $DOCKER_IMAGE
    tags:
        - deploy
deploy:
    stage: deploy
    before_script:
        - echo "$REGISTRY_PASSWORD" | docker login -u "$REGISTRY_USER" --password-stdin
    variables:
        GIT_STRATEGY: none
    script:
        - docker pull $DOCKER_IMAGE
        - docker rm -f $DOCKER_CONTAINER
        - docker run --name $DOCKER_CONTAINER -dp 8888:8081 $DOCKER_IMAGE
        - docker image prune -af
    tags:
        - deploy

showlog:
    stage: showlog
    variables:
        GIT_STRATEGY: none
    script:
        - sleep 20
        - docker logs $DOCKER_CONTAINER
    tags:
        - deploy
```

### Dựng Docker CI/CD Dockerfile không push lên dockerhub

``` yml
variables:
  DOCKER_IMAGE: ${CI_PROJECT_NAME}:${CI_COMMIT_TAG}_${CI_COMMIT_SHORT_SHA}
  DOCKER_CONTAINER: ${CI_PROJECT_NAME}

stages:
  - build
  - deploy
  - showlog

build:
  stage: build
  variables:
    GIT_STRATEGY: clone
  script:
    - docker build -t $DOCKER_IMAGE .
  tags:
    - deploy

deploy:
  stage: deploy
  variables:
    GIT_STRATEGY: none
  script:
    - docker rm -f $DOCKER_CONTAINER || true
    - echo "Running new container..."
    - docker run -d --name $DOCKER_CONTAINER -p 8080:8080 $DOCKER_IMAGE
    - docker image prune -af
  tags:
    - deploy

showlog:
  stage: showlog
  variables:
    GIT_STRATEGY: none
  script:
    - sleep 10
    - docker logs --tail=50 $DOCKER_CONTAINER
  tags:
    - deploy

```

### Docker CI/CD docker-compose

``` bash
# Nhớ thêm image: trong file docker-compose <br> sau bước build nếu muốn push lên dockerhub registry
image: ${REGISTRY_URL}/${REGISTRY_PROJECT}/backend:${CI_COMMIT_TAG}_${CI_COMMIT_SHORT_SHA}
# trong file docker-compose <br> sau bước build nếu muốn push lên dockerhub registry
```

``` yml
variables:
  DOCKER_COMPOSE_FILE: fullstack-compose.yml
  GIT_STRATEGY: clone

stages:
  - build
  - deploy
  - showlog

build:
  stage: build
  before_script:
    - echo "$REGISTRY_PASSWORD" | docker login -u "$REGISTRY_USER" --password-stdin $REGISTRY_URL
  script:
    # Build & push toàn bộ image được định nghĩa trong docker-compose.yml
    - docker compose -f $DOCKER_COMPOSE_FILE build
    - docker compose -f $DOCKER_COMPOSE_FILE push
  tags:
    - deploy

deploy:
  stage: deploy
  variables:
    GIT_STRATEGY: none
  script:
    - echo "Pull latest images from registry..."
    - docker compose -f $DOCKER_COMPOSE_FILE pull
    - echo "Restarting services..."
    - docker compose -f $DOCKER_COMPOSE_FILE down
    - docker compose -f $DOCKER_COMPOSE_FILE up -d
    - docker image prune -af
  tags:
    - deploy

showlog:
  stage: showlog
  variables:
    GIT_STRATEGY: none
  script:
    - sleep 10
    - docker compose -f fullstack-compose.yml logs backend --tail=50
  tags:
    - deploy
```

### Nếu không muốn push docker image lên docker hub

``` yml
variables:
  DOCKER_COMPOSE_FILE: fullstack-compose.yml
  GIT_STRATEGY: clone

stages:
  - build
  - deploy
  - showlog

build:
  stage: build
  script:
    - echo "Building local Docker images..."
    - docker compose -f $DOCKER_COMPOSE_FILE build
  tags:
    - deploy

deploy:
  stage: deploy
  variables:
    GIT_STRATEGY: none
  script:
    - echo "Deploying local Docker images..."
    - docker compose -f $DOCKER_COMPOSE_FILE down
    - docker compose -f $DOCKER_COMPOSE_FILE up -d
    - docker image prune -af
  tags:
    - deploy

showlog:
  stage: showlog
  variables:
    GIT_STRATEGY: none
  script:
    - sleep 10
    - docker compose -f fullstack-compose.yml logs backend --tail=50
  tags:
    - deploy

```

# Thiết lập cài đặt Jenkins

- Setup thư mục 

``` bash
mkdir tools
cd tools/
mkdir jenkins
cd jenkins/
nano jenkins-install.sh
```

- File tự động

``` bash
apt install openjdk-17-jdk -y
java --version
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA
apt-get update
apt install jenkins -y
systemctl start jenkins
systemctl enable jenkins
ufw allow 8080
```

- Chạy file

``` bash
chmod +x jenkins-install.sh
bash jenkins-install.sh
systemctl status jenkins

# Lấy mật khẩu
cat /var/lib/jenkins/secrets/initialAdminPassword
```

### shel host cho Jenkins

``` bash
sudo apt install nginx
nano /etc/nginx/conf.d/jenkins.duyduc.tech.conf

# nội dung bên trong nginx
server {
    listen 80;
    server_name jenkins.duyduc.tech;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep_alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# reload nginx
nginx -t 
nginx -s reload
```

- Nhớ tạo 1 User ADMIN để truy cập vào Jenkins

# Setup CI/CD Jenkins


### Trên server cần triển khai (lab-server)

``` bash
apt install openjdk-17-jdk openjdk-17-jre -y

# Nếu là dự án NodeJS thì cài đặt
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
apt-get install nodejs -y
sudo npm install -g npm@latest

# Thêm user Jenkins vào trong server
adduser jenkins
```

### Trên giao diện Jenkins ( Tạo agent )

1. vào dashboard -> manage jenkins -> nodes -> new node (đặt tên : lab-server , tích chọn Permanent Agent) -> create

2. Trong Name -> lables ( tương ứng với tên server ví dụ lab-server ) -> Remote root directory (tạo thư mục /var/lib/jenkins và <br> nhớ tạo /var/lib/jenkins trong cả lab-server)-> trước khi save nhìn bước ở dưới -> save

3. mở tab google thứ 2 vào link của jenkins -> dashboard -> manage jenkins -> security -> Agents (TCP port for inbound agent) chọn <br> fixed để port 8999 (port này trên) jenkins-server và không có dịch vụ nào chạy -> save -> sau đó hãy save bước ở trên 

4. netstat -tlpun ( trong jenkins-server sẽ thấy cổng 8999 )

5. Nhấn vào lab-server sau khi được tạo sẽ xuất hiện hướng dẫn Recommend sử dụng Unix nhớ copy lại

### Bên trong lab-server

``` bash
chown jenkins. /var/lib/jenkins 
cd /var/lib/jenkins
su jenkins

# Lấy từ bên trong Jenkins sau khi setup và paste vào trong này
echo 2885cab07ec8e099947e97435d5625d2da2ea7934efc40fcf71588f83ab6ebc6 > secret-file
curl -sO http://jenkins.duyduc.tech:8080/jnlpJars/agent.jar
java -jar agent.jar -url http://jenkins.duyduc.tech:8080/ -secret @secret-file -name "lab-server" -webSocket -workDir "/var/lib/jenkins" > nohup.out 2>&1 &
```

* quay lại trang jenkins rồi F5 lại sẽ thấy Agent is connected.

### Trên giao diện Jenkins

1. Dashboard -> new Item -> Folder (đặt tên tùy theo dự án Action in lab) -> save 

2. Dashboard -> manage jenkins -> plugins -> tích chọn vào gitlab && blue ocean -> <br> install -> tích chọn vào Restart Jenkins when installation is complete

3. Sau khi install thành công -> installed plugin sẽ thấy blue ocean && gitlab-plugins

4. Dashboard -> manage jenkins -> system -> kéo xuống sẽ thấy Gitlab

* Collection name : gitlab server

* Gitlab host URL : http://gitlab.duyduc.tech (ví dụ)

* Credential (ở đây cần kết nối qua API Token)  

### Qua Gitlab tạo một user jenkins có quyền admin

* Đăng nhập bằng tài khoản jenkins -> edit profile -> access Token -> Token name (đặt tên token : token for jenkins server connection hoặc tùy theo ý tưởng của bạn)

* Select scopes -> tích chọn api -> create personal access token

* nhớ lưu lại Api token sau khi tạo

### qua lại bên Jenkins

* Credentials: Add -> jenkins -> Add credentials <br> -> kind (Gitlab API token) 
<br> -> API token (paste API Token vừa lấy ở gitlab vào đây) <br> -> ID : Jenkins-gitlab-user <br> -> description : Jenkins-gitlab-user -> Add 

* Sau khi tạo thành công sẽ thấy ở phần Credentials -> chọn Gitlab API token(jenkins-gitlab-user) - Test Connection -> save

### Hướng dẫn kết nối gitlab của dự án đến jenkins

* Dashboard -> Action in lab -> new Item -> pipeline(tên dự án ví dụ shoeshop) -> Ok -> Discard old build -> Max of builds to keep (10) <br> -> kéo xuống build triggers -> build when a change is pushed to gitlab -> chọn Push Event && Accepted Merge Request Events <br> -> kéo xuống Pipeline -> Definition ( chọn pipeline script from SCM ) -> SCM ( chọn git ) -> Repository URL ( dùng link git của dự án => http://gitlab.duyduc.tech/vegetfood/vegetfood.git ) <br> -> kéo xuống Credentials -> Add -> jenkins -> Add Credentials -> Kind (Username with password) -> Username : jenkins (được tạo trên gitlab) && Password : paste API Token vừa lấy ở gitlab vào đây <br> -> ID : jenkins-gitlab-user-account -> Description : jenkins-gitlab-user-account -> Add <br> -> Và sau đó out ra Credentials -> chọn user vừa được tạo (jenkins-gitlab-user-account) <br> -> kéo xuống Branches to build -> Branch Specifier (chọn nhánh muốn build khi có merge request ví dụ develop , ngoài ra có thể Add Branch các nhánh khác vào) -> save

### Vào Gitlab của dự án 

* Menu -> Admin -> Setting -> Network -> Outbound requests -> click chọn cả 2 tuỳ chọn (Allow Requests to the local network from the web hooks and services <br>, Allow Requests to the local network from system hooks)  -> Save changes

* Settings -> Webhooks -> ở phần URL (format chính của URL http://<URL của jenkins>/project/<Đường dẫn dự án trên jenkins>/ <br> ví dụ:
http://103.228.75.154:8080/project/Action_in_lab/vegetfood) <br> -> tích chọn Push Event && Tags event && merge request events && bỏ tuỳ chọn enable ssl -> Add webhook

### Trên dashboard Jenkins

* Cấu hình Jenkins để cho phép anonymous được trigger job

1. Truy cập Jenkins với tài khoản admin -> Vào Manage Jenkins → Security.

2. Trong mục Authorization, chọn:

* ✅ "Matrix-based security".

* Tìm hàng "anonymous" → bật quyền:
		
* Job > Build (và Job > Read, nếu cần)

* Lưu lại.

### Vào Gitlab của dự án 

* Kiểm tra lại GitLab webhook.
	  
* Sau khi Add webhook -> ở dưới sẽ xuất hiện Project Hooks(1) -> Chọn Test -> tích chọn Push events 

### Bên trong lab-server

``` bash
visudo

### nội dung 
jenkins  ALL=(ALL) NOPASSWD: ALL
jenkins  ALL=(ALL) NOPASSWD: /bin/mkdir*
jenkins  ALL=(ALL) NOPASSWD: /bin/cp*
jenkins  ALL=(ALL) NOPASSWD: /bin/chown*
jenkins  ALL=(ALL) NOPASSWD: /bin/su springbe
```

### Vào Gitlab của dự án

1. Tạo một Jenkinsfile bên nhánh develop của gitlab và triển khai pipeline trên Repo nhánh develop của dự án đó 

2. Truy cập Jenkins web: http://your-jenkins-url
			
3. Chọn Manage Jenkins → Credentials

4. Chọn phạm vi Global -> Click (global) → Add Credentials

5. Ở màn hình “Add Credentials”, chọn:
			
- Kind: Secret file

- File: chọn file postgres.env bạn đã chuẩn bị

- ID: đặt một ID dễ nhớ, ví dụ: postgres-env-file

- Description: PostgreSQL Environment File (tuỳ chọn)

- Nhấn OK để lưu

### Jenkinsfile cho Springboot

``` bash
pipeline {
    agent { label 'server-product' }

    environment {
        DEPLOY_DIR = '/opt/backend-data'
        DATA_LOG = '/var/log/java'
        JAR_NAME = 'ecom-proj-0.0.1-SNAPSHOT.jar'
        JAR_PATH = "target/${JAR_NAME}"
        PORT = '8080'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    sh 'mvn clean install'
                }
            }
        }

    stage('Deploy') {
        steps {
            sh '''
                mkdir -p ${DEPLOY_DIR}
                mkdir -p ${DATA_LOG}
                sudo fuser -k $PORT/tcp || true
                sudo chown -R jenkins:jenkins ${DATA_LOG}
                sudo cp ${JAR_PATH} ${DEPLOY_DIR}
                sudo chown -R jenkins:jenkins ${DEPLOY_DIR}
                cd ${DEPLOY_DIR}
                sudo -u jenkins nohup java -jar ${JAR_NAME} > ${DATA_LOG}/backend.log 2>&1 &
            '''
        }
    }


        stage('Show Log') {
            steps {
                sh '''
                    tail -n 20 ${DATA_LOG}/backend.log || echo "No log file found."
                '''
            }
        }
    }

    post {
        failure {
            echo "❌ Build or deployment failed!"
        }
        success {
            echo "✅ Pipeline completed successfully!"
        }
    }
}

```

### Jenkinsfile cho NestJS

``` bash
pipeline {
    agent { label 'lab-server' }

    environment {
        DEPLOY_DIR = "/opt/backend-data"
        APP_NAME   = "main.js"
        DIST_DIR   = "dist"
        PORT       = "3000"
    }

    stages {

        stage('Build') {
            environment {
                GIT_STRATEGY = "clone"
            }
            steps {
                sh '''
                    npm install
                    npm run build
                '''
            }
        }

        stage('Deploy') {
            environment {
                GIT_STRATEGY = "none"
            }
            steps {
                # Trường hợp env trong jenkins
                withCredentials([file(credentialsId: 'postgres-env-file', variable: 'ENV_FILE')]) {
                    sh '''
                        sudo fuser -k $PORT/tcp || true
                        sudo mkdir -p $DEPLOY_DIR
                        sudo cp -r $DIST_DIR/* $DEPLOY_DIR/
                        sudo cp package*.json $DEPLOY_DIR/
                        sudo cp $ENV_FILE $DEPLOY_DIR/.env
                        sudo chown -R jenkins:jenkins $DEPLOY_DIR
                        cd $DEPLOY_DIR
                        export $(cat .env | xargs)
                        npm install --omit=dev
                        sudo -u jenkins nohup node $APP_NAME > nohup.out 2>&1 &
                    '''
                }

                # Trường hợp không cần env 
                script {
                    sh """
                        sudo fuser -k $PORT/tcp || true
                        sudo mkdir -p $DEPLOY_DIR
                        sudo cp -r $DIST_DIR/* $DEPLOY_DIR/
                        sudo cp package*.json $DEPLOY_DIR/
                        sudo chown -R jenkins:jenkins $DEPLOY_DIR
                        cd $DEPLOY_DIR
                        npm install --omit=dev
                        sudo -u jenkins nohup node $APP_NAME > nohup.out 2>&1 &
                    """
                }
            }
        }

        stage('Show Log') {
            environment {
                GIT_STRATEGY = "none"
            }
            steps {
                sh '''
                    tail -n 100 $DEPLOY_DIR/nohup.out || echo "❌ No log found."
                '''
            }
        }
    }

    post {
        always {
            script {
                sh """
                    sudo chown -R jenkins:jenkins $DEPLOY_DIR || true
                    sudo chmod -R 755 $DEPLOY_DIR || true
                """
            }
        }
    }
}

```
* Sau khi triển khai xong thì qua bên Jenkins của dự án sẽ xuất hiện dưới Build History , có thể vào Blue Ocean để xem logs của Pipelines

# Setup CI/CD with github action

## BƯỚC 1: Chuẩn bị Server

### 1.1 Tạo SSH Key trên server

``` bash
# Chạy trên SERVER của bạn
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/github_actions
```

#### Lệnh này tạo ra 2 file:

- ~/.ssh/github_actions → Private Key (dùng cho GitHub)
- ~/.ssh/github_actions.pub → Public Key (dùng cho server)

### 1.2 Thêm Public Key vào authorized_keys

``` bash
# Vẫn trên SERVER
cat ~/.ssh/github_actions.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### 1.3 Lấy Private Key để điền vào GitHub

``` bash
cat ~/.ssh/github_actions
# Copy toàn bộ nội dung từ -----BEGIN... đến ...END-----
```

## BƯỚC 2: Cấu hình GitHub Secrets

- Vào repo GitHub → Settings → Secrets and variables → Actions → New repository secret

- Tạo các secret sau:

| Secret Name     | Giá trị |
|-----------------|--------|
| SSH_PRIVATE_KEY | Nội dung private key vừa copy |
| SSH_HOST        | IP Public (vd: 123.456.789.0) |
| SSH_USER        | User SSH (vd: root) |
| SSH_PORT        | Port SSH (thường là 22) |
| SSH_PATH        | Đường dẫn dự án trên server (vd: /root/fullstack/nest-task-ecs/nest-order-api) |

## BƯỚC 3: Cấu trúc thư mục

- Giả sử monorepo của bạn trông như này:

``` bash
fullstack/
├── frontend-example/
├── infra-terraform/
├── nest-task-ecs/
│   ├── nest-item-api/   ← CI/CD cái này
│   └── nest-order-api/
├── .env
└── docker-compose.yml
```

## BƯỚC 4: Tạo file CI/CD

- Tạo file .github/workflows/nest-item-api.yml :

``` yaml
name: NestJS Item API CI/CD

on:
  push:
    branches:
      - main
    paths:
      - 'nest-task-ecs/nest-item-api/**'
  pull_request:
    branches:
      - main
    paths:
      - 'nest-task-ecs/nest-item-api/**'

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./nest-task-ecs/nest-item-api

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: nest-task-ecs/nest-item-api/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Build NestJS
        run: npm run build

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: nest-item-api-build
          path: nest-task-ecs/nest-item-api/dist/
          retention-days: 1

  test:
    name: Test
    runs-on: ubuntu-latest
    needs: build
    defaults:
      run:
        working-directory: ./nest-task-ecs/nest-item-api

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: nest-task-ecs/nest-item-api/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm run test
        continue-on-error: true

  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: [build, test]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy lên server qua SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script: |
            echo "Pull code mới nhất"
            cd ${{ secrets.SSH_PATH }}
            git pull origin main

            echo "Cài dependencies"
            cd nest-task-ecs/nest-item-api
            npm ci
            npm audit fix --force
            npm run build
            nohup npm start > app.log 2>&1 &
            echo "Deploy thành công!"
```

# Trường hợp không có IP public dùng SSM 

## Bước 1: Thêm Secrets mới vào GitHub

- Vào repo GitHub → Settings → Secrets and variables → Actions → New repository secret

- Tạo các secret sau:

| Secret Name     | Giá trị |
|-----------------|--------|
| AWS_ACCESS_KEY_ID | Access key IAM user|
| AWS_SECRET_ACCESS_KEY   | Secret key IAM user |
| AWS_REGION        | vd: ap-southeast-1 |
| EC2_INSTANCE_ID        | vd: i-0abc123def456 |
| SSH_PATH        | Đường dẫn dự án trên server (vd: /root/fullstack) |

## Bước 2: Cài SSM Agent trên EC2 Ubuntu

``` bash
# SSH vào EC2 lần cuối rồi chạy
sudo snap install amazon-ssm-agent --classic
sudo systemctl enable snap.amazon-ssm-agent.amazon-ssm-agent.service
sudo systemctl start snap.amazon-ssm-agent.amazon-ssm-agent.service

# Kiểm tra đã chạy chưa
sudo systemctl status snap.amazon-ssm-agent.amazon-ssm-agent.service
```

## Bước 3: Gắn IAM Role cho EC2

- Vào AWS Console → EC2 → chọn instance → Actions → Security → Modify IAM Role

- Tạo role mới với policy: AmazonSSMManagedInstanceCore → gắn vào EC2

## Bước 4: File CI/CD update lại phần Deploy

``` yaml
name: NestJS Item API CI/CD

on:
  push:
    branches:
      - main
    paths:
      - 'nest-task-ecs/nest-item-api/**'
  pull_request:
    branches:
      - main
    paths:
      - 'nest-task-ecs/nest-item-api/**'

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./nest-task-ecs/nest-item-api

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: nest-task-ecs/nest-item-api/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Build NestJS
        run: npm run build

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: nest-item-api-build
          path: nest-task-ecs/nest-item-api/dist/
          retention-days: 1

  test:
    name: Test
    runs-on: ubuntu-latest
    needs: build
    defaults:
      run:
        working-directory: ./nest-task-ecs/nest-item-api

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: nest-task-ecs/nest-item-api/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm run test
        continue-on-error: true

  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: [build, test]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Deploy qua SSM
        id: ssm_deploy
        run: |
          COMMAND_ID=$(aws ssm send-command \
            --instance-ids "${{ secrets.EC2_INSTANCE_ID }}" \
            --document-name "AWS-RunShellScript" \
            --parameters 'commands=[
              "cd ${{ secrets.SSH_PATH }}",
              "git pull origin main",
              "cd nest-task-ecs/nest-item-api",
              "npm ci",
              "npm audit fix --force",
              "npm run build",
              "nohup npm start > app.log 2>&1 &",
              "echo Deploy thanh cong!"
            ]' \
            --output text \
            --query "Command.CommandId")
          echo "command_id=$COMMAND_ID" >> $GITHUB_OUTPUT

      - name: Chờ SSM chạy xong và in log
        run: |
          aws ssm wait command-executed \
            --command-id "${{ steps.ssm_deploy.outputs.command_id }}" \
            --instance-id "${{ secrets.EC2_INSTANCE_ID }}"

          aws ssm get-command-invocation \
            --command-id "${{ steps.ssm_deploy.outputs.command_id }}" \
            --instance-id "${{ secrets.EC2_INSTANCE_ID }}" \
            --query "StandardOutputContent" \
            --output text
```
