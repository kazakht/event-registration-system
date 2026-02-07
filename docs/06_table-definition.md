# テーブル定義書

本ドキュメントは、SQL Server上のテーブル定義を記載します。Entity Framework Core での使用を前提とした命名規則を採用しています。

---

## Events（イベント）

| 論理名       | 物理名      | 型               | PK/FK | Not Null | デフォルト値             | 備考                                                      |
| ------------ | ----------- | ---------------- | ----- | -------- | ------------------------ | --------------------------------------------------------- |
| イベントID   | Id          | uniqueidentifier | PK    | ○        | -                        | 主キー、EF Core で自動生成（DB側のデフォルト値なし）      |
| イベント名   | Name        | nvarchar(200)    | -     | ○        | -                        | イベントのタイトル、検索対象                              |
| 開催日時     | StartsAt    | datetime2(7)     | -     | ○        | -                        | イベント開催日時                                          |
| 開催場所     | Location    | nvarchar(500)    | -     | ○        | -                        | 開催場所の住所や施設名                                    |
| イベント概要 | Description | nvarchar(max)    | -     | ○        | -                        | イベントの詳細説明                                        |
| 作成日時     | CreatedAt   | datetime2(7)     | -     | ○        | SYSDATETIME()            | レコード作成日時（JST）                                   |
| 更新日時     | UpdatedAt   | datetime2(7)     | -     | ○        | SYSDATETIME() (INSERT時) | レコード更新日時（JST）、UPDATE時はアプリ側で明示的に更新 |

**インデックス:**

- IX_Events_Name (Name)
- IX_Events_StartsAt (StartsAt)
- IX_Events_Location (Location)

---

## Tickets（チケット）

| 論理名         | 物理名            | 型               | PK/FK | Not Null | デフォルト値             | 備考                                                      |
| -------------- | ----------------- | ---------------- | ----- | -------- | ------------------------ | --------------------------------------------------------- |
| チケットID     | Id                | uniqueidentifier | PK    | ○        | -                        | 主キー、EF Core で自動生成（DB側のデフォルト値なし）      |
| イベントID     | EventId           | uniqueidentifier | FK    | ○        | -                        | Events.Id への外部キー                                    |
| チケット種別名 | TicketType        | nvarchar(100)    | -     | ○        | -                        | 一般、学生、VIPなど                                       |
| 価格           | Price             | int              | -     | ○        | -                        | チケット価格（円）                                        |
| 総数           | TotalQuantity     | int              | -     | ○        | -                        | チケットの総発行枚数                                      |
| 残数           | AvailableQuantity | int              | -     | ○        | -                        | 購入可能な残りチケット数                                  |
| 作成日時       | CreatedAt         | datetime2(7)     | -     | ○        | SYSDATETIME()            | レコード作成日時（JST）                                   |
| 更新日時       | UpdatedAt         | datetime2(7)     | -     | ○        | SYSDATETIME() (INSERT時) | レコード更新日時（JST）、UPDATE時はアプリ側で明示的に更新 |

**外部キー制約:**

- FK_Tickets_Events: EventId → Events.Id (ON DELETE CASCADE)

**インデックス:**

- IX_Tickets_EventId (EventId)

**チェック制約:**

- CK_Tickets_AvailableQuantity: AvailableQuantity >= 0
- CK_Tickets_TotalQuantity: TotalQuantity >= 0
- CK_Tickets_Price: Price >= 0
- CK_Tickets_Quantity_Consistency: AvailableQuantity <= TotalQuantity

---

## Users（ユーザー）

| 論理名         | 物理名    | 型               | PK/FK | Not Null | デフォルト値  | 備考                                                 |
| -------------- | --------- | ---------------- | ----- | -------- | ------------- | ---------------------------------------------------- |
| ユーザーID     | Id        | uniqueidentifier | PK    | ○        | -             | 主キー、EF Core で自動生成（DB側のデフォルト値なし） |
| メールアドレス | Email     | nvarchar(256)    | UK    | ○        | -             | 参加者のメールアドレス、ユニーク制約                 |
| 作成日時       | CreatedAt | datetime2(7)     | -     | ○        | SYSDATETIME() | レコード作成日時（JST）                              |

---

## Registrations（参加登録）

| 論理名     | 物理名       | 型               | PK/FK | Not Null | デフォルト値  | 備考                                                 |
| ---------- | ------------ | ---------------- | ----- | -------- | ------------- | ---------------------------------------------------- |
| 登録ID     | Id           | uniqueidentifier | PK    | ○        | -             | 主キー、EF Core で自動生成（DB側のデフォルト値なし） |
| ユーザーID | UserId       | uniqueidentifier | FK    | ○        | -             | Users.Id への外部キー                                |
| チケットID | TicketId     | uniqueidentifier | FK    | ○        | -             | Tickets.Id への外部キー                              |
| 登録日時   | RegisteredAt | datetime2(7)     | -     | ○        | SYSDATETIME() | 参加登録日時（JST）                                  |

**外部キー制約:**

- FK_Registrations_Users: UserId → Users.Id (ON DELETE CASCADE)
- FK_Registrations_Tickets: TicketId → Tickets.Id (ON DELETE CASCADE)

**インデックス:**

- IX_Registrations_UserId (UserId)
- IX_Registrations_TicketId (TicketId)

**注記:**
同一ユーザーの同一イベント重複登録防止は、アプリケーション層で実装してください。登録時にRegistrationsテーブルをチェックし、ユーザーが既にそのイベントのチケットに登録済みかを確認します。

---

## 備考

### 命名規則

- **物理名**: PascalCase（EF Core の規約に準拠）
- **主キー**: すべて `Id` で統一
- **日時型**: `datetime2(7)` を使用（精度が高く、EF Core のデフォルト）
- **文字列型**: Unicode対応のため `nvarchar` を使用

### タイムゾーン

- すべての日時フィールドは JST（日本標準時）固定
- DB/OS のタイムゾーンを JST に固定
- SQL Server の `SYSDATETIME()` 関数でサーバーのローカル時刻（JST）を使用
- アプリケーション側でもタイムゾーン変換は不要

### 外部キー制約の削除動作

- `ON DELETE CASCADE`: 親レコード削除時に子レコードも削除
- イベント削除時はチケット・登録も削除される設計

### EF Core での自動生成

#### 主キー（GUID）の生成方式

- **生成方法**: EF Core がアプリケーション側で GUID を生成
- **DB のデフォルト値**: なし（`NEWID()` や `NEWSEQUENTIALID()` は使用しない）
- **理由**:
  - SQL Server で `NEWID()` をクラスタ化インデックスの主キーのデフォルト値に設定すると、ランダムな GUID によりインデックスの断片化が発生しやすく、性能劣化の原因となる
  - `NEWSEQUENTIALID()` も選択肢だが、EF Core での GUID 生成がより一般的で、アプリケーション側で値を制御しやすい
  - EF Core は `DatabaseGeneratedOption.Identity` により、INSERT 前に自動的に GUID を生成する

#### タイムスタンプの自動生成

- `CreatedAt`: DBのデフォルト値（SYSDATETIME()）により INSERT 時に自動設定
- `UpdatedAt`:
  - INSERT 時は DB のデフォルト値（SYSDATETIME()）で設定
  - UPDATE 時はアプリケーション側で明示的に更新が必要
  - **注意**: DB のデフォルト値は INSERT 時のみ適用され、UPDATE 時には自動更新されない
