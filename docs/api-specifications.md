# API 仕様書

本ドキュメントは、イベント参加登録システムの公開 API 仕様を定義します。

---

## 共通仕様

### ベース URL

```text
https://api.example.com
```

### Content-Type

- リクエスト: `application/json`
- レスポンス: `application/json; charset=utf-8`

### 日時フォーマット

- ISO 8601 形式: `YYYY-MM-DDTHH:mm:ss` (例: `2024-06-15T14:30:00`)
- タイムゾーン: JST（日本標準時）固定

### エラーレスポンス共通形式

ASP.NET Core の `ProblemDetails` 形式に準拠します。

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "email": ["The email field is required."]
  },
  "traceId": "00-trace-id-here"
}
```

---

## API 一覧

| エンドポイント                | メソッド | 概要             | 処理ID |
| ----------------------------- | -------- | ---------------- | ------ |
| /api/events                   | GET      | イベント検索     | BP-001 |
| /api/events/{eventId}         | GET      | イベント詳細取得 | BP-002 |
| /api/events/{eventId}/tickets | GET      | チケット情報取得 | BP-003 |
| /api/registrations            | POST     | 参加登録         | BP-004 |

---

## BP-001: イベント検索

### 基本情報

- **処理名**: イベント検索
- **HTTPメソッド**: GET
- **エンドポイント**: `/api/events`
- **概要**: 開催予定イベントの一覧取得およびキーワード検索

### リクエスト

#### Query Parameters

| パラメータ名 | データ型 | 必須 | 説明                                         |
| ------------ | -------- | ---- | -------------------------------------------- |
| keyword      | string   | 任意 | イベント名または概要の部分一致検索キーワード |

#### リクエスト例

```http
GET /api/events?keyword=音楽 HTTP/1.1
Host: api.example.com
```

### レスポンス (200 OK)

#### レスポンスボディ

```json
[
  {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "name": "春の音楽フェスティバル2024",
    "startsAt": "2024-06-15T14:00:00",
    "location": "東京ドーム"
  },
  {
    "id": "8b92c3e1-2a4f-4d8e-9f1a-7c5b6d3e8a2f",
    "name": "クラシック音楽コンサート",
    "startsAt": "2024-07-20T18:30:00",
    "location": "サントリーホール"
  }
]
```

#### フィールド定義

| フィールド名 | データ型 | 説明              |
| ------------ | -------- | ----------------- |
| id           | string   | イベントID (GUID) |
| name         | string   | イベント名        |
| startsAt     | string   | 開催日時          |
| location     | string   | 開催場所          |

### エラーレスポンス

#### 400 Bad Request

クエリパラメータの形式が不正な場合（通常は発生しない）。

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "Bad Request",
  "status": 400,
  "traceId": "00-trace-id-here"
}
```

---

## BP-002: イベント詳細取得

### 基本情報

- **処理名**: イベント詳細取得
- **HTTPメソッド**: GET
- **エンドポイント**: `/api/events/{eventId}`
- **概要**: イベント詳細情報および関連チケット情報を取得

### リクエスト

#### Path Parameters

| パラメータ名 | データ型 | 必須 | 説明              |
| ------------ | -------- | ---- | ----------------- |
| eventId      | string   | ○    | イベントID (GUID) |

#### リクエスト例

```http
GET /api/events/3fa85f64-5717-4562-b3fc-2c963f66afa6 HTTP/1.1
Host: api.example.com
```

### レスポンス (200 OK)

#### レスポンスボディ

```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "name": "春の音楽フェスティバル2024",
  "startsAt": "2024-06-15T14:00:00",
  "location": "東京ドーム",
  "description": "国内外の人気アーティストが集結する春の音楽イベント。ジャンルを超えた多彩なラインナップでお届けします。",
  "tickets": [
    {
      "id": "1a2b3c4d-5e6f-7a8b-9c0d-1e2f3a4b5c6d",
      "ticketType": "一般",
      "price": 8000,
      "totalQuantity": 5000,
      "availableQuantity": 1200
    },
    {
      "id": "2b3c4d5e-6f7a-8b9c-0d1e-2f3a4b5c6d7e",
      "ticketType": "学生",
      "price": 5000,
      "totalQuantity": 1000,
      "availableQuantity": 300
    }
  ]
}
```

#### フィールド定義

| フィールド名                | データ型 | 説明              |
| --------------------------- | -------- | ----------------- |
| id                          | string   | イベントID (GUID) |
| name                        | string   | イベント名        |
| startsAt                    | string   | 開催日時          |
| location                    | string   | 開催場所          |
| description                 | string   | イベント概要      |
| tickets                     | array    | チケット情報配列  |
| tickets[].id                | string   | チケットID (GUID) |
| tickets[].ticketType        | string   | チケット種別名    |
| tickets[].price             | integer  | 価格（円）        |
| tickets[].totalQuantity     | integer  | 総数              |
| tickets[].availableQuantity | integer  | 残数              |

### エラーレスポンス

#### 404 Not Found

