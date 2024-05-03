# APIサーバの設定
## 概要
この説明は、ハンズオン課題における`Week2：基本サービス`にて必要となる、APIサーバに対する設定方法を説明しています。

## 前提
- APIサーバのEC2インスタンスにssh接続などでログインしていること

## 手順

### 1. yumのアップデート
システムを最新の状態に保つためにyumパッケージをアップデートします。
```shell
sudo yum update -y
```

### 2. Gitのインストール
EC2インスタンスにソースコードをダウンロードするために、Gitをインストールします。
```shell
sudo yum install -y git
```

### 3. Goのインストール
APIサーバとして機能するGo言語をインストールします。
```shell
sudo yum install -y golang
```
インストール後、Goのバージョンを確認します。
```shell
go version
```

### 4. ソースコードのダウンロード
Gitを使用してソースコードをダウンロードします。
```shell
git clone https://github.com/CloudTechOrg/cloudtech-reservation-api.git
```

### 5. サービスの自動起動設定
システムの再起動時にもAPIが自動で起動するようにsystemdを設定します。

まずはviエディターを使用し、サービス起動時の設定ファイルを作成します。
```shell
sudo vi /etc/systemd/system/goserver.service
```
以下の内容をファイルに追記し、保存を行います。
```
[Unit]
Description=Go Server

[Service]
WorkingDirectory=/home/ec2-user/cloudtech-reservation-api
ExecStart=/usr/bin/go run main.go
User=ec2-user
Restart=always

[Install]
WantedBy=multi-user.target
```
設定を有効にし、サービスを開始します。
```shell
sudo systemctl daemon-reload
sudo systemctl enable goserver.service
sudo systemctl start goserver.service
```

### 6. リバースプロキシの設定
8080ポートで動作するGoのAPIを80ポートで利用できるように、Nginxをリバースプロキシとして設定します。
```shell
sudo yum install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```
Nginxの設定ファイルを編集し、適切なリバースプロキシ設定を行います。
```shell
sudo vi /etc/nginx/nginx.conf
```
設定を更新した後、Nginxを再起動します。
```shell
sudo systemctl restart nginx
```

## 起動方法
下記は、ローカルまたは外部からAPIを呼び出す方法です。
```
# ローカルからのアクセス
http://localhost:8080

# 外部からのアクセス
http://[IPアドレス]:8080
```


# データベース接続
## 概要
APIサーバから、RDSに接続できるように設定する

## 前提
- APIサーバのEC2インスタンスにsshなどでログインしていること

## 手順

### 1. mysqlのインストール

パッケージマネージャーdefのインストール
```shell
sudo dnf update -y
```

MSQLのリポジトリ設定
```shell
sudo rpm -Uvh https://dev.mysql.com/get/mysql80-community-release-el8-1.noarch.rpm
```

MySQLサーバのインストール
```shell
sudo dnf install mysql-community-server -y
```

MySQLサービスの起動
```shell
sudo systemctl start mysqld
```

システム起動時にMySQLが自動起動するように設定
```shell
sudo systemctl enable mysqld
shell
```

### 2. RDSに接続

以下のコマンドで、RDSのMySQLに接続
```
mysql -h 【エンドポイント】 -P 3306 -u admin -p
```

エンドポイントは、RDSのコンソールから確認可能


### 3. データベースとテーブルの作成

以下のコマンドで、reservation_dbというデータベースを作る
```sql
create database reservation_db;
```

以下のコマンドで、Reservationsテーブルを作成する
```sql
CREATE TABLE reservation_db.Reservations (
    ID INT AUTO_INCREMENT PRIMARY KEY,
    company_name VARCHAR(255) NOT NULL,
    reservation_date DATE NOT NULL,
    number_of_people INT NOT NULL
);
```

以下のコマンドで、Reservationsテーブルに1件追加する
```sql
INSERT INTO reservation_db.Reservations (company_name, reservation_date, number_of_people)
VALUES ('株式会社テスト', '2024-04-21', 5);
```

### 5. 動作確認
以下のCURLコマンドで、データベース接続が正しく行われていることを確認する

```shell
curl http://localhost:8080/test
```