---
title: "VPSを借りてmisskeyをセットアップしてみた"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vps,misskey,setup]
published: true
---
ドメイン、サブドメインを持っている事前提です。
**1.VPSを借ります**
メモリ2GB以上でないとDockerのBuildで失敗します。
OSはUbuntuの22.04にしてみました。
※運営している友達曰く4GB以上必要そうな感じです。
![](/images/misskey/vps.png)
**2.consoleを開きます**
![](/images/misskey/vps.console.png)
**3.VPSのrootにログイン**
```
login: root
password: # VPSを借りた時に設定したパスワード
```
**4.ユーザー追加とsudo権限の付与、追加したUserでログイン**
```
sudo adduser USER_NAME
sudo gpasswd -a USER_NAME
su USER_NAME
cd ~
```
**5.Dockerのインストール**
```
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt update

apt-cache policy docker-ce
sudo apt install docker-ce
```
下記のコマンドでバージョンが出たら成功
```
sudo docker -v
```
**6.Docker-Composeのインストール**
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
下記のコマンドでバージョンが出たら成功
```
sudo docker-compose -v
```

権限の付与

```
sudo gpasswd -a USER_NAME docker
```
**7.Nginxのインストール**
以下のDockerImageを使う
一つのサーバーで複数のサイト利用の為:
https://hub.docker.com/r/jwilder/nginx-proxy
サイトのHTTPS化のため:
https://hub.docker.com/r/jrcs/letsencrypt-nginx-proxy-companion/
`docker-compose.yml`ファイルを作るのが面倒なため、GITを用意した
https://github.com/FRICK-ELDY/nginx-docker-install/blob/master/docker-compose.yml

任意:GitHubディレクトリを作成
```command
mkdir github
cd github
```
クローンした`docker-compose.yml`を`docker-compose up -d`する
```Elixir
git clone https://github.com/FRICK-ELDY/nginx-docker-install.git
cd nginx-docker-install
# misskeyとdocker-compose.ymlが分離する為、
# Docker networkを作成し同じネットワークを利用する
sudo docker network create external_network
# docker-composeを起動させておく
sudo docker-compose up -d
```
**8.Misskeyの構築**
Gitからクローン
```Elixir
# GitHubのディレクトリに戻る
cd ../ 
# repositoryの取得
git clone -b master https://github.com/misskey-dev/misskey.git
cd misskey
git checkout master
```
.configフォルダにあるexampleファイルのコピーを作成
```Elixir
cp .config/docker_example.yml .config/default.yml
cp .config/docker_example.env .config/docker.env
```
設定ファイルの編集
```Elixir:.config/default.yml
vi .config/default.yml #Vimでファイルを開く
#<i>編集モード,<ESC>ノーマルモード,<:>コマンドモード
# -------------------VIM-------------------
# 9行目
- url: https://example.tld/
+ url: https://misskey.frick96.com/
# 44-45行目
- user: example-misskey-user
- user: example-misskey-pass
+ user: misskey-user
+ user: misskey-pass
# -------------------VIM-------------------
# 編集モードからノーマルモードへ切り替えて
# :wq で保存して終了(:q!で保存せずに終了)
```
```Elixir:.config/docker.env
vi .config/docker.env #Vimでファイルを開く
#<i>編集モード,<ESC>ノーマルモード,<:>コマンドモード
# -------------------VIM-------------------
- POSTGRES_PASSWORD=example-misskey-user
- POSTGRES_USER=example-misskey-pass
+ POSTGRES_PASSWORD=misskey-user
+ POSTGRES_USER=misskey-pass
# -------------------VIM-------------------
# 編集モードからノーマルモードへ切り替えて
# :wq で保存して終了(:q!で保存せずに終了)
```
docker-compose.ymlの編集
公式ではbuildを使った方法だが、MisskeyのDockerImageがあるのでそちらを使用。
build時間が不要になる。
```Elixir:docker-compose.yml
# exampleファイルをコピー
cp ./docker-compose.yml.example ./docker-compose.yml
#Vimでファイルを開く
vi docker-compose.yml
#<i>編集モード,<ESC>ノーマルモード,<:>コマンドモード
# -------------------VIM-------------------
version: "3"

services:
  web:
#   build: . 削除
+   image: misskey/misskey:latest
    restart: always
    links:
      - db
      - redis
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    ports:
      - "3000:3000"
    networks:
      - internal_network
      - external_network
    volumes:
      - ./files:/misskey/files
      - ./.config:/misskey/.config:ro
+   environment:
+     VIRTUAL_HOST: misskey.frick96.com # default.ymlのurlを指定
+     VIRTUAL_POST: 3000
+     LETSENCRYPT_HOST: misskey.frick96.com # default.ymlのurlを指定
+     LETSENCRYPT_EMAIL: example@example.com

  redis:
    restart: always
    image: redis:7-alpine
    networks:
      - internal_network
    volumes:
      - ./redis:/data
    healthcheck:
      test: "redis-cli ping"
      interval: 5s
      retries: 20

  db:
    restart: always
    image: postgres:15.1-alpine
    networks:
      - internal_network
    env_file:
      - .config/docker.env
    volumes:
      - ./db:/var/lib/postgresql/data
    healthcheck:
      test: "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"
      interval: 5s
      retries: 20

networks:
  internal_network:
    internal: true
  external_network:
+   external: true
# -------------------VIM-------------------
# 編集モードからノーマルモードへ切り替えて
# :wq で保存して終了(:q!で保存せずに終了)
```
Misskeyを立ち上げ
```
sudo docker compose run --rm web pnpm run init
sudo docker compose up -d
```
![](/images/misskey/misskey.png)

無事、公開されました！
お疲れ様でしたω
https://misskey.frick96.com/
良ければフォロー、ハートお願いしますω
# 参考
Misskeyのアップデート方法は以下から
https://misskey-hub.net/docs/install/docker.html

参考サイト
https://girak.net/2022/06/03/300
https://7ka.org/install-misskey-on-docker/
https://blog.noellabo.jp/entry/2019/08/14/8i3RHuZ1wJNDinIn

