# システム概要：イベント登録システム

## 1. 開発目的

- 学習者がAI駆動開発（Copilot）の実践的な活用方法を習得するためのデモシステム。
- イベント参加者がイベントを探し、参加登録できるシステムを提供する。

## 2. ターゲットユーザー

- イベント参加者：イベントを閲覧・参加申し込みする人

## 3. 主要機能（MVPスコープ）

- イベント参加者向けのイベント検索・一覧表示
  - キーワード検索機能によりイベントを検索できる

- イベント参加者向けのイベント詳細
  - 開催日時
  - 場所
  - 内容の概要
  - チケット情報（残数を含む）

- チケット関連
  - １つのイベントに対して複数種類のチケット設定が可能（一般、学生、VIPなど）
  - ユーザーは参加登録時にチケットを選択できる

- イベント参加者向けの参加登録
  - ログインなしでメールアドレスをにゅうりょくするのみで参加登録可能
  - ユーザーアカウント作成は不要
  - 選択したチケットで参加登録できる

## 4. 技術スタック

- Frontend:
  - Framework: Next.js (App Router)
  - Language: TypeScript
  - UI: Tailwind CSS, shadcn/ui

- Backend:
  - Language: C# (ASP.NET Core MVC)
  - Runtime: .NET 8 (ASP.NET Core)
  - Database: Microsoft SQL Server (Entity Framework Core)
  - API: MVC Controllers / REST API (Kestrel hosting) or Minimal APIs
