# Podman 優勢
```
podman 可用來：

建立虛擬 ubuntu 或其它 linux os
建立 Apache 或各種伺服器
建立 MySQL 或執行各種資料庫
那為什麼不直接用 docker 就好了，有幾個原因：
- Kubernetes 已經棄用 Docker
- Red Hat 也使用 podman 取代 Docker
- Docker 不再是完全免費使用，podman 依舊是免費開放原始碼
- podman is daemonless
- podman is rootless
- podman 不需要先在背景執行服務 (daemonless)，不像 Docker 需要一個背景服務來管理所有的容器，一旦服務有問題，所有啟動的容器就一起掛了。
- podman 不強制用 root 權限執行，相對比 Docker 安全許多。
- Podman 有 pod 的概念，也就是服務集群。
- 習慣用 Docker Compose 的人，也有 Podman Compose。
- 類似 Docker Desktop 的 Podman Desktop，並且不用擔心它的商業授權問題。


安裝路徑
podman(https://github.com/containers/podman/releases)
> podman-4.4.2-setup.exe
> a file referred to as a Containerfile can be a file named either ‘Containerfile’ or ‘Dockerfile’

podman desktop(https://podman-desktop.io/downloads/Windows)

```

## 建立Services - WordPress
1. 建立 pod
```
podman pod create --name wordpress-test -p 8081:80

指令說明：
podman pod create，建立 pod
–name，指定名稱
-p，port 對應，外部:內部
注意 pod 建立後就不能改 port mapping。
```
2. 執行 MariaDB
```
podman run -d --pod=wordpress-test \
  -e MYSQL_ROOT_PASSWORD="geheim" \
  -e MYSQL_DATABASE="wp" \
  -e MYSQL_USER="wordpress" \
  -e MYSQL_PASSWORD="w0rdpr3ss" \
  --name=wordpress-test-db mariadb
  
為什麼不是 MySQL，因為呢，MariaDB 有提供 arm64 版本的 image，可以在 M1 MacBook 上執行，MySQL 沒有，
而且 MariaDB 的所有參數和使用方式，和 MySQL 一模一樣，啊就是為了無縫取代 MySQL。

指令說明：
podman run，執行容器
-d，run as daemon
–pod，指定在上面建立的 pod 中執行
-e，設定環境變數，資料庫的帳號密碼等等
–name，指定容器名稱，方便管理
```
3. 執行 WordPress
```
podman run -d  --pod=wordpress-test \
  -e WORDPRESS_DB_NAME="wp" \
  -e WORDPRESS_DB_USER="wordpress" \
  -e WORDPRESS_DB_PASSWORD="w0rdpr3ss" \
  -e WORDPRESS_DB_HOST="127.0.0.1" \
  --name wordpress-test-web wordpress
  
指令說明：
podman run，執行容器
-d，run as daemon
–pod，指定在上面建立的 pod 中執行
-e，設定環境變數，資料庫的帳號密碼等等
–name，指定容器名稱，方便管理

本機架設 WordPress 完成，開啟 http://localhost:8081，全新的 WordPress 安裝開始。
```

## 指令
```
完成初始化與啟動:
podman machine init(只要安裝後執行一次即可)
podman machine start

指令清單:
podman --help

列出所有镜像：
podman images

刪除鏡像:
podman image rm ID|Name
(Container需先刪除)

建立鏡像:
podman build -f Dockerfile.dmz -t todolistuiimg-dev:v1.00 . --no-cache
- error getting credentials - err: docker-credential-gcloud resolves to executable in current directory (.\docker-credential-gcloud), out: `` 
(remove c:/Users/<your account>/.docker/config.json)

啟動容器:
podman run -d -p 1443:1443 --name my-nginx todolistuiimg-dev:v1.00
-d : 背景執行
–name : 自訂易容器名稱
-p : 定義輸出的 port ( <host port>:<expose port>)
-v : 定義掛載的目錄 ( <host dir>:<application dir>)

停止、刪除容器:
podman ps
podman stop ContainerID
podman rm ContainerID
(podman container rm ContainerName)

執行 container 內的指令:
podman exec -it rocky1 sh
那加上 -it 之後，你就會在 container 內停留操作 sh/bash

監看容器Log
podman logs ContainerName

登入Cloud Repository
podman login irepocld.cminl.oa --tls-verify=false
(--tls-verify=false for fix X509 issue)
podman tag todolistapi-dev:1.06 irepocld.cminl.oa/todolistapi-dev/todolistapi-dev:1.06
podman push irepocld.cminl.oa/todolistapi-dev/todolistapi-dev:1.06 --tls-verify=false

Podman跟VS/VS Code IDE整合 ?
```
