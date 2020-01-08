# Docker上のSQL Server 2019 （linux版）環境を構築する方法（Docker Composeを使用）

## 目的
- Docker Composeを使用して、Windows 10 Pro/EntのDocker上で動作するlinux版のSQL Server 2019 環境を構築する方法を学習する。

## 参考
- MSドキュメント「Docker 上で SQL Server コンテナー イメージを構成する」の「RHEL ベースのコンテナー イメージを実行する」
  - https://docs.microsoft.com/ja-jp/sql/linux/sql-server-linux-configure-docker?view=sql-server-linux-ver15#rhel

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

## Docker ComposeでSQL Server 2019 環境を構築
### SQL Server 2019 の動作フォルダを作成
- 動作フォルダとして、任意のフォルダを作成する。

### SQL Server 2019 のComposeファイルを作成
- 動作フォルダに「docker-compose.yaml」を作成する。
- 「docker-compose.yaml」にDockerコンテナを利用したサービスの設定を記述する。
  - SQL Server 2019 のComposeファイル「docker-compose.yaml」
    ```
    version: '3'

    services:
      mssql:
          image: mcr.microsoft.com/mssql/server:2019-GA-ubuntu-16.04
          container_name: 'mssql2019-GA-ubuntu'
        environment:
          - MSSQL_SA_PASSWORD=<saユーザーのパスワード(SQL Serverのパスワードルールに従うこと）>
          - ACCEPT_EULA=Y
          - MSSQL_PID=Developer # default: Developer
          # - MSSQL_PID=Express
          # - MSSQL_PID=Standard
          # - MSSQL_PID=Enterprise
          # - MSSQL_PID=EnterpriseCore
        ports:
          - 14331:1433 # <ホスト側ポート番号>:<コンテナ側ポート番号>
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
- SSMSなどDBクライアントツールで `localhost,14331` に接続することで操作可能
  - 接続方法などの参考
    - https://docs.microsoft.com/ja-jp/sql/linux/sql-server-linux-configure-docker?view=sql-server-linux-ver15#connect-and-query
  - SSMS（SQL Server Management Studio）
    - 以下からダウンロード可能
      - https://docs.microsoft.com/ja-jp/sql/ssms/download-sql-server-management-studio-ssms
  - ADS（Azure Data Studio）
    - macOSやLinuxにも対応したMS製DBツール
    - 以下からダウンロード可能
      - https://docs.microsoft.com/ja-jp/sql/azure-data-studio/what-is

### Docker Composeで起動したSQL Server 2019 環境を停止
```
$ docker-compose stop

# 稼働状況を確認
$ docker-compose ps
```

### Docker Composeで停止した起動したSQL Server 2019 環境を再開
```
$ docker-compose start

# 稼働状況を確認
$ docker-compose ps
```
### DBの復元（SSMSを使用）
- ダウンロードした「AdventureWorks2017.bak」ファイルを、Docker Compose の起動ディレクトリ配下の `\mssql\data` フォルダ内に配置
  - 「AdventureWorks2017.bak」の入手先
    - MS Doc
      - https://docs.microsoft.com/ja-jp/sql/samples/adventureworks-install-configure?view=sql-server-2017
    - GitHub
      - https://github.com/Microsoft/sql-server-samples/releases/tag/adventureworks
- SSMSで `localhost,14331` に接続する。
- 以下を参考に、SSMSでオブジェクトエクスプローラーの「データベース」を右クリックしたメニューから「データベースの復元」をクリックし、「データベースの復元ダイアログ」にて「AdventureWorks2017.bak」を復元する。
  - https://docs.microsoft.com/ja-jp/sql/relational-databases/backup-restore/restore-a-database-to-a-new-location-sql-server
- 復元したDB「AdventureWorks2017」に `SELECT` などで動作を確認する。
