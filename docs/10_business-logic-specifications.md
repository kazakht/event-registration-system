# 業務処理機能 詳細仕様書

本ドキュメントは、イベント参加登録システムの業務処理機能（サービス層）の詳細仕様を定義します。

---

## BL-001: イベント検索

### 概要

現在日時以降に開催予定のイベントを一覧取得する。キーワードが指定された場合は、イベント名・概要・開催場所に対する部分一致検索を行い、該当するイベントのみに絞り込む。

### 業務ロジック

1. Controller から以下のパラメータを受け取る。
   - `keyword`（string, 任意）: 検索キーワード
2. `Events` テーブルに対し、`StartsAt >= 現在日時` の条件で開催予定イベントを抽出する。
3. `keyword` が指定されている場合（null または空文字でない場合）、以下のカラムを検索対象としてキーワードによる部分一致検索を行い、該当するイベントのみに絞り込む。
   - `Events.Name`
   - `Events.Description`
   - `Events.Location`
   - **検索方式**:
     - `Events.Name` および `Events.Location`: `LIKE '%keyword%'` による部分一致検索を基本とする。`docs/06_table-definition.md` に定義された `IX_Events_Name` および `IX_Events_Location` インデックスにより、性能を確保する。
     - `Events.Description`: `nvarchar(max)` であり、単純な `LIKE '%keyword%'` はフルスキャンになりやすいため、Full-Text Index による全文検索（`CONTAINS`）や検索用に正規化した別カラムの追加を検討する。その場合も、ユーザーから見たマッチングの挙動が部分一致検索と同等になるようにトークナイズや検索条件を設計すること。
4. 抽出結果から以下のフィールドを射影して返却する。
   - `Events.Id`
   - `Events.Name`
   - `Events.StartsAt`
   - `Events.Location`

### 例外処理

- 該当するイベントが存在しない場合は、空配列を返却する（エラーとしない）。

---

## BL-002: イベント詳細取得

### 概要

指定されたイベントIDに基づき、イベントの詳細情報および関連するチケット情報を結合して取得する。

### 業務ロジック

1. Controller から以下のパラメータを受け取る。
   - `eventId`（GUID, 必須）: 対象イベントのID
2. `Events` テーブルから `Id = eventId` に一致するレコードを1件取得する。
3. 該当レコードが存在しない場合、処理を中断し `404 Not Found`（detail: "Event not found."）を返却する。
4. `Tickets` テーブルから `EventId = eventId` に一致するチケット一覧を取得する。取得フィールドは以下の通り。
   - `Tickets.Id`
   - `Tickets.TicketType`
   - `Tickets.Price`
   - `Tickets.TotalQuantity`
   - `Tickets.AvailableQuantity`
5. 手順2で取得したイベント情報と手順4で取得したチケット一覧を結合し、以下の構造で返却する。
   - `Events.Id`
   - `Events.Name`
   - `Events.StartsAt`
   - `Events.Location`
   - `Events.Description`
   - `Tickets`（配列）

### 例外処理

| 条件                              | HTTPステータス | detail           |
| --------------------------------- | -------------- | ---------------- |
| 指定された `eventId` が存在しない | 404 Not Found  | Event not found. |

---

## BL-003: チケット情報取得

### 概要

指定されたイベントIDに紐づくチケットの種別・価格・残数を一覧で取得する。

### 業務ロジック

1. Controller から以下のパラメータを受け取る。
   - `eventId`（GUID, 必須）: 対象イベントのID
2. `Events` テーブルから `Id = eventId` に一致するレコードの存在を確認する。
3. 該当イベントが存在しない場合、処理を中断し `404 Not Found`（detail: "Event not found."）を返却する。
4. `Tickets` テーブルから `EventId = eventId` に一致するチケット一覧を取得する。取得フィールドは以下の通り。
   - `Tickets.Id`
   - `Tickets.TicketType`
   - `Tickets.Price`
   - `Tickets.TotalQuantity`
   - `Tickets.AvailableQuantity`
5. 取得したチケット一覧を返却する。

### 例外処理

| 条件                              | HTTPステータス | detail           |
| --------------------------------- | -------------- | ---------------- |
| 指定された `eventId` が存在しない | 404 Not Found  | Event not found. |

---

## BL-004: 参加登録

### 概要

メールアドレスとチケットIDを受け取り、イベントへの参加登録を行う。チケット在庫の確認、ユーザーの取得または新規作成、同一イベントへの重複登録チェック、在庫のデクリメントをトランザクション内で一貫して実行する。

### 業務ロジック

