# サーバーとDBの開発環境

## 今回できること

Java 21・Spring Boot 4.1.1のアプリをコンテナで起動し、Actuatorで起動状態を確認します。業務API、画面、DB接続、認証、自動テスト基盤はまだありません。

Dev ContainersでJavaのコード解析とデバッグを行えます。PostgreSQL 18は独立したサービスとして起動できますが、アプリとは未接続です。SPAは次のPRで追加します。各PRがマージされるまで次の実装は進めません。

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

`compose.yaml`はプロジェクトのルートに置き、共通の起動構成として使用します。リポジトリ全体を`/workspace`へ共有するため、通常起動とDev Containersの両方で`.vscode`を含む同じファイル配置になります。`.devcontainer/devcontainer.json`はこのComposeと専用の`compose.devcontainer.yaml`を参照します。専用設定では、アプリを止めてもエディター接続を保てるようコンテナを待機させます。

## 前提

- M1 Mac、稼働中のColima（Docker runtime）、Docker CLI、Docker Compose。
- リポジトリのルートで以下のコマンドを実行します。
- `docker context show`が`colima`であり、`docker info`でサーバー情報が取得できることを確認します。Colimaが未起動なら`colima start`を実行します。
- 初回はイメージ・Maven・依存ライブラリを取得するため、インターネット接続が必要です。
- MacへJavaやMavenを追加インストールする必要はありません。
- 初回は`.env.example`を`.env`へコピーし、`POSTGRES_PASSWORD`をローカル開発専用の値へ変更します。既存の`.env`は上書きしません。`.env`はGit対象外です。

```sh
cp -n .env.example .env
```

`.env`はComposeの設定読み込みに必要です。DBサービスにのみ渡し、Spring BootにはDB設定を追加しません。`docker compose config`は値を表示するため、構文確認には`docker compose config --quiet`を使います。

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

コンテナと専用ネットワークを片付ける場合は次を実行します。MavenキャッシュとDBデータは残ります。

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

MacのVS Codeで閲覧できます。Javaのコード解析とデバッグは以下のDev Containers手順を使います。

## VS Codeでコンテナを開く

1. Mac側でColimaを起動し、VS Codeでリポジトリのルートを開きます。
2. コマンドパレット（Cmd+Shift+P）から`Dev Containers: Reopen in Container`を実行します。
3. 左下に`Dev Container: Procurement Java 21`が表示されるまで待ちます。初回はVS Code ServerとJava拡張の取得にネットワーク接続が必要です。
4. Javaファイルを開き、Javaプロジェクトの読み込みが終わるまで待ちます。コンテナ内の`/opt/java/openjdk`をJava 21として利用します。

追加する拡張は`redhat.java`（Mavenプロジェクトの認識・コード解析）と`vscjava.vscode-java-debug`（デバッグ）の2つです。Mavenのアプリ依存は増やしません。

Dev Containerではサーバーを自動起動しません。`Tasks: Run Task`から`backend: run`を選ぶと起動します。タスクのターミナルでCtrl+Cを押すとアプリだけ停止します。通常起動タスクとデバッグ起動タスクは同時実行しません。

### Tasksやデバッグ設定が表示されない場合

VS Codeで開くフォルダは`/workspace`です。`/workspace/backend`だけを開くと、親ディレクトリの`.vscode`設定は読み込まれません。コンテナ内のターミナルで次を確認します。

```sh
ls -l /workspace/.vscode/tasks.json /workspace/.vscode/launch.json
```

ファイルがなければ、古いマウント構成のコンテナが動いている可能性があります。`Dev Containers: Rebuild Container`を実行して設定を反映してください。ファイルがある場合は、`/workspace`を開いているか、VS Codeが制限モードになっていないかを確認し、必要に応じて`Developer: Reload Window`で再読込します。ワークスペースの信頼はユーザーが内容を確認して判断します。

「ターミナル → タスクの実行」から`backend: run`と`backend: debug (wait for attach)`を確認できます。デバッグ設定は「実行とデバッグ」の構成選択から`Attach to backend (Dev Container)`を選びます。

## 起動処理をデバッグする

以下はDev Container内で操作します。Mac側の通常ウィンドウでは実行しません。

1. 起動中の`backend: run`などがあれば、そのターミナルでCtrl+Cを押して停止します。
2. `ProcurementApplication.java`の`SpringApplication.run(...)`の行にブレークポイントを設定します。
3. `Tasks: Run Task`から`backend: debug (wait for attach)`を選択します。
4. ターミナルに`Listening for transport dt_socket at address: 5005`が出たら、「実行とデバッグ」で`Attach to backend (Dev Container)`を選び、F5で接続します。
5. ブレークポイントで停止したことを確認し、F5で続行します。起動完了後にMacのブラウザから`http://localhost:18081/actuator/health`を確認します。

