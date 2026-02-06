# プロジェクト概要
このプロジェクトは、一般ユーザーがイベントを閲覧し、参加申し込みを行うためのシステムです。

# 技術スタック
- **Frontend**: Next.js (App Router), TypeScript, Tailwind CSS, v0.devで生成したUIコンポーネント
- **Backend**: ASP.NET Core MVC (Web APIとして使用), .NET 8
- **Database**: SQL Server
- **ORM**: Entity Framework Core (EF Core)

# コーディング規約と設計指針
## 全般
- 回答およびコードコメントは常に **日本語** で行ってください。
- 実装前に、まずどのような方針でコードを書くかステップバイステップで説明してください。
- 破壊的な変更を含む場合は、必ず警告してください。
- 常に `docs/` フォルダ内の設計ドキュメント（要件定義、ER図、テーブル定義、API仕様書）を参照し、不整合がないようにしてください。

## バックエンド (ASP.NET Core)
- RESTful APIの原則に従ってください。
- コントローラーの命名は `◯◯Controller.cs` とし、依存性の注入（DI）を活用してください。
- 非同期処理（async/await）を標準的に使用してください。
- エラーハンドリングは、`api-specifications.md` で定義したステータスコード（404, 409等）を厳守してください。
- 定員チェックや申込登録など、データ整合性が重要な処理には EF Core のトランザクションを使用してください。

## フロントエンド (Next.js)
- コンポーネントは React Server Components (RSC) と Client Components を適切に使い分けてください。
- API通信は、バックエンドのルートパス（`/api/`）と JSON 構造に従ってください。
- ページネーション（`page`, `limit`）およびメタデータの処理を適切に実装してください。

## データベース
- 命名規則は `table-definition.md` に従ってください（PascalCase を推奨）。
- 日時データには `DateTime2` を使用してください。

# 禁止事項
- 管理者向け機能の実装は、明示的に指示がない限り含めないでください。
- インラインでの大規模なスタイル定義は避け、Tailwind CSS を使用してください。
