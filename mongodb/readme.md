# Docker上のMongoDB環境を構築する方法（Docker Composeを使用）

## 目的
- Docker Composeを使用して、Windows 10 Pro/EntのDocker上で動作するMongoDB環境を構築する。

## 参考
- DockerHubにあるMongoのDocker Official Imagesのページ
  - https://hub.docker.com/_/mongo

## 前提条件
- Docker for Windowsを導入していること。
  - 導入方法は以下を参照
  - 「[Windows 10 Pro/EntでDockerを動かす方法](https://github.com/yamadakou/Docker-Learn/tree/master/How-to-Docker-on-Windows10)」
    - https://github.com/yamadakou/Docker-Learn/tree/master/How-to-Docker-on-Windows10

## Docker Compose
- 複数のコンテナを使う Dockerアプリケーションを、定義・実行するツール。
- アプリケーションのサービスの設定に、YAMLで記述可能な Composeファイルを使う。
- コマンドを１つ実行するだけで、DockerファイルとComposeファイルに設定した全てのサービスを作成・起動することができる。
  - 1つのコンテナの場合も利用できる。
- Docker Desctopには、Docker Composeが同梱されているため、追加のインストールは不要。
- 詳細はDocker Composeのドキュメントを参照
  - https://docs.docker.com/compose/
  - 日本語訳
    - http://docs.docker.jp/compose/toc.html

## Docker ComposeでMongoDB環境を構築
### MongoDBの動作フォルダを作成
- 動作フォルダとして、任意のフォルダを作成する。

### MongoDBのComposeファイルを作成
- 動作フォルダに「docker-compose.yaml」を作成する。
- 「docker-compose.yaml」にDockerコンテナを利用したサービスの設定を記述する。
  - SQL Server 2019 RC1のComposeファイル「docker-compose.yaml」
    ```yaml
    # Use root/example as user/password credentials
    version: '3.1'

    services:

      mongo:
        image: mongo
        restart: always
        environment:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example
        ports:
          - 27017:27017
        volumes:
          - ./db:/data/db
          - ./configdb:/data/configdb

      mongo-express:
        image: mongo-express
        restart: always
        ports:
          - 8081:8081
        environment:
          ME_CONFIG_MONGODB_ADMINUSERNAME: root
          ME_CONFIG_MONGODB_ADMINPASSWORD: example
    ```

### 作成したComposeファイルでDocker Composeを起動
```shell
$ docker-compose up

# 稼働状況を確認（別コマンドプロンプトで実行）
$ docker-compose ps
```
- エラーが発生し、リトライを繰り返す場合は以下の手順を試す。
  - コンテナを停止する。（別コマンドプロンプトで実行）
    ```shell
    $ docker-compose stop
    ```
  - 「docker-compose.yaml」の `volumes:` の内容をコメントアウトする。
    ```yaml
    ・・・
      mongo:
        image: mongo
        restart: always
        environment:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example
        ports:
          - 27017:27017
        #volumes:
        #  - ./db:/data/db
        #  - ./configdb:/data/configdb

      mongo-express:
    ・・・
    ```
  - Docker Composeを起動する。
    ```shell
    $ docker-compose up

    # 稼働状況を確認（別コマンドプロンプトで実行）
    $ docker-compose ps
    ```
  - エラーが発生しなければ、再度コンテナを停止する。（別コマンドプロンプトで実行）
    ```shell
    $ docker-compose stop
    ```
  - 「docker-compose.yaml」の `volumes:` のコメントアウトを戻す。
    ```yaml
    ・・・
      mongo:
        image: mongo
        restart: always
        environment:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example
        ports:
          - 27017:27017
        volumes:
          - ./db:/data/db
          - ./configdb:/data/configdb

      mongo-express:
    ・・・
    ```
  - Docker Composeを起動する。
    ```shell
    $ docker-compose up

    # 稼働状況を確認（別コマンドプロンプトで実行）
    $ docker-compose ps
    ```

  - ファイルシステム（NTFS）の問題のようなので、解消しない場合は以下の記事を参考にしてみてください。
    - https://denor.jp/docker-compose%E3%81%A7mongodb%E3%81%A8mongo-express%E3%82%92%E8%B5%B7%E5%8B%95%E3%81%99%E3%82%8B%E3%81%AB%E3%81%AF

- ブラウザで `localhost:8081` にアクセスし、mongo-expressで操作可能です。
  - mongo-expressについての参考
    - https://github.com/mongo-express/mongo-express
  - Advanced 検索窓 使い方の参考
    - https://qiita.com/koshilife/items/53875d4fc5a7a0f0a3ea

### Docker Composeで起動したMongoDB環境を停止
```shell
$ docker-compose stop

# 稼働状況を確認
$ docker-compose ps
```

### Docker Composeで停止した起動したMongoDB環境を再開
```
$ docker-compose start

# 稼働状況を確認
$ docker-compose ps
```