`suspend=y`によりデバッガーが接続するまでJavaの実行を待つため、接続前や起動途中のブレークポイントではHTTP応答がありません。5005はコンテナ内のループバックでのみ待ち受け、Macへ公開・自動転送しません。

終了時はデバッガーを切断し、デバッグ起動タスクのターミナルでCtrl+Cを押してアプリも停止します。切断だけではJVMが動き続ける場合があります。接続失敗時は待機メッセージ、Java拡張の読み込み完了、通常起動との二重実行を確認してください。

## VS Codeを閉じる・通常起動へ戻す

`shutdownAction: none`のため、ウィンドウを閉じてもコンテナは停止しません。ただし、VS Codeタスクで起動したアプリの継続稼働は保証しません。継続稼働が必要なら通常のCompose起動を使います。

`Dev Containers: Reopen Folder Locally`でMac側へ戻り、Macのターミナルで次を実行すると、コンテナを通常の自動起動構成へ戻せます。

```sh
docker compose up -d --build --force-recreate backend
```

Dev Containerの待機構成で`docker compose restart backend`を実行してもアプリは起動しません。上記の再作成を使用してください。設定ファイルの変更後は`Dev Containers: Rebuild Container`で反映します。

Git操作は従来どおりMac側のCodex・ターミナルで行います。コンテナにはGitやDocker CLI、Dockerソケットを追加していません。

## PostgreSQL単体の起動・確認

Mac側のリポジトリルートで実行します。以下はbackendを再作成しないため、Dev Containerでの作業にも影響しません。

```sh
docker compose up -d --wait db
docker compose ps db
docker compose exec db sh -c 'pg_isready -h 127.0.0.1 -U "$POSTGRES_USER" -d "$POSTGRES_DB"'
docker compose exec db sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -v ON_ERROR_STOP=1 -c "SELECT 1;"'
```

状態が`healthy`となり、SQL結果が`1`なら成功です。対話的にSQLを実行する場合は以下を使い、`\q`で終了します。

```sh
docker compose exec db sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"'
```

PostgreSQL 18.6のARM64対応イメージをダイジェストで固定しています。DBはコンテナ内の5432で待ち受け、Macの`127.0.0.1:15433`へ公開します。Mac上のDBクライアントから利用でき、他の端末向けには公開しません。既存の`spring-db`等とは、ポート・コンテナ・ネットワーク・ボリュームを分離しています。

### MacのDBクライアントから接続する

| 項目 | 設定値 |
| --- | --- |
| 接続種類 | PostgreSQL |
| ホスト | `127.0.0.1` |
| ポート | `15433` |
| データベース | `.env`の`POSTGRES_DB`（見本では`procurement`） |
| ユーザー | `.env`の`POSTGRES_USER`（見本では`procurement`） |
| パスワード | `.env`の`POSTGRES_PASSWORD` |
| SSLモード | `disable`（ローカル開発用。TLS未構成） |

接続後、`SELECT 1;`でSQLが実行できることを確認します。パスワードは手順書やGitに転記しません。コンテナ同士で接続する際の接続先は`db:5432`で、Mac向けの`15433`とは異なります。

設定変更の反映は`docker compose up -d --wait db`で行います。ポートが競合した場合は既存サービスを停止せず、`lsof -nP -iTCP:15433 -sTCP:LISTEN`と`docker ps`で使用状況を確認してください。

DBデータは`procurement-asset-management_postgres-data`に保存します。PostgreSQL 18の公式イメージに合わせ、マウント先は`/var/lib/postgresql`です。ローカル開発用ユーザーは初期化管理者であり、本番の権限設計ではありません。

### 停止・再起動とデータ保持

```sh
docker compose stop db
docker compose up -d --wait db
```

通常の停止、コンテナ再作成、`docker compose down`ではデータを保持します。`down -v`やボリューム削除はデータを失うので通常操作には使用しません。

`POSTGRES_DB`・`POSTGRES_USER`・`POSTGRES_PASSWORD`は空のデータ領域の初期化に使用されます。初期化後に`.env`だけを書き換えても既存DBや認証情報は変更されません。値を変える必要がある場合は既存DB側の変更手順を検討し、ボリュームを安易に削除しないでください。

### サーバーとの分離

backendにDBへの`depends_on`や接続ライブラリは追加していません。`.env`を準備したうえで`docker compose up -d backend`によりDBなしでもサーバーを起動できます。Dev Containersで開いた場合は通常どおりVS Codeの起動タスクを使用します。DB停止中でもActuatorの`UP`はDBの正常性を保証しません。

DBが起動しない場合は`docker compose logs --tail=100 db`で確認します。初回のパスワード未設定や、すでに初期化済みのDBと`.env`の不一致を確認してください。
