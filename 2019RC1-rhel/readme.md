# Docker上のSQL Server 2019 RC1（linux版）環境を構築

## 目的
- Windows10でのDocker環境と、Docker上で動作するlinux版のSQL Server 2019 RC1環境を構築する。

## 参考
- MSドキュメント「Docker 上で SQL Server コンテナー イメージを構成する」の「RHEL ベースのコンテナー イメージを実行する」
  - https://docs.microsoft.com/ja-jp/sql/linux/sql-server-linux-configure-docker?view=sql-server-linux-ver15#rhel

- Dockerドキュメント「WindowsにDocker Desktopをインストールする」
  - https://docs.docker.com/docker-for-windows/install/

- 「AdventureWorks2017.bak」の入手先
  - MS Doc
    - https://docs.microsoft.com/ja-jp/sql/samples/adventureworks-install-configure?view=sql-server-2017
  - GitHub
    - https://github.com/Microsoft/sql-server-samples/releases/tag/adventureworks

## Dockerのインストール
### 事前準備
#### CPU仮想化の確認
- CPUの仮想化が有効になっているか確認する。
  - タスクマネージャーの「パフォーマンス」タブにある「CPU」画面にある「仮想化」が有効となっているか確認
  - 無効の場合、各PCのマニュアルに従いBIOSからCPU仮想化が有効になるよう設定変更を行う。

#### Hyper-Vを有効化
- MSドキュメントに従いHyper-Vを有効化する。
  - MSドキュメント「Windows 10 上に Hyper-V をインストールする」
    - https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v

