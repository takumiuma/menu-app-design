# Task 3: ユーザー管理APIエンドポイントの実装

## 概要

Auth0 subによるユーザー作成/取得を行うAPIエンドポイントを実装する。初回認証時は新規ユーザーを作成し、既存ユーザーの場合は既存レコードを返却する。

## 要件

- 要件4.1: Auth0から提供されるユーザー識別子（sub）を取得する
- 要件4.2: ユーザー識別子をデータベースのusersテーブルに保存する

## 実装内容

### 1. POST /v1/users エンドポイント

```go
// リクエスト
type CreateUserRequest struct {
    Auth0Sub string `json:"auth0Sub" binding:"required"`
}

// レスポンス
type UserResponse struct {
    User User `json:"user"`
}
```

### 2. ユーザー作成/取得ロジック

- Auth0 subでユーザーを検索
- 存在しない場合: 新規ユーザーを作成
- 存在する場合: 既存ユーザー情報を返却
- エラーハンドリング

### 3. データベース操作

- GORM を使用したユーザーの検索・作成
- トランザクション処理
- 重複チェック

## API仕様

```
POST /v1/users
Content-Type: application/json

Request Body:
{
  "auth0Sub": "auth0|1234567890abcdef"
}

Response (201 Created / 200 OK):
{
  "user": {
    "userId": 1,
    "auth0Sub": "auth0|1234567890abcdef",
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}

Error Response (400 Bad Request):
{
  "error": "Invalid auth0Sub format"
}
```

## 受け入れ基準

- [ ] POST /v1/users エンドポイントが実装されている
- [ ] Auth0 subでユーザーを検索できる
- [ ] 新規ユーザーの場合、データベースに作成される
- [ ] 既存ユーザーの場合、既存レコードが返却される
- [ ] 適切なHTTPステータスコードを返す
- [ ] バリデーションエラーを適切に処理する
- [ ] データベースエラーを適切に処理する

## 技術スタック

- Go
- Gin (HTTPフレームワーク)
- GORM
- PostgreSQL (想定)

## 関連ファイル

- ユーザーAPI実装ファイル
- ユーザーサービス層ファイル
