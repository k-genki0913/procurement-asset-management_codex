# サーバーの開発環境

## 今回できること

Java 21・Spring Boot 4.1.1のアプリをコンテナで起動し、Actuatorで起動状態を確認します。業務API、画面、DB接続、認証、自動テスト基盤はまだありません。

Dev Containers、PostgreSQL、SPAはそれぞれ次のPRで追加します。各PRがマージされるまで次の実装は進めません。

## 構成と依存

| 設定 | 内容・用途 |
| --- | --- |
| Java | Eclipse Temurin 21.0.12+8。ARM64対応イメージのダイジェストをDockerfileで固定 |
| Spring Boot | 4.1.1。親POMで依存・ビルドプラグインのバージョンを管理 |
| Maven | 3.9.11。公式Maven Wrapper 3.3.4のスクリプトで取得・実行 |
| `spring-boot-starter-webmvc` | 組み込みWebサーバーとSpring MVCを提供 |
| `spring-boot-starter-actuator` | `/actuator/health`で起動状態を提供 |
| Spring Boot Maven Plugin | 起動と実行可能JARの生成 |

直接のアプリ依存は上記スターター2つのみです。スターターが必要とする推移的依存はMavenが取得します。将来の機能のための依存は追加していません。

Maven WrapperスクリプトはApache公式配布物から取得したものです。通常は編集不要です。

Spring Bootの設定は`backend/src/main/resources/application.yaml`に統一します。

`compose.yaml`はプロジェクトのルートに置き、Codex・ターミナル・VS Codeで共通の起動構成として使用します。次のPRでは`.devcontainer/devcontainer.json`からこのComposeを参照し、移動や複製はしません。Dev Containers専用の追加設定が必要な場合のみ、`.devcontainer/`に上書き用Composeを置きます。

## 前提

- M1 Mac、稼働中のColima（Docker runtime）、Docker CLI、Docker Compose。
- リポジトリのルートで以下のコマンドを実行します。
- `docker context show`が`colima`であり、`docker info`でサーバー情報が取得できることを確認します。Colimaが未起動なら`colima start`を実行します。
- 初回はイメージ・Maven・依存ライブラリを取得するため、インターネット接続が必要です。
- MacへJavaやMavenを追加インストールする必要はありません。
- 今回は環境変数の入力が不要なので、`.env`やその見本は作成しません。必要になるPRで追加します。

## ビルドと起動

```sh
docker compose build
docker compose run --rm backend ./mvnw --batch-mode package
docker compose up -d
docker compose logs -f backend
```

初回の起動は依存ダウンロードに時間がかかります。ログの`Started ProcurementApplication`を確認します。ログ追跡はCtrl+Cで終了できます（コンテナは停止しません）。

ブラウザで <http://localhost:18081/actuator/health> を開くか、次を実行します。

```sh
curl --fail --include http://127.0.0.1:18081/actuator/health
```

HTTP 200と`{"status":"UP"}`が返れば成功です。DBは接続していないため、DB接続確認ではありません。公開するActuatorエンドポイントは`health`のみで、詳細情報とエンドポイント一覧は非公開です。`/`には画面を用意していないため404になります。

## 編集・停止・再起動

`backend/`のソースはコンテナへ共有されます。Javaや設定ファイルの変更後は次の操作で再コンパイル・再起動します。自動リロードは未導入です。

```sh
docker compose restart backend
```

停止と起動は以下のとおりです。

```sh
docker compose stop backend
docker compose start backend
```

コンテナと専用ネットワークを片付ける場合は次を実行します。Mavenキャッシュは残ります。

```sh
docker compose down
```

再作成は`docker compose up -d --build`です。通常の停止に`--volumes`や`-v`は付けません。Dockerfile変更時は再ビルドが必要です。

## 確認とトラブルシューティング

```sh
docker compose ps
docker compose logs --tail=100 backend
docker compose exec backend java -version
docker compose exec backend ./mvnw --version
```

- `18081`が使用中なら既存のプロセスを停止せず、競合を確認してください。既存の`spring-verify`の`18080`とは分離しています。
- パッケージ作成失敗時はMavenのエラーを確認し、ネットワーク障害なら回復後に同じコマンドを再実行します。
- コンテナは非rootユーザーで動作します。Mavenキャッシュは専用ボリューム、`target/`は共有ソース内の生成物です。`target/`はGit対象外です。
- ポートは`127.0.0.1`に限定し、ローカル開発専用です。本番向けの配布イメージではありません。
- 自動テストはまだありません。`package`成功はコンパイル・JAR生成の確認であり、業務動作や自動テストの合格を意味しません。

## レビューする順序

1. `backend/pom.xml`：Java・Bootのバージョンと直接依存2つ。
2. `backend/src/main/`：起動クラスとActuator公開範囲。
3. `backend/Dockerfile`と`compose.yaml`：非root実行、固定イメージ、ローカルポート、共有フォルダとキャッシュ。

MacのVS Codeで閲覧できます。コンテナ内のJavaを使ったコード解析・デバッグ設定は次のPRで追加します。
