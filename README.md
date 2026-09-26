# procurement-asset-management_codex
購入申請・承認・発注から、備品の貸出・返却・廃棄までを管理する業務アプリ。Codexを中心に開発。

## 開発環境

現在はJava 21・Spring Boot 4.1.1のサーバー起動環境のみを用意しています。
MacへJavaやMavenをインストールせず、Colima・Docker Composeで実行します。

- [サーバーの起動・確認手順](docs/development-environment.md)
- [開発とGitの作業手順](docs/development-workflow.md)

環境構築はサーバー → Java用Dev Containers → PostgreSQL → SPAの順に、各PRをマージしてから次へ進めます。
