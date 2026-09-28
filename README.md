# procurement-asset-management_codex
購入申請・承認・発注から、備品の貸出・返却・廃棄までを管理する業務アプリ。Codexを中心に開発。

## 開発環境

Java 21・Spring Boot 4.1.1のサーバー、PostgreSQL 18、React・TypeScript・ViteのSPA起動確認画面を用意しています（各サービスは未接続）。
VS CodeのDev ContainersでJavaのコード解析とデバッグも行えます。
MacへJava・Maven・Node.jsをインストールせず、Colima・Docker Composeで実行します。

- [SPA・サーバー・DBの起動・確認手順](docs/development-environment.md)
- [開発とGitの作業手順](docs/development-workflow.md)

環境構築はサーバー → Java用Dev Containers → PostgreSQL → SPAの順に、各PRをマージしてから次へ進めます。
