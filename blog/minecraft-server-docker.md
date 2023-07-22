---
title: Dockerとserveoを使って爆速でプラグイン対応マイクラサーバー構築
description: Dockerとserveoを使って爆速でプラグイン対応マイクラサーバー構築
slug: minecraft-server-docker
authors:
  - name: marukun_
    url: https://github.com/marukun712
    image_url: https://github.com/marukun712.png

tags: [docker, minecraft]
hide_table_of_contents: false
---
# はじめに
プラグイン対応マイクラサーバーの構築って、手順が多くてめんどくさいですよね。
そこで、今回はDockerとserveoを使ってポート解放なしでマイクラサーバーを簡単に構築する方法をご紹介します。
# 環境
- MacOS Ventura 13.3
- Docker 20.10.16
- docker-compose 1.29.2

Dockerはhomebrewでの導入をお勧めします。

https://qiita.com/nemui_/items/ed753f6b2eb9960845f7

# 手順
今回はこのイメージを使ってマイクラサーバーを構築します。

[itzg/minecraft-server](https://hub.docker.com/r/itzg/minecraft-server)

https://github.com/itzg/docker-minecraft-server

## Dockerイメージのpull
Dockerイメージをpullします。
```bash
$ docker pull itzg/minecraft-server
```
## docker-compose.ymlを書く

今回はdocker-composeを使ってコンテナを起動します。

docker-compose.ymlを作成し、以下のように記述します。
```yaml
version: '3'

services:
  mc:
    image: itzg/minecraft-server
    ports:
      - 25565:25565
    environment:
      - EULA=TRUE
      - TYPE=SPIGOT
      - MEMORY=4G
    tty: true
    stdin_open: true
    restart: unless-stopped
    volumes:
      # attach a directory relative to the directory containing this compose file
      - ./minecraft-server:/data

```
## 解説

### version
docker-composeのバージョンを定義しています。
### services
docker-composeではアプリケーションを構成する各要素のことをまとめてservicesと呼んでいます。
### image
起動するdockerイメージを指定しています。
### ports 
サーバーが使用するポートを指定しています。
### environment
コンテナ内で使われる環境変数を設定しています。
このイメージではこの環境変数を変更することでサーバーの種類やメモリ割当量などを変更しています。
このイメージで使えるサーバーの種類やその他の環境変数は

https://github.com/itzg/docker-minecraft-server/blob/master/README.md#server-types

ここから確認できます。
### volumes
ここで指定したホスト側のディレクトリをコンテナ内にマウントしています。
マウントしたフォルダにマイクラサーバーのワールドデータなどが保存されます。

なんとコレだけでサーバー構築の設定が完了しました！
## サーバーを起動する
```bash
docker-compose up -d
```
実行するとサーバーがバックグラウンドで起動します。

マイクラからも接続できました。やったね！
![スクリーンショット 2023-04-04 8.48.16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2204837/40fc2766-be72-37f8-2812-66a0cb756ca6.png)

## コンソールにログインする
```bash
docker exec -i サーバーのコンテナ名 rcon-cli
```
rcon-cliを実行してマイクラサーバーのコンソールにログインすることができます。

## serveoで外部からアクセスできるようにする
```
ssh -R hogehoge.serveo.net:25560:localhost:25565 serveo.net
```
sshでポートフォワーディングします。

これで、サーバーが外部からもアクセスできる状態になりました。
hogehogeの部分を変更することで使いたいサブドメインを設定することができます。

# まとめ
docker-composeとserveoを利用すれば手軽にマイクラサーバーを構築することができる。
