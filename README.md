# procurement-asset-management_codex
購入申請・承認・発注から、備品の貸出・返却・廃棄までを管理する業務アプリ。Codexを中心に開発。

## 要件定義（相談中）

- [会社・業務の前提](docs/requirements/business-context.md)
- [認証・認可と購入承認の要件](docs/requirements/authentication-authorization.md)
- [アカウント変更の管理・通知・確認](docs/requirements/account-administration.md)

合意事項・提案・未決定事項を区別して記録しています。要件定義は途中であり、記載した業務機能が実装済みであることを意味しません。

## 開発環境

Java 21・Spring Boot 4.1.1のサーバー、PostgreSQL 18、React・TypeScript・ViteのSPA起動確認画面を用意しています（各サービスは未接続）。
VS CodeのDev ContainersでJavaのコード解析とデバッグも行えます。
MacへJava・Maven・Node.jsをインストールせず、Colima・Docker Composeで実行します。

- [SPA・サーバー・DBの起動・確認手順](docs/development-environment.md)
- [開発とGitの作業手順](docs/development-workflow.md)

環境構築はサーバー → Java用Dev Containers → PostgreSQL → SPAの順に、各PRをマージしてから次へ進めます。
