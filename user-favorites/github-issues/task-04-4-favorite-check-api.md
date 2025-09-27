# Task 4.4: お気に入り状態確認APIの実装

## 概要

特定のメニューアイテムが認証されたユーザーのお気に入りに登録されているかを確認するAPIエンドポイントを実装する。

## 要件

- 要件1.1: 認証済みの状態でメニューアイテムを表示している際にお気に入りボタンを表示する
- 要件1.5: ユーザーが未認証の場合はお気に入りボタンは表示されない

## 実装内容

### 1. GET /v1/favorites/check/:menuId エンドポイント

```go
// レスポンス
type FavoriteCheckResponse struct {
    IsFavorite bool `json:"isFavorite"`
    FavoriteID *uint `json:"favoriteId,omitempty"`
}
```

### 2. お気に入り状態確認ロジック

- Auth0トークンからユーザーIDを取得
- 指定されたメニューIDがそのユーザーのお気に入りに存在するかチェック
- お気に入りの場合はfavoriteIdも返却（削除時に使用）

### 3. 認証チェック

- 認証ユーザーのお気に入り状態のみ確認
- 他ユーザーのお気に入り状態は取得不可

## API仕様

```
GET /v1/favorites/check/:menuId
Authorization: Bearer <auth0_token>

Response (200 OK - お気に入り済み):
{
  "isFavorite": true,
  "favoriteId": 123
}

Response (200 OK - お気に入り未登録):
{
  "isFavorite": false
}

Error Response (404 Not Found):
{
  "error": "Menu not found"
}
```

## 受け入れ基準

- [ ] GET /v1/favorites/check/:menuId エンドポイントが実装されている
- [ ] Auth0トークンからユーザーIDを取得できる
- [ ] 指定メニューが認証ユーザーのお気に入りかチェックする
- [ ] お気に入りの場合はisFavorite: true とfavoriteIdを返す
- [ ] お気に入りでない場合はisFavorite: false を返す
- [ ] 存在しないメニューIDの場合404エラーを返す
- [ ] 他ユーザーのお気に入り状態は取得できない
- [ ] データベースエラーを適切に処理する

## 技術スタック

- Go
- Gin (HTTPフレームワーク)
- GORM
- Auth0 JWT検証

## 関連ファイル

- お気に入りAPI実装ファイル
- 認証ミドルウェア (Task 2で実装)