1. Controller から以下のパラメータを受け取る。
   - `email`（string, 必須）: 参加者のメールアドレス
     - RFC 5322 準拠の正規表現でメールアドレス形式を検証する。不正な場合は `400 Bad Request` を返却する。
   - `ticketId`（GUID, 必須）: 対象チケットのID
2. **--- トランザクション開始 ---**
3. `Tickets` テーブルから `Id = ticketId` に一致するレコードを取得する。該当レコードが存在しない場合、`404 Not Found`（detail: "Ticket not found."）を返却し、トランザクションをロールバックする。
4. 取得したチケットの `AvailableQuantity` を確認する。`AvailableQuantity <= 0` の場合、`409 Conflict`（detail: "Ticket is sold out."）を返却し、トランザクションをロールバックする。
5. `Users` テーブルを `Email = email` で検索する。
   - 該当ユーザーが存在する場合、そのレコードの `Id` を `userId` として使用する。
   - 該当ユーザーが存在しない場合、`Users` テーブルに新規レコードを INSERT する。
     - `Email` カラムには UNIQUE 制約が設定されているため、同時リクエストによる重複挿入が発生した場合は UNIQUE 違反例外となる。
     - UNIQUE 違反例外が発生した場合は、`Users` テーブルを再検索し、取得できた `Id` を `userId` として使用する（UPSERT 相当の手順）。
     - INSERT が成功した場合は、追加したレコードの `Id` を `userId` として使用する。
6. `Registrations` テーブルを `UserId = userId` で検索し、取得したレコードの `TicketId` を元に `Tickets` テーブルを参照して、同一イベント（`Tickets.EventId`）への登録が既に存在するか確認する。既に登録が存在する場合、`409 Conflict`（detail: "User is already registered for this event."）を返却し、トランザクションをロールバックする。
7. `Registrations` テーブルに以下のレコードを INSERT する。
   - `Id`: EF Core による GUID 自動生成
   - `UserId`: 手順5で取得した `userId`
   - `TicketId`: パラメータの `ticketId`
   - `RegisteredAt`: DB デフォルト値（SYSDATETIME()）
8. `Tickets` テーブルの該当レコード（`Id = ticketId`）に対し、`AvailableQuantity` を 1 デクリメントする（`AvailableQuantity = AvailableQuantity - 1`）。このとき、同時申し込みによる定員超過を防止するため、以下のいずれかの排他制御を適用する。
   - **楽観的同時実行制御（推奨）**: EF Core の `RowVersion` 列を利用し、更新時に行バージョンの一致を検証する。バージョン不一致（`DbUpdateConcurrencyException`）が発生した場合はリトライまたは `409 Conflict` を返却する。現時点のテーブル定義には `RowVersion` 列が未定義のため、将来のスキーマ変更で対応する。
   - **悲観的ロック（代替）**: UPDATE 文に行ロック（`WITH (UPDLOCK, ROWLOCK)`）を取得することで、他トランザクションによる同一行の同時更新をブロックする。
   - **WHERE 条件による防止（暫定対応）**: `AvailableQuantity > 0` を UPDATE の WHERE 条件に含め、更新件数が 0 の場合は `409 Conflict`（detail: "Ticket is sold out."）を返却する。`Tickets` テーブルにはチェック制約 `CK_Tickets_AvailableQuantity: AvailableQuantity >= 0` が設定されているため、負の値への更新はDB制約でも防止される。
9. **--- トランザクション終了（コミット）---**
10. 登録結果として以下の情報を返却する。
    - `Registrations.Id`（登録ID）
    - `Users.Id`（ユーザーID）
    - `Users.Email`（メールアドレス）
    - `Tickets.Id`（チケットID）
    - `Tickets.TicketType`（チケット種別名）
    - `Events.Name`（イベント名） ※ `Tickets.EventId` 経由で `Events` テーブルから取得
    - `Registrations.RegisteredAt`（登録日時）

### 例外処理

| 条件                                               | HTTPステータス            | detail                                                            |
| -------------------------------------------------- | ------------------------- | ----------------------------------------------------------------- |
| `email` の形式が不正                               | 400 Bad Request           | The email field is not a valid e-mail address.                    |
| 指定された `ticketId` が存在しない                 | 404 Not Found             | Ticket not found.                                                 |
| チケットの在庫がない（`AvailableQuantity <= 0`）   | 409 Conflict              | Ticket is sold out.                                               |
| 同一ユーザーが同一イベントに既に登録済み           | 409 Conflict              | User is already registered for this event.                        |
| 排他制御による更新失敗（同時実行競合）             | 409 Conflict              | Ticket is sold out.                                               |
| トランザクション内で予期しないエラーが発生した場合 | 500 Internal Server Error | An unexpected error occurred and the transaction was rolled back. |
