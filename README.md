# procurement-asset-management_codex
購入申請・承認・発注から、備品の貸出・返却・廃棄までを管理する業務アプリ。Codexを中心に開発。

## 開発環境

Java 21・Spring Boot 4.1.1のサーバーと、独立したPostgreSQL 18の起動環境を用意しています（アプリとDBは未接続）。
VS CodeのDev ContainersでJavaのコード解析とデバッグも行えます。
MacへJavaやMavenをインストールせず、Colima・Docker Composeで実行します。

- [サーバー・DBの起動・確認手順](docs/development-environment.md)
- [開発とGitの作業手順](docs/development-workflow.md)

環境構築はサーバー → Java用Dev Containers → PostgreSQL → SPAの順に、各PRをマージしてから次へ進めます。