- 3つの方法があるので、いずれかの方法で、Hyper-Vをインストールする。
  - [PowerShell を使用して Hyper-V を有効にする](https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v#enable-hyper-v-using-powershell)
  - [CMD と DISM を使用して Hyper-V を有効にする](https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v#enable-hyper-v-with-cmd-and-dism)
  - [[設定] で Hyper-V ロールを有効にする](https://docs.microsoft.com/ja-jp/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v#enable-the-hyper-v-role-through-settings)


### Docker for Windows のインストール
#### Docker for Windowsのダウンロード
- Dockerドキュメント「Install Docker Desktop on Windows」の「Download from Docker Hub」ボタンから遷移した[Docker Hub](https://hub.docker.com/)でアカウント登録後、ダウンロードできる。
- ダウンロードしたインストーラを起動し、ウィザードに従ってインストールを行う。

#### バージョン確認
- コマンドプロンプトやPowerShellからDockerコマンドを実行してバージョンを確認する。
- Dockerのバージョン確認
```
$ docker version
``` 
- 結果（コマンドプロントからの例）
```
c:\>docker version
Client: Docker Engine - Community
 Version:           19.03.2
 API version:       1.40
 Go version:        go1.12.8
 Git commit:        6a30dfc
 Built:             Thu Aug 29 05:26:49 2019
 OS/Arch:           windows/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.2
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.8
  Git commit:       6a30dfc
  Built:            Thu Aug 29 05:32:21 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.2.6
  GitCommit:        894b81a4b802e4eb2a91d1ce216b8817763c29fb
 runc:
  Version:          1.0.0-rc8
  GitCommit:        425e105d5a03fabd737a126ad93d62a9eeede87f
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

## Dockerの基本操作
- Dockerドキュメント（基本コマンド）
  - https://docs.docker.com/engine/reference/commandline/docker/

###  イメージの検索/取得
#### [docker search](https://docs.docker.com/engine/reference/commandline/search/)
- [DockerHub](https://hub.docker.com/)に登録されたDockerイメージを検索する。
- 例：RedisのDockerイメージを検索
```
c:\>docker search redis
NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
redis                            Redis is an open source key-value store that…   7322                [OK]              
bitnami/redis                    Bitnami Redis Docker Image                      127                                     [OK]
sameersbn/redis                                                                  77                                      [OK]
grokzen/redis-cluster            Redis cluster 3.0, 3.2, 4.0 & 5.0               56                                     
rediscommander/redis-commander   Alpine image for redis-commander - Redis man…   31                                      [OK]
kubeguide/redis-master           redis-master with "Hello World!"                29                                     
redislabs/redis                  Clustered in-memory database engine compatib…   23                                    
arm32v7/redis                    Redis is an open source key-value store that…   17                                    
redislabs/redisearch             Redis With the RedisSearch module pre-loaded…   17                                    
oliver006/redis_exporter          Prometheus Exporter for Redis Metrics. Supp…   14                                    
webhippie/redis                  Docker images for Redis                         10                                      [OK]
s7anley/redis-sentinel-docker    Redis Sentinel                                  9                                       [OK]
redislabs/redisgraph             A graph database module for Redis               8                                       [OK]
insready/redis-stat              Docker image for the real-time Redis monitor…   8                                       [OK]
bitnami/redis-sentinel           Bitnami Docker Image for Redis Sentinel         6                                       [OK]
arm64v8/redis                    Redis is an open source key-value store that…   6                                     
centos/redis-32-centos7          Redis in-memory data structure store, used a…   4                                     
redislabs/redismod               An automated build of redismod - latest Redi…   4                                       [OK]
circleci/redis                   CircleCI images for Redis                       2                                       [OK]
frodenas/redis                   A Docker Image for Redis                        2                                       [OK]
tiredofit/redis                  Redis Server w/ Zabbix monitoring and S6 Ove…   1                                       [OK]
runnable/redis-stunnel           stunnel to redis provided by linking contain…   1                                       [OK]
wodby/redis                      Redis container image with orchestration        1                                       [OK]
xetamus/redis-resource           forked redis-resource                           0                                       [OK]
cflondonservices/redis           Docker image for running redis                  0                                      
```
- 1番目の「OFFICIAL」が「[OK]」となっており、Docker公式のRedis組込み済みイメージであることが分かる。

#### [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [DockerHub](https://hub.docker.com/)や他のレジストリに登録されたDockerイメージを取得（ダウンロード）する。
- 例：[Docker公式のRedis組込み済みイメージ](https://hub.docker.com/_/redis)を取得
```
c:\>docker pull redis
Using default tag: latest
latest: Pulling from library/redis
b8f262c62ec6: Pull complete
93789b5343a5: Pull complete
49cdbb315637: Pull complete
2c1ff453e5c9: Pull complete
9341ee0a5d4a: Pull complete 
770829e1df34: Pull complete
Digest: sha256:5dcccb533dc0deacce4a02fe9035134576368452db0b4323b98a4b2ba2d3b302
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
```
- バージョンを指定しないとデフォルトであるLatestバージョンを取得する。

#### [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- Dockerコンテナの一覧を表示する。
- 例
```
c:\>docker images
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
redis                                      latest              63130206b0fa        5 days ago          98.2MB
mcr.microsoft.com/mssql/rhel/server        2019-RC1            e3be04ae2efd        4 weeks ago         1.82GB
mcr.microsoft.com/mssql/server             2017-latest         644ca19cb10d        6 months ago        1.38GB
hello-world                                latest              fce289e99eb9        8 months ago        1.84kB
microsoft/mssql-server-linux               2017-latest         314918ddaedf        9 months ago        1.35GB
microsoft/mssql-server-linux               latest              314918ddaedf        9 months ago        1.35GB
k8s.gcr.io/kube-proxy-amd64                v1.10.11            7387003276ac        9 months ago        98.3MB
k8s.gcr.io/kube-apiserver-amd64            v1.10.11            e851a7aeb6e8        9 months ago        228MB
k8s.gcr.io/kube-controller-manager-amd64   v1.10.11            978cfa2028bf        9 months ago        151MB
k8s.gcr.io/kube-scheduler-amd64            v1.10.11            d2c751d562c6        9 months ago        51.2MB
docker/kube-compose-controller             v0.4.12             02a45592fbea        12 months ago       27.8MB
docker/kube-compose-api-server             v0.4.12             0f92c77fa676        12 months ago       41.2MB
k8s.gcr.io/etcd-amd64                      3.1.12              52920ad46f5b        18 months ago       193MB
k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64     1.14.8              c2ce1ffb51ed        20 months ago       41MB
k8s.gcr.io/k8s-dns-sidecar-amd64           1.14.8              6f7f2dc7fab5        20 months ago       42.2MB
k8s.gcr.io/k8s-dns-kube-dns-amd64          1.14.8              80cc5ea4b547        20 months ago       50.5MB
k8s.gcr.io/pause-amd64                     3.1                 da86e6ba6ca1        21 months ago       742kB
```
- 取得したコンテナを確認できる。

#### まとめ
- `docker search` でDickerイメージを検索して、
- `docker pull` でDockerイメージを取得して、
- `docker images` でDockerイメージが取得できていることを確認する。


### コンテナの起動・プロセスの確認・停止
#### [docker run](https://docs.docker.com/engine/reference/commandline/run/)
- コンテナを起動する。
  - `-i` と `-t` のオプションを付けるとターミナルでコンテナを実行することができる。
    - `exit` を入力するとターミナルを終了し、ホストターミナルに戻るが、ターミナルを終了した場合、コンテナも停止する。
    - コンテナを停止せずにターミナルを抜けるには `CTRL + p + q` で抜ける。
- 例：[Docker公式のRedis組込み済みイメージ](https://hub.docker.com/_/redis)をDockerコンテナで起動
```
c:\>docker run --name docker-redis -d -p 6379:6379 redis
2c7b52aaf216d93cc9c727ce8294c80dba375edd53d55a2ac80de7fb238d2d08
```

#### [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)
- コンテナの一覧を表示する。
  - オプションを指定しない場合、稼働中のコンテナのみ表示する。
- 例：稼働中のコンテナ一覧で、[Docker公式のRedis](https://hub.docker.com/_/redis)が起動しているか確認
```
c:\>docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
2c7b52aaf216        redis               "docker-entrypoint.s…"   48 seconds ago      Up 47 seconds       0.0.0.0:6379->6379/tcp   docker-redis
```
- コンテナ起動時に指定した名前「docker-redis」でポート6379を使用して起動していることが分かる。
- 任意のRedisクライアントで `localhost:6379` に接続すると、コンテナ上のRedisに対する操作が可能。
  - Redisクライアントの例
    - [RDBTools](https://rdbtools.com/)の[Redis GUI Client for Windows](https://rdbtools.com/docs/install/windows/)

#### [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) / [docker kill](https://docs.docker.com/engine/reference/commandline/kill/)
- コンテナを停止する。
- docker stop
  - メインのコンテナ・プロセスに [`SIGTERM`](https://www.wdic.org/w/TECH/SIGTERM) を送信後、一定期間が経過したら [`SIGKILL`](https://www.wdic.org/w/TECH/SIGKILL) を送信。
  - 期間の指定は-f を使用。(デフォルトは10秒)
- docker kill
  - メインのコンテナ・プロセスに [`SIGKILL`](https://www.wdic.org/w/TECH/SIGKILL) を直ちに送信。
- 例：名前「docker-redis」で起動したコンテナを停止
```
c:\>docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
2c7b52aaf216        redis               "docker-entrypoint.s…"   48 seconds ago      Up 47 seconds       0.0.0.0:6379->6379/tcp   docker-redis

c:\>docker stop docker-redis
docker-redis

c:\>docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
- `docker ps` で名前「docker-redis」のコンテナが停止したことが分かる。

#### [docker start](https://docs.docker.com/engine/reference/commandline/start/)
- 停止したコンテナを再起動する。
- コンテナは以前に指定したものと同じオプションで起動。
  - `-a` でアタッチすることが可能。
- 例：停止した名前「docker-redis」のコンテナを再起動する。
```
c:\>docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

c:\>docker start docker-redis
docker-redis

c:\>docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
2c7b52aaf216        redis               "docker-entrypoint.s…"   27 minutes ago      Up 2 seconds        0.0.0.0:6379->6379/tcp   docker-redis
```
- `docker ps` で名前「docker-redis」のコンテナが起動したことが分かる。

#### まとめ
- `docker run` でDickerコンテナを起動して、
- `docker stop` でDickerコンテナを停止して、
- `docker kill` でDickerコンテナを強制停止して、
- `docker start` でDickerコンテナを再起動して、
- `docker ps` でDockerコンテナの稼働状況を確認する。

### コンテナの削除・イメージの削除
#### [docker rm](https://docs.docker.com/engine/reference/commandline/rm/)
- 動作しているコンテナを確認
```
c:\>docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

- 停止しているコンテナを確認
```
c:\>docker ps -a
CONTAINER ID        IMAGE                                        COMMAND                  CREATED             STATUS                          PORTS               NAMES
73aa6a0a9dc2        redis                                        "docker-entrypoint.s…"   12 hours ago        Exited (0) About a minute ago                       docker-redis
1456069ce688        mcr.microsoft.com/mssql/server:2017-latest   "/opt/mssql/bin/sqls…"   6 months ago        Created                                             sql1
```
- コンテナの削除
```
c:\>docker rm docker-redis
docker-redis

c:\>docker ps -a
CONTAINER ID        IMAGE                                        COMMAND                  CREATED             STATUS              PORTS               NAMES
1456069ce688        mcr.microsoft.com/mssql/server:2017-latest   "/opt/mssql/bin/sqls…"   6 months ago        Created                                 sql1
```
- 削除した「docker-redis」のコンテナが、停止しているコンテナから削除されたことが分かる。

#### [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- 現在のコンテナとイメージの確認
```
c:\>docker ps -a
CONTAINER ID        IMAGE                                        COMMAND                  CREATED             STATUS              PORTS               NAMES
1456069ce688        mcr.microsoft.com/mssql/server:2017-latest   "/opt/mssql/bin/sqls…"   6 months ago        Created                                 sql1

c:\>docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
redis                            latest              63130206b0fa        7 days ago          98.2MB
mcr.microsoft.com/mssql/server   2017-latest         314918ddaedf        9 months ago        1.35GB
docker4w/nsenter-dockerd         latest              2f1c802f322f        11 months ago       187kB
```
- 「mcr.microsoft.com/mssql/server」のイメージを削除（してみる）
```
c:\>docker rmi 314918ddaedf
Error response from daemon: conflict: unable to delete 314918ddaedf (must be forced) - image is being used by stopped container 1456069ce688
```
  - コンテナが使用している旨のエラーメッセージが表示され、削除できない。

- 「redis」のイメージを削除
```
c:\>docker rmi redis
Untagged: redis:latest
Untagged: redis@sha256:5dcccb533dc0deacce4a02fe9035134576368452db0b4323b98a4b2ba2d3b302
Deleted: sha256:63130206b0fa808e4545a0cb4a1f14f6d40b8a7e2e6fda0a31fd326c2ac0971c
Deleted: sha256:9476758634326bb436208264d0541e9a0d42e4add35d00c2a7408f810223013d
Deleted: sha256:0f3d9de16a216bfa5e2c2bd0e3c2ba83afec01a1b326d9f39a5ea7aecc112baf
Deleted: sha256:452d665d4efca3e6067c89a332c878437d250312719f9ea8fff8c0e350b6e471
Deleted: sha256:d6aec371927a9d4bfe4df4ee8e510624549fc08bc60871ce1f145997f49d4d37
Deleted: sha256:2957e0a13c30e89650dd6c00644c04aa87ce516284c76a67c4b32cbb877de178
Deleted: sha256:2db44bce66cde56fca25aeeb7d09dc924b748e3adfe58c9cc3eb2bd2f68a1b68

c:\>docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
mcr.microsoft.com/mssql/server   2017-latest         314918ddaedf        9 months ago        1.35GB
docker4w/nsenter-dockerd         latest              2f1c802f322f        11 months ago       187kB
```
  - コンテナが削除済みであれば、イメージも削除可能。

#### まとめ
- `docker rm` でDickerコンテナを削除して、
- `docker rmi` でDockerイメージを削除する。


## Docker Compose
- 複数のコンテナを使う Dockerアプリケーションを、定義・実行するツール。
- アプリケーションのサービスの設定に、YAMLで記述可能な Composeファイルを使う。
- コマンドを１つ実行するだけで、DockerファイルとComposeファイルに設定した全てのサービスを作成・起動することができる。
  - 1つのコンテナの場合も利用できる。
- Docker Desctopには、Docker Composeが同梱されているため、追加のインストールは不要。
- 詳細は、Docker Composeの日本語ドキュメントを参照
  - http://docs.docker.jp/compose/toc.html

## Docker ComposeでSQL Server 2019 RC1環境を構築
### SQL Server 2019 RC1の動作フォルダを作成
- 動作フォルダとして、任意のフォルダを作成する。

### SQL Server 2019 RC1のComposeファイルを作成
- 動作フォルダに「docker-compose.yaml」を作成する。
- 「docker-compose.yaml」にDockerコンテナを利用したサービスの設定を記述する。
  - SQL Server 2019 RC1のComposeファイル「docker-compose.yaml」
    ```
    version: '3'

    services:
    mssql:
        image: mcr.microsoft.com/mssql/rhel/server:2019-RC1
        container_name: 'mssql2019-rc1-rhel'
        environment:
        - MSSQL_SA_PASSWORD=<saユーザーのパスワード(SQL Serverのパスワードルールに従うこと）>
        - ACCEPT_EULA=Y
        - MSSQL_PID=Developer # default: Developer
        # - MSSQL_PID=Express
        # - MSSQL_PID=Standard
        # - MSSQL_PID=Enterprise
        # - MSSQL_PID=EnterpriseCore
        ports:
        - 1433:1433
        volumes: # Mounting a volume does not work on Docker for Mac
        - ./mssql/log:/var/opt/mssql/log
        - ./mssql/data:/var/opt/mssql/data
    ```

### 作成したComposeファイルでDocker Composeを起動
```
$ docker-compose up

# 稼働状況を確認
$ docker-compose ps
```
- SSMSなどDBクライアントツールで `localhost:1433` に接続することで操作可能

### Docker Composeで起動した起動したSQL Server 2019 RC1環境を停止
```
$ docker-compose stop

# 稼働状況を確認
$ docker-compose ps
```
