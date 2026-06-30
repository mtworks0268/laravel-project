その順番が良いです。

Laravel初心者の場合は、

**① ローカル構築**
↓
**② Git管理**
↓
**③ VPSデプロイ**

の順に進めると理解しやすいです。

---

# 全体像

最終的にこうなります。

```text
Mac

Docker
├── nginx
├── php
├── mysql
└── phpmyadmin

        ↓

Laravel

        ↓

GitHub

        ↓

VPS
```

まずはローカルだけ完成させましょう。

---

# Step1 プロジェクト作成

作業ディレクトリ作成

```bash
mkdir laravel-docker
cd laravel-docker
```

構成

```text
laravel-docker/

├── src/
├── docker/
├── docker-compose.yml
```

作成

```bash
mkdir src
mkdir docker
```

---

# Step2 PHP Dockerfile作成

作成

```bash
mkdir docker/php
touch docker/php/Dockerfile
```

docker/php/Dockerfile

```dockerfile
FROM php:8.3-fpm

RUN apt-get update && apt-get install -y \
    git \
    unzip \
    zip \
    curl \
    libzip-dev

RUN docker-php-ext-install pdo_mysql zip

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www
```

---

# Step3 Nginx設定

作成

```bash
mkdir docker/nginx
touch docker/nginx/default.conf
```

default.conf

```nginx
server {
    listen 80;

    root /var/www/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass php:9000;
        fastcgi_index index.php;

        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

---

# Step4 docker-compose作成

プロジェクト直下

```bash
touch docker-compose.yml
```

docker-compose.yml

```yaml
services:

  nginx:
    image: nginx:latest
    ports:
      - "8000:80"
    volumes:
      - ./src:/var/www
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - php

  php:
    build:
      context: .
      dockerfile: docker/php/Dockerfile
    volumes:
      - ./src:/var/www

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel
      MYSQL_USER: laravel
      MYSQL_PASSWORD: secret
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
      - "8080:80"
    environment:
      PMA_HOST: mysql

volumes:
  mysql_data:
```

---

# Step5 コンテナ起動

```bash
docker-compose up -d
```

確認

```bash
docker ps
```

表示されればOK

```text
nginx
php
mysql
phpmyadmin
```

---

# Step6 Laravelインストール

PHPコンテナへ入る

```bash
docker-compose exec php bash
```

Laravel作成

```bash
composer create-project laravel/laravel .
```

※最後のドット重要

```bash
.
```

現在のディレクトリにインストールされます。

---

# Step7 .env設定

src/.env

DB部分を変更

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306

DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=secret
```

---

# Step8 APP_KEY作成

コンテナ内

```bash
php artisan key:generate
```

---

# Step9 Migration

```bash
php artisan migrate
```

成功例

```text
users
password_reset_tokens
sessions
```

作成される

---

# Step10 動作確認

Laravel

```text
http://localhost:8000
```

Laravel画面表示

---

phpMyAdmin

```text
http://localhost:8080
```

ログイン

```text
user: laravel
pass: secret
```

---

# Step11 Breeze導入

ここからあなたが今勉強している内容です。

```bash
composer require laravel/breeze --dev
```

```bash
php artisan breeze:install
```

選択

```text
Blade with Alpine
```

---

実行

```bash
npm install
npm run build
```

---

Migration

```bash
php artisan migrate
```

---

確認

```text
http://localhost:8000/register
```

登録画面

```text
http://localhost:8000/login
```

ログイン画面

---

# Step12 Git管理

ローカル完成後

```bash
git init
```

```bash
touch .gitignore
```

Laravel標準の .gitignore を使用。

```bash
git add .
git commit -m "first commit"
```

GitHub作成

```bash
git remote add origin リポジトリURL
git push -u origin main
```

---

ここまでできれば、

✅ Docker
✅ Laravel
✅ MySQL
✅ phpMyAdmin
✅ Breeze認証
✅ GitHub

まで完成します。

その次に

**Step13 VPSへDockerごとデプロイ**
**Step14 独自ドメイン**
**Step15 HTTPS化（Let's Encrypt）**
**Step16 GitHub Actions自動デプロイ**

へ進むのがおすすめです。





Dockerでは**PHPをアンインストールする必要はありません**。

`php:8.3-fpm`はDockerイメージの一部なので、**Dockerfileを書き換えて再ビルドするだけ**でPHP 8.4.1になります。

## 手順1 Dockerfileを変更

すでに変更できています。

```dockerfile
FROM php:8.4.1-fpm

RUN apt-get update && apt-get install -y \
    git \
    unzip \
    zip \
    curl \
    libzip-dev \
    gnupg

RUN docker-php-ext-install pdo_mysql zip

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Node.js 22 LTS
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y nodejs

WORKDIR /var/www
```

---

## 手順2 コンテナを停止

```bash
docker-compose down
```

---

## 手順3 イメージを再ビルド

キャッシュを使わない方が確実です。

```bash
docker-compose build --no-cache
```

または

```bash
docker-compose up --build -d
```

---

## 手順4 起動

```bash
docker-compose up -d
```

---

## 手順5 PHPのバージョン確認

コンテナ名を確認

```bash
docker ps
```

例えば

```text
laravel_app
```

なら

```bash
docker exec -it laravel_app php -v
```

または

```bash
docker-compose exec app php -v
```

結果

```text
PHP 8.4.1 (cli)
```

となれば成功です。

---

## 8.3のイメージを削除したい場合

現在のイメージを確認します。

```bash
docker images
```

例

```text
php        8.3-fpm
php        8.4.1-fpm
```

8.3を削除するには

```bash
docker rmi php:8.3-fpm
```

ただし、他のコンテナが使用中の場合は削除できません。その場合は、まず該当コンテナを停止・削除してから実行します。

使用していないイメージやキャッシュをまとめて削除したい場合は、

```bash
docker system prune -a
```

を使うこともできます。ただし、**未使用のイメージ・コンテナ・ネットワークがすべて削除される**ので注意してください。

## おすすめのDockerfile

PHP 8.4系のLaravel開発なら、以下のように必要なライブラリやキャッシュ削減を意識した構成にすると使いやすいです。

```dockerfile
FROM php:8.4.1-fpm

RUN apt-get update && apt-get install -y \
    git \
    unzip \
    zip \
    curl \
    libzip-dev \
    gnupg \
    libicu-dev \
    libonig-dev \
    libxml2-dev \
    && docker-php-ext-install \
        pdo_mysql \
        zip \
        intl \
        mbstring \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y nodejs

WORKDIR /var/www
```

この構成はLaravel 12でも一般的な開発環境として使いやすく、`intl`や`mbstring`などLaravelでよく利用される拡張もあらかじめ含めています。
