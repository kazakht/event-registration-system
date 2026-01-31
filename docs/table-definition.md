# テーブル定義書

本ドキュメントは、SQL Server上のテーブル定義を記載します。Entity Framework Core での使用を前提とした命名規則を採用しています。

---

## Events（イベント）

| 論理名 | 物理名 | 型 | PK/FK | Not Null | デフォルト値 | 備考 |
|--------|--------|-----|-------|----------|--------------|------|
| イベントID | Id | uniqueidentifier | PK | ○ | NEWID() | 主キー、EF Core で自動生成 |
| イベント名 | Name | nvarchar(200) | - | ○ | - | イベントのタイトル、検索対象 |
| 開催日時 | EventDate | datetime2(7) | - | ○ | - | イベント開催日時 |
| 開催場所 | Location | nvarchar(500) | - | ○ | - | 開催場所の住所や施設名 |
| イベント概要 | Description | nvarchar(max) | - | ○ | - | イベントの詳細説明 |
| 作成日時 | CreatedAt | datetime2(7) | - | ○ | GETDATE() | レコード作成日時（JST） |
| 更新日時 | UpdatedAt | datetime2(7) | - | ○ | GETDATE() | レコード更新日時（JST）、UPDATE時はアプリ側で明示的に更新 |

**インデックス:**
- IX_Events_Name (Name)
- IX_Events_EventDate (EventDate)

---

## Tickets（チケット）

| 論理名 | 物理名 | 型 | PK/FK | Not Null | デフォルト値 | 備考 |
|--------|--------|-----|-------|----------|--------------|------|
| チケットID | Id | uniqueidentifier | PK | ○ | NEWID() | 主キー、EF Core で自動生成 |
| イベントID | EventId | uniqueidentifier | FK | ○ | - | Events.Id への外部キー |
| チケット種別名 | TicketType | nvarchar(100) | - | ○ | - | 一般、学生、VIPなど |
| 価格 | Price | int | - | ○ | - | チケット価格（円） |
| 総数 | TotalQuantity | int | - | ○ | - | チケットの総発行枚数 |
| 残数 | AvailableQuantity | int | - | ○ | - | 購入可能な残りチケット数 |
| 作成日時 | CreatedAt | datetime2(7) | - | ○ | GETDATE() | レコード作成日時（JST） |
| 更新日時 | UpdatedAt | datetime2(7) | - | ○ | GETDATE() | レコード更新日時（JST）、UPDATE時はアプリ側で明示的に更新 |

**外部キー制約:**
- FK_Tickets_Events: EventId → Events.Id (ON DELETE CASCADE)

**インデックス:**
- IX_Tickets_EventId (EventId)

**チェック制約:**
- CK_Tickets_AvailableQuantity: AvailableQuantity >= 0
- CK_Tickets_TotalQuantity: TotalQuantity >= 0
- CK_Tickets_Price: Price >= 0

---

## Users（ユーザー）

| 論理名 | 物理名 | 型 | PK/FK | Not Null | デフォルト値 | 備考 |
|--------|--------|-----|-------|----------|--------------|------|
| ユーザーID | Id | uniqueidentifier | PK | ○ | NEWID() | 主キー、EF Core で自動生成 |
| メールアドレス | Email | nvarchar(256) | UK | ○ | - | 参加者のメールアドレス、ユニーク制約 |
| 作成日時 | CreatedAt | datetime2(7) | - | ○ | GETDATE() | レコード作成日時（JST） |

**ユニークキー制約:**
- UK_Users_Email (Email)

---

## Registrations（参加登録）

| 論理名 | 物理名 | 型 | PK/FK | Not Null | デフォルト値 | 備考 |
|--------|--------|-----|-------|----------|--------------|------|
| 登録ID | Id | uniqueidentifier | PK | ○ | NEWID() | 主キー、EF Core で自動生成 |
| ユーザーID | UserId | uniqueidentifier | FK | ○ | - | Users.Id への外部キー |
| チケットID | TicketId | uniqueidentifier | FK | ○ | - | Tickets.Id への外部キー |
| 登録日時 | RegisteredAt | datetime2(7) | - | ○ | GETDATE() | 参加登録日時（JST） |

**外部キー制約:**
- FK_Registrations_Users: UserId → Users.Id (ON DELETE CASCADE)
- FK_Registrations_Tickets: TicketId → Tickets.Id (ON DELETE CASCADE)

**インデックス:**
- IX_Registrations_UserId (UserId)
- IX_Registrations_TicketId (TicketId)

**複合ユニークキー制約（オプション）:**
- UK_Registrations_UserTicket (UserId, TicketId) ※同一ユーザーが同一チケットに重複登録を防ぐ場合

---

## 備考

### 命名規則
- **物理名**: PascalCase（EF Core の規約に準拠）
- **主キー**: すべて `Id` で統一
- **日時型**: `datetime2(7)` を使用（精度が高く、EF Core のデフォルト）
- **文字列型**: Unicode対応のため `nvarchar` を使用

### タイムゾーン
- すべての日時フィールドは JST（日本標準時）で保存
- SQL Server の `GETDATE()` 関数でサーバーのローカル時刻（JST）を使用
- アプリケーション側でもタイムゾーン変換は不要

### 外部キー制約の削除動作
- `ON DELETE CASCADE`: 親レコード削除時に子レコードも削除
- イベント削除時はチケット・登録も削除される設計

### EF Core での自動生成
- `Id`: `[DatabaseGenerated(DatabaseGeneratedOption.Identity)]` 属性で GUID 自動生成
- `CreatedAt`: DBのデフォルト値（GETDATE()）により INSERT 時に自動設定
- `UpdatedAt`: 
  - INSERT 時は DB のデフォルト値（GETDATE()）で設定
  - UPDATE 時はアプリケーション側で明示的に更新が必要（SaveChanges のオーバーライドまたはインターセプタで実装）
  - **注意**: DB のデフォルト値は INSERT 時のみ適用され、UPDATE 時には自動更新されない

### 日時フィールドの更新方法
レコード更新時の `UpdatedAt` 自動設定は、以下のいずれかの方法で実装します：
- **EF Core インターセプタ**: `SaveChangesInterceptor` を継承して Modified 状態のエンティティの UpdatedAt を設定
- **SaveChanges オーバーライド**: `DbContext.SaveChanges()` をオーバーライドして Modified エンティティを検出し UpdatedAt を設定
- **トリガー**: SQL Server のトリガーで UPDATE 時に UpdatedAt を自動更新（非推奨：EF Core の変更追跡と競合の可能性あり）
