# 目的

NIMによるScanをシンプルに事項するためのDocker Composeのデプロイ方法。

こちらのページの元となる情報は以下です。
- https://github.com/fabriziofiorucci/NGINX-NMS-Docker


# 前提
以下コンポーネントを利用します

```
docker docker-compose
```

# 手順
## 0. git clone
```
cd ~/
git clone https://github.com/BeF5/f5j-nms-docker-simple.git
```

## 1. ライセンスファイルのコピー
ライセンスファイルを取得し、フォルダにコピーしてください
```
$ ls
nginx-repo.crt  nginx-repo.key
```

# a. Docker Container の実行
## 1. Simple NIM の Docker Image Build
```
cd f5j-nms-docker-simple/docer-compose
cp ~/nginx-repo* .
sudo ./scripts/buildNIM.sh -C nginx-repo.crt -K nginx-repo.key -i -t nim
```

## 2. Container Imageの登録
作成されたDocker Iamageを適切なContainer Registryに登録

## 3. 実行
```
sudo docker-compose -f docker-compose.yaml up -d
```

## 4. 実行完了後のステータス
NIMが正しく動作した場合のサンプルのステータスを示します

動作するDockerイメージの状態。clickhouseと前の手順でBuildしたnimのイメージが動作します

```
$ sudo docker ps
CONTAINER ID   IMAGE                                    COMMAND                  CREATED          STATUS          PORTS                                                                                            NAMES
90479d97b943   clickhouse/clickhouse-server:21.12.4.1   "/entrypoint.sh"         17 minutes ago   Up 17 minutes   0.0.0.0:8123->8123/tcp, :::8123->8123/tcp, 0.0.0.0:9000->9000/tcp, :::9000->9000/tcp, 9009/tcp   f5j-nms-docker-simple_clickhouse_1
cbe21f598fb7   nim:latest                               "/bin/sh -c /deploym…"   17 minutes ago   Up 17 minutes   0.0.0.0:443->443/tcp, :::443->443/tcp, 0.0.0.0:5000->5000/tcp, :::5000->5000/tcp                 f5j-nms-docker-simple_nginx-nim2_1

```

サンプルログ
```
$ sudo docker logs $(sudo docker ps -a -f name=nim -q)
Waiting for ClickHouse...
Waiting for ClickHouse...
Using openssl version 3.0.2 to update NGINX password for admin in file: /etc/nms/nginx/.htpasswd
 * Starting nginx nginx
   ...done.
[91] [INF] Starting nats-server
[91] [INF]   Version:  2.9.1
[91] [INF]   Git:      [not set]
[91] [INF]   Name:     NAKHTJIAR5EXUKXQO4ASOM427BVOPXY34UR2FE5L2O5255IA55NQTT4J
[91] [INF]   ID:       NAKHTJIAR5EXUKXQO4ASOM427BVOPXY34UR2FE5L2O5255IA55NQTT4J
[91] [INF] Listening for client connections on 127.0.0.1:4222
[91] [INF] Server is ready
[80] [INF] Starting nats-server
[80] [INF]   Version:  2.9.1
[80] [INF]   Git:      [not set]
[80] [INF]   Name:     NDRB6PWV4DYBD4AUAKYRZJX4JWT6YX4SCZAR5VP44ONPIFFCISGRLEE4
[80] [INF]   Node:     5e1qS4jS
[80] [INF]   ID:       NDRB6PWV4DYBD4AUAKYRZJX4JWT6YX4SCZAR5VP44ONPIFFCISGRLEE4
[80] [INF] Starting JetStream
[80] [INF]     _ ___ _____ ___ _____ ___ ___   _   __  __
[80] [INF]  _ | | __|_   _/ __|_   _| _ \ __| /_\ |  \/  |
[80] [INF] | || | _|  | | \__ \ | | |   / _| / _ \| |\/| |
[80] [INF]  \__/|___| |_| |___/ |_| |_|_\___/_/ \_\_|  |_|
[80] [INF]
[80] [INF]          https://docs.nats.io/jetstream
[80] [INF]
[80] [INF] ---------------- JETSTREAM ----------------
[80] [INF]   Max Memory:      1.00 GB
[80] [INF]   Max Storage:     10.00 GB
[80] [INF]   Store Directory: "/var/lib/nms/streaming/jetstream"
[80] [INF] -------------------------------------------
[80] [INF] Listening for client connections on 127.0.0.1:9100
[80] [INF] Server is ready
⇨ http server started on /var/run/nms/core.sock
⇨ http server started on /var/run/nms/dpm.sock
```

# b. Ansible の実行
## 0. 事前作業

予め作業ホストから対象のホストに対してSSHの接続ができること。SSH公開鍵認証が望ましい
対象ホストにAnsibleがインストールされていること

## 1. Simple NIM の Docker Image Build

```
cd ~/f5j-nms-docker-simple/ansible
cp ~/nginx-repo* .
ansible-playbook -i inventory/hosts -l host1 nms/nms-setup.yaml
```

コマンドの実行が完了すると以下のような出力結果となります

```
PLAY RECAP ***************************************************************************************
10.1.1.5                  : ok=91   changed=44   unreachable=0    failed=0    skipped=50   rescued=0    ignored=0
```