指定されたイベントIDが存在しない場合。

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.4",
  "title": "Not Found",
  "status": 404,
  "detail": "Event not found.",
  "traceId": "00-trace-id-here"
}
```

---

## BP-003: チケット情報取得

### 基本情報

- **処理名**: チケット情報取得
- **HTTPメソッド**: GET
- **エンドポイント**: `/api/events/{eventId}/tickets`
- **概要**: 指定イベントのチケット種別・価格・残数を取得

### リクエスト

#### Path Parameters

| パラメータ名 | データ型 | 必須 | 説明              |
| ------------ | -------- | ---- | ----------------- |
| eventId      | string   | ○    | イベントID (GUID) |

#### リクエスト例

```http
GET /api/events/3fa85f64-5717-4562-b3fc-2c963f66afa6/tickets HTTP/1.1
Host: api.example.com
```

### レスポンス (200 OK)

#### レスポンスボディ

```json
[
  {
    "id": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
    "ticketType": "一般",
    "price": 8000,
    "totalQuantity": 5000,
    "availableQuantity": 1200
  },
  {
    "id": "2b3c4d5e-6f7g-8h9i-0j1k-2l3m4n5o6p7q",
    "ticketType": "学生",
    "price": 5000,
    "totalQuantity": 1000,
    "availableQuantity": 300
  },
  {
    "id": "3c4d5e6f-7g8h-9i0j-1k2l-3m4n5o6p7q8r",
    "ticketType": "VIP",
    "price": 15000,
    "totalQuantity": 200,
    "availableQuantity": 0
  }
]
```

#### フィールド定義

| フィールド名      | データ型 | 説明              |
| ----------------- | -------- | ----------------- |
| id                | string   | チケットID (GUID) |
| ticketType        | string   | チケット種別名    |
| price             | integer  | 価格（円）        |
| totalQuantity     | integer  | 総数              |
| availableQuantity | integer  | 残数              |

### エラーレスポンス

#### 404 Not Found

指定されたイベントIDが存在しない場合。

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.4",
  "title": "Not Found",
  "status": 404,
  "detail": "Event not found.",
  "traceId": "00-trace-id-here"
}
```

---

## BP-004: 参加登録

### 基本情報

- **処理名**: 参加登録
- **HTTPメソッド**: POST
- **エンドポイント**: `/api/registrations`
- **概要**: メールアドレスとチケットIDで参加登録を行う

### リクエスト

#### Request Body

| フィールド名 | データ型 | 必須 | 説明                   |
| ------------ | -------- | ---- | ---------------------- |
| email        | string   | ○    | 参加者のメールアドレス |
| ticketId     | string   | ○    | チケットID (GUID)      |

#### リクエスト例

```http
POST /api/registrations HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "email": "user@example.com",
  "ticketId": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p"
}
```

### レスポンス (201 Created)

#### レスポンスボディ

```json
{
  "id": "9f8e7d6c-5b4a-3210-fedc-ba9876543210",
  "userId": "4d3c2b1a-0987-6543-21fe-dcba09876543",
  "email": "user@example.com",
  "ticketId": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
  "ticketType": "一般",
  "eventName": "春の音楽フェスティバル2024",
  "registeredAt": "2024-05-01T10:30:00"
}
```

#### フィールド定義

| フィールド名 | データ型 | 説明              |
| ------------ | -------- | ----------------- |
| id           | string   | 登録ID (GUID)     |
| userId       | string   | ユーザーID (GUID) |
| email        | string   | メールアドレス    |
| ticketId     | string   | チケットID (GUID) |
| ticketType   | string   | チケット種別名    |
| eventName    | string   | イベント名        |
| registeredAt | string   | 登録日時          |

### エラーレスポンス

#### 400 Bad Request

リクエストボディのバリデーションエラー。

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "email": ["The email field is not a valid e-mail address."],
    "ticketId": ["The ticketId field is required."]
  },
  "traceId": "00-trace-id-here"
}
```

#### 404 Not Found

指定されたチケットIDが存在しない場合。

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.4",
  "title": "Not Found",
  "status": 404,
  "detail": "Ticket not found.",
  "traceId": "00-trace-id-here"
}
```

#### 409 Conflict

チケットの在庫不足、または同一ユーザーが同一イベントに重複登録しようとした場合。

**在庫不足の場合:**

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.8",
  "title": "Conflict",
  "status": 409,
  "detail": "Ticket is sold out.",
  "traceId": "00-trace-id-here"
}
```

**重複登録の場合:**

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.8",
  "title": "Conflict",
  "status": 409,
  "detail": "User is already registered for this event.",
  "traceId": "00-trace-id-here"
}
```

---

## 補足事項

### トランザクション処理

- **BP-004 (参加登録)** では以下の操作をトランザクション内で実行します:
  1. `Users` テーブルへのユーザー登録（存在しない場合）
  2. `Registrations` テーブルへの登録レコード追加
  3. `Tickets` テーブルの `availableQuantity` をデクリメント

### 同時実行制御

- チケット在庫の同時更新を防ぐため、楽観的同時実行制御（Optimistic Concurrency）または悲観的ロック（Pessimistic Lock）の実装を推奨します
- EF Core の `RowVersion` 列を使用した楽観的制御が一般的です

### レート制限

- API の乱用を防ぐため、IP アドレスごとのレート制限を実装することを推奨します
- 例: 1分間に60リクエストまで

### CORS 設定

- フロントエンドアプリケーションからのアクセスを許可するため、適切な CORS ポリシーを設定してください
