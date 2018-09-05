## AWSのEC2でDockerを使ってWordPressを立ち上げる。

#### AWS EC2の立ち上げ
コンソール上からインスタンスを立ち上げ
OSはAWSのLinuxを利用
下記コマンドでインスタンスにログイン

```
$ ssh -i "key-pair" ec2-user@<<public ip or public DNS>>
```
#### Dockerのインストール
インストールと起動

```
$ sudo apt-get update
$ sudo apt-get install docker.io
$ sudo service docker start
$ sudo service docker status
 ● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enab
   Active: active (running) since Wed 2018-09-05 07:39:42 UTC; 3h 53min ago
     Docs: https://docs.docker.com
 Main PID: 3125 (dockerd)
    Tasks: 49
   Memory: 101.9M
      CPU: 50.274s
   CGroup: /system.slice/docker.service
 
 ```
   
#### dockerグループにec2-userを追加。

```
$ sudo usermod -a -G docker $User
$ cat /etc/group |grep docker
docker:x:116:

```

#### docker-composeのインストール。

公式ドキュメントにある通りcurlでインストール

```
$ curl -L "https://github.com/docker/compose/releases/download/1.9.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose --version
docker-compose version 1.9.0, build 2585387
```

#### WordPressを立ち上げるため、Dockerfileを作成。

```
# Dockerイメージの取得
FROM busybox

# 作成者情報
MAINTAINER 0.1 your-account@your-domain.com

# データの設定
VOLUME /var/lib/mysql
```

#### コンテナの生成
```
$ docker build -t wordpress .
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM busybox
latest: Pulling from library/busybox

fdab12439263: Pull complete 
Digest: sha256:f102731ae8898217038060081c205aa3a4ae3f910c2aaa7b3adeb6da9841d963
Status: Downloaded newer image for busybox:latestf
 ---> 1efc1d465fd6
Step 2 : MAINTAINER 0.1 your-account@your-domain.com
 ---> Running in 0a251281984a
 ---> 55a5492c2587
Removing intermediate container 0a251281984a
Step 3 : VOLUME /var/lib/mysql
 ---> Running in 67ec86d59105
 ---> b5937066eefa
Removing intermediate container 67ec86d59105
Successfully built b5937066eefa

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
wordpress           latest              b5937066eefa        About a minute ago   1.095 MB
busybox             latest              1efc1d465fd6        2 weeks ago          1.095 MB
```

#### コンテナの起動

```
$ docker run -it --name wordpress wordpress
/ # lsf
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # exit

$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
3fb730b66d6e        wordpress           "sh"                36 seconds ago      Exited (0) 26 seconds ago                       data-only
```

#### WebサーバとDBサーバのコンテナ作成。

WordPress構築のため、下記2つのコンテナを生成します。

Webサーバコンテナ (web-serverコンテナ)
WordPressイメージを使うが、内部的にはApacheとPHPで動作している
DBサーバコンテナ (db-serverコンテナ)
MySQLイメージを使う
Webサーバからのデータ処理を受け付ける

#### docker-composerの設定

#### docker-compose.ymlの作成。


#### コンテナの起動。

```
$ docker-compose up -d

$ docker-compose ps


$ docker start -ia wordpress
/ # ls /var/lib/mysql
auto.cnf            ib_logfile0         ibdata1             mysql               sys
ib_buffer_pool      ib_logfile1         ibtmp1              performance_schema  wordpress
/ # exit
```
#### コンテナの稼働状況確認

```
$ docker-compose ps
       Name                      Command               State         Ports        
---------------------------------------------------------------------------------
docker_db-server_1    docker-entrypoint.sh mysqld      Up      3306/tcp           
docker_web-server_1   docker-entrypoint.sh apach ...   Up      0.0.0.0:80->80/tcp

```

#### コンテナでのコマンド実行例

```
$ docker-compose run web-server /bin/bash
root@968b067b5b2d:/var/www/html# pwd
/var/www/html

```

#### コンテナ一括停止とリスタートコマンド/コンテナ一括削除

```
$ docker-compose stop
$ docker-compose restart
$ docker-compose rm
```
#### バックアップ

$ docker export data-only > backup.tar

#### リストア
$ tar xvf backup.tar


