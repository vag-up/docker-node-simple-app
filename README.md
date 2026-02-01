# Docker環境構築（Node.js + MariaDB + Nginx + Redis）

Vagrantを使用してUbuntu 24.04上にDocker環境を自動構築するプロジェクトです。
Ansibleを使用して必要なソフトウェアのインストールと設定を行い、Node.js アプリケーションを実行するための環境を構築します。

サンプルとして、シンプルな Express アプリケーション（`playbooks/app`）が含まれています。

## 環境構成

- **OS**: Ubuntu 24.04 (bento/ubuntu-24.04)
- **Docker**: geerlingguy.docker ロールによるインストール
- **タイムゾーン**: Asia/Tokyo
- **SSL/TLS**: 自己署名証明書によるHTTPS接続

### Dockerコンテナ構成

| コンテナ | 説明 |
|---------|------|
| Node.js | Node.js v24 アプリケーションサーバー |
| Nginx | リバースプロキシ/Webサーバー（HTTPS対応） |
| MariaDB | データベースサーバー |
| Redis | キャッシュサーバー |

## 前提条件

- [VirtualBox](https://www.virtualbox.org/) がインストールされていること
- [Vagrant](https://www.vagrantup.com/) がインストールされていること
- 十分なメモリ（最低2GB）と空きディスク容量があること

## ディレクトリ構成

```
.
├── README.md             # このファイル
├── Vagrantfile           # Vagrant設定ファイル
└── playbooks/            # Ansibleプレイブック
    ├── main.yml              # メインプレイブック
    ├── requirements.yml      # 必要なロールの定義
    ├── app/                  # Node.jsアプリケーション（Express サンプル）
    │   ├── package.json          # 依存関係定義
    │   ├── README.md             # アプリケーション説明
    │   └── src/
    │       └── index.js          # メインアプリケーション
    ├── vars/                 # 変数定義
    │   └── main.yml              # メイン変数ファイル
    ├── tasks/                # タスク定義
    │   ├── japanese.yml          # 日本語環境設定
    │   ├── redis.yml             # Redisコンテナ構築
    │   ├── mariadb.yml           # MariaDBコンテナ構築
    │   ├── nginx.yml             # Nginxコンテナ構築
    │   ├── nodejs.yml            # Node.jsコンテナ構築
    │   └── app.yml               # アプリケーションデプロイ
    └── containers/           # コンテナ設定
        ├── mariadb/              # MariaDB用Dockerfile等
        ├── nginx/                # Nginx用設定ファイル等
        ├── nodejs/               # Node.js用Dockerfile等
        └── redis/                # Redis用Dockerfile等
```

## IPアドレス設定

仮想マシンには固定IPアドレス `192.168.33.10` が設定されます。
必要に応じて `Vagrantfile` の `config.vm.network` の設定を変更してください。

## 起動手順

1. リポジトリをクローンする
2. 以下のコマンドを実行してVirtualBox上に環境を構築

```bash
vagrant up
```

## 接続方法

環境構築後、以下のいずれかの方法で仮想マシンに接続できます。

1. Vagrantから接続:
```bash
vagrant ssh
```

2. SSHで直接接続:
```bash
ssh vagrant@192.168.33.10
```
※デフォルトパスワード: vagrant

### rootユーザーへの切り替え
接続後、以下のコマンドでrootユーザーに切り替えることができます。
```bash
sudo su -
```

## Dockerコンテナの確認

環境構築後、仮想マシン内で以下のコマンドでコンテナの状態を確認できます。

```bash
docker ps                    # 実行中のコンテナ一覧
docker logs nodejs           # Node.jsコンテナのログ
docker logs nginx            # Nginxコンテナのログ
docker logs mariadb          # MariaDBコンテナのログ
docker logs redis            # Redisコンテナのログ
```

## データベース設定

MariaDBの初期設定は以下の通りです（`playbooks/vars/main.yml` で変更可能）:

| 項目 | 値 |
|------|-----|
| rootパスワード | root_password |
| データベース名 | sample-db |
| ユーザー名 | sample_user |
| パスワード | sample_password |

## カスタマイズ

- `playbooks/vars/main.yml` を編集することで、コンテナ名やDB設定を変更できます
- `playbooks/main.yml` を編集することで、追加のパッケージやタスクを追加できます
- `Vagrantfile` の `vb.memory` を編集して、仮想マシンのメモリ割り当てを変更できます

## SSL/HTTPS接続

この環境では、NginxがHTTPS（ポート443）でリッスンし、自己署名SSL証明書を使用してセキュアな接続を提供します。

### 証明書の詳細

- **証明書タイプ**: 自己署名証明書（OpenSSLで自動生成）
- **有効期限**: 365日
- **鍵長**: RSA 4096ビット
- **対応プロトコル**: TLS 1.2, TLS 1.3

### HTTPからHTTPSへの自動リダイレクト

HTTPポート（80）へのアクセスは自動的にHTTPS（443）にリダイケクトされます。

### ブラウザからのアクセス時の注意

自己署名証明書を使用しているため、ブラウザで初回アクセス時に「安全ではありません」という警告が表示されます。
これは開発環境では正常な動作です。以下の手順で続行できます：

1. ブラウザで `https://192.168.33.10` にアクセス
2. 「詳細設定」または「Advanced」をクリック
3. 「安全ではないページに移動」または「Proceed to 192.168.33.10 (unsafe)」をクリック

### curlコマンドでのアクセス

`curl` コマンドを使用する場合は、`-k`（または `--insecure`）オプションを付けることで、自己署名証明書を許可できます。

## アプリケーション

`playbooks/app` ディレクトリに Express を使用したシンプルなサンプルアプリケーションが含まれています。

### 提供されるエンドポイント

- `GET /` - アプリケーション情報
- `GET /health` - ヘルスチェック
- `GET /hello/:name` - 挨拶メッセージ
- `POST /echo` - JSONエコーバック

### アプリケーションへのアクセス

環境構築後、以下のURLでアクセスできます（自己署名証明書を使用しているため `-k` オプションが必要です）：

```bash
# ヘルスチェック
curl -k https://192.168.33.10/health

# Hello エンドポイント
curl -k https://192.168.33.10/hello/World

# Echo エンドポイント
curl -k -X POST https://192.168.33.10/echo \
  -H "Content-Type: application/json" \
  -d '{"message":"test"}'
```

**注意**: `-k` オプションは自己署名証明書を許可するために必要です。本番環境では使用しないでください。

詳細は [`playbooks/app/README.md`](playbooks/app/README.md) を参照してください。

### 独自のアプリケーションを使用する場合

`playbooks/app` ディレクトリの内容を置き換えるか、以下の要件を満たすアプリケーションを配置してください：

- `package.json` に `start` スクリプトが定義されていること
- ポート 3000 でリッスンすること
- pnpm を使用すること

## トラブルシューティング

### ネットワーク接続の問題
ネットワーク設定に問題が発生した場合は、`Vagrantfile` の IPアドレスを
使用環境に合わせて変更してください。

### 仮想マシンの起動に失敗する場合
VirtualBoxの設定や競合を確認し、必要に応じてVirtualBoxを再起動してください。

### プロビジョニングを再実行する場合
```bash
vagrant provision
```
