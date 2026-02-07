# バックエンド処理一覧

## 概要

一般ユーザー向けの公開機能に対応するバックエンド処理を定義します。管理者用の作成・編集・削除機能は含みません。

## 処理一覧

| id     | 分類         | 処理名           | 概要                                                             | 操作手順                                                                      | ファイル名                   | 関連する画面名                          | HTTPメソッド | ルートパス                    | OpenAPI仕様                                                                               |
| ------ | ------------ | ---------------- | ---------------------------------------------------------------- | ----------------------------------------------------------------------------- | ---------------------------- | --------------------------------------- | ------------ | ----------------------------- | ----------------------------------------------------------------------------------------- |
| BP-001 | イベント管理 | イベント検索     | 開催予定イベントの一覧取得／キーワード検索対応                   | クエリ受け取り、フィルタ・検索、一覧返却                                      | `EventsController.cs`        | SC-001: イベント検索・一覧              | GET          | /api/events                   | `GET /api/events?keyword={keyword}` → 200: EventListResponse[]                            |
| BP-002 | イベント管理 | イベント詳細取得 | イベント詳細および関連チケット情報を取得                         | eventId受け取り、取得、チケット結合、返却                                     | `EventsController.cs`        | SC-002: イベント詳細                    | GET          | /api/events/{eventId}         | `GET /api/events/{eventId}` → 200: EventDetailResponse \| 404: NotFound                   |
| BP-003 | チケット管理 | チケット情報取得 | 指定イベントのチケット種別・価格・残数を取得                     | eventId受け取り、チケット一覧取得、返却                                       | `TicketsController.cs`       | SC-002: イベント詳細                    | GET          | /api/events/{eventId}/tickets | `GET /api/events/{eventId}/tickets` → 200: TicketResponse[] \| 404: NotFound              |
| BP-004 | 参加登録     | 参加登録         | メールアドレスとチケットIDで登録を行う（在庫・重複チェック含む） | リクエスト受け取り、バリデーション、在庫/重複確認、DB更新（トランザクション） | `RegistrationsController.cs` | SC-003: 参加登録 / SC-004: 参加登録完了 | POST         | /api/registrations            | `POST /api/registrations` → 201: RegistrationResponse \| 400: BadRequest \| 409: Conflict |

## 操作手順（詳細）

### BP-001: イベント検索

1. クエリパラメータ（`keyword`）を受け取る（任意）。
2. 現在日時以降のイベント（`StartsAt >= now`）をベースに抽出する。
3. `keyword` が指定されている場合は `Events.Name` 、 `Events.Description`、`Events.Location` に対して部分一致検索（LIKE）を実行する。
4. 必要なフィールド（`Id`, `Name`, `StartsAt`, `Location`）を整形して返却する。

### BP-002: イベント詳細取得

1. パスパラメータ `eventId` を受け取る。
2. `Events` テーブルから該当イベントを検索。
3. 見つからない場合は `404 Not Found` を返す。
4. 見つかった場合、`Tickets` テーブルから当該 `EventId` のチケット一覧を取得。
5. イベント情報とチケット一覧を結合した `EventDetailResponse` を返却する。

### BP-003: チケット情報取得

1. パスパラメータ `eventId` を受け取る。
2. `Tickets` テーブルを `EventId` で検索し、`TicketType`, `Price`, `TotalQuantity`, `AvailableQuantity` を取得。
3. 該当イベントが存在しない、またはチケットが見つからない場合は `404 Not Found` を返却する設計とする。

### BP-004: 参加登録

1. リクエストボディから `email` と `ticketId` を受け取る。
2. `email` の形式を正規表現でバリデーション（失敗: 400）。
3. `ticketId` の存在を `Tickets` テーブルで確認（存在しない: 400/404）。
4. `AvailableQuantity` を確認し、在庫がない場合は `409 Conflict` を返却。
5. `Users` テーブルを `email` で検索し、なければ新規作成（重複対策は UNIQUE 制約で担保）。
6. `Registrations` テーブルに同一 `UserId` と同じイベント（EventId）の登録がないか確認し、既存なら `409 Conflict` を返却。
7. トランザクションを開始し、`Registrations` に登録を追加、`Tickets.AvailableQuantity` をデクリメントする。
8. トランザクションをコミットし、成功で `201 Created` と登録情報を返却する。

## 対応する要件マッピング

| 処理ID | 機能要件ID                             | 画面ID         |
| ------ | -------------------------------------- | -------------- |
| BP-001 | FR-002, FR-003                         | SC-001         |
| BP-002 | FR-005, FR-006, FR-007, FR-008, FR-009 | SC-002         |
| BP-003 | FR-010                                 | SC-002         |
| BP-004 | FR-011, FR-012, FR-013, FR-014         | SC-003, SC-004 |
