# Dataiku DSS with Docker Compose

## 起動方法

```bash
docker compose up
```

ブラウザで http://localhost:10000 にアクセスする。

## ディレクトリ構成

`./dss` ディレクトリがコンテナの `/home/dataiku/dss` にマウントされ、DSS のデータ・設定が永続化される。

## Dev Container について

このリポジトリは VS Code の Dev Container でも利用できる構成を想定しており、開発環境をコンテナ内で整えて利用することができます。

## Apple Silicon (arm64) での注意点

### Unix ドメインソケットの問題

`dataiku/dss` イメージは `linux/amd64` のみ提供されているため、Apple Silicon Mac では Rosetta 2 エミュレーションで動作する。

このとき、supervisord が Unix ドメインソケット (`svd.sock`) をマウントされたボリューム内に作成しようとすると、macOS の Docker ボリュームマウントが Unix ソケットをサポートしていないため、以下のエラーが発生する:

```
Error: Cannot open an HTTP server: socket.error reported errno.EINVAL (22)
```

**対処法:** 初回起動後に生成される `dss/install-support/supervisord.conf` のソケットパスをコンテナ内の `/tmp/` に変更する。

```ini
[unix_http_server]
file = /tmp/svd.sock   # /home/dataiku/dss/run/svd.sock から変更

[supervisorctl]
serverurl = unix:///tmp/svd.sock   # 同上
```

再インストールになどにこのファイルが上書きされた場合は再度同様の変更が必要。


## ライセンス

このリポジトリの内容は MIT ライセンスのもとで提供されており、ライセンス条件に従って利用・改変・再配布できます。
