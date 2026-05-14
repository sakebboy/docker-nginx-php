# docker-nginx-php

本機開發用 **Nginx + PHP-FPM（7.4 與 8.3.12 並行）+ MySQL + Redis** 的 Docker Compose 堆疊。

## 環境組成

| 服務名稱（Compose） | 說明 |
|---------------------|------|
| `docker-nginx` | Nginx，站台設定於 `nginx/conf.d/*.conf` |
| `docker-php-fpm` | PHP **7.4** FPM，映像建置目錄 `php-fpm/` |
| `docker-php8-fpm` | PHP **8.3.12** FPM，映像建置目錄 `php8-fpm/`（build **context 為本 repo 根目錄**，與 `php-fpm/` 共用 Oracle Instant Client zip） |
| `docker-mysql` | MySQL，資料目錄預設 `~/mysql-docker-data` |
| `redis` | Redis 5.x |

所有網站程式碼透過 **`../web:/var/www`** 掛載進容器（請與本 repo 同層建立 `web` 資料夾，例如 `D:\web` 與 `D:\docker-nginx-php` 並列）。

## 目錄結構（精簡）

```
docker-nginx-php/
├── docker-compose.yml
├── nginx/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── conf.d/          # 各站台 server 設定
├── php-fpm/
│   └── Dockerfile       # PHP 7.4
├── php8-fpm/
│   └── Dockerfile       # PHP 8.3.12
├── mysql/
└── redis.conf
../web/                  # 與本 repo 同層，對應容器內 /var/www
```

## 建置前置（PHP 8 映像）

- **Oracle Instant Client**：將 `instantclient-basic-linux.x64-12.1.0.2.0.zip` 與 `instantclient-sdk-linux.x64-12.1.0.2.0.zip` 放在 **`php-fpm/`**（與 7.4 映像相同位置；8.3 建置會從 repo 根目錄讀取 `php-fpm/` 內檔案）。
- **SAP NW RFC**（若 Dockerfile 未關閉）：需具備 `php8-fpm/sap/...` 等建置所需檔案。

## 常用指令

建議使用 Docker Compose V2（`docker compose`）。若仍使用舊版獨立程式，可將下列 `docker compose` 改為 `docker-compose`。

```bash
# 背景啟動（依需求可加服務名，例如僅 nginx + 兩個 php）
docker compose up -d

# 啟動／停止（容器已建立時）
docker compose start
docker compose stop

# 重建映像後啟動（例如 Dockerfile 或套件變更）
docker compose build
docker compose up -d
```

### Nginx 設定變更後重新載入

在 **Nginx 容器內** 執行（容器名稱請以 `docker compose ps` 為準，常見為 `docker-nginx-php-docker-nginx-1`）：

```bash
docker compose exec docker-nginx nginx -s reload
```

## PHP 7.4 與 8.3 並行

- 預設各 `conf.d` 內 `fastcgi_pass` 多為 **`docker-php-fpm:9000`**（PHP 7.4）。
- 需使用 PHP 8.3 的站台，請在該 `server` 的 `location ~ \.php$` 改為：

```nginx
fastcgi_pass docker-php8-fpm:9000;
```

兩個 FPM 服務須同時 `up`，無須為切換版本而改 Dockerfile。

### 容器內執行 PHP／Composer

```bash
docker compose exec docker-php-fpm php -v
docker compose exec docker-php8-fpm php -v
docker compose exec docker-php8-fpm bash
```

主機對應除錯埠（若映像有開）：PHP 7.4 常為 `6001:6001`，PHP 8.3 為 `6002:6001`（見 `docker-compose.yml`）。

## Windows 本機開發效能說明

專案放在 **`D:\web` 等 NTFS 路徑** 再掛進 Linux 容器時，大量小檔（例如 Laravel）會明顯比本機 XAMPP 慢。若可接受，請將 **`web` 與本 repo 一併放在 WSL2 的 Linux 檔案系統**（例如 `\\wsl$\Ubuntu\home\使用者\projects\`），並在 **WSL 終端機** 內執行 `docker compose`，通常可大幅改善。Docker Desktop 請啟用 **WSL 2 based engine** 及 **WSL Integration**（設定內可勾選）。

## 基底映像（對照）

| 元件 | 基底映像（摘要） |
|------|------------------|
| Nginx | `nginx:latest`（見 `nginx/Dockerfile`） |
| PHP 7.4 | `php:7.4-fpm-bullseye`（見 `php-fpm/Dockerfile`） |
| PHP 8.3 | `php:8.3.12-fpm-bookworm`（見 `php8-fpm/Dockerfile`） |

## PHP 擴充（摘要）

兩套 FPM 皆含多數常用擴充；**8.3** 另含 **FTP**（`ext-ftp`）、Microsoft ODBC **18**、Swoole **5.x**、OCI8 **3.x**、SAP RFC 模組較新版本等。完整清單以各目錄下 `Dockerfile` 為準（例如 `pcntl`、`sqlsrv`、`pdo_sqlsrv`、`redis`、`ldap`、`ssh2`、`intl`、`mysqli`、`pdo_mysql`、`bcmath`、`soap`、`zip`、`opcache`、`gd`、`odbc`、`oci8`、`pdo_oci`、`ftp`、`swoole`、`mcrypt` 等）。
