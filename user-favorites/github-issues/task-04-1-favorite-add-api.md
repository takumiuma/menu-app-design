# Task 4.1: お気に入り追加APIの実装

## 概要

認証されたユーザーがメニューアイテムをお気に入りに追加するAPIエンドポイントを実装する。認証チェックとユーザー分離、重複チェックを含む。

## 要件

- 要件1.2: ユーザーがお気に入りボタンをクリックするとメニューアイテムをお気に入りリストに追加する
- 要件1.3: メニューアイテムがお気に入りに追加されるとボタンの表示状態が変更される
- 要件1.4: お気に入り追加処理が完了するとユーザーに成功メッセージを表示する

## 実装内容

### 1. POST /v1/favorites エンドポイント

```go
// リクエスト
type AddFavoriteRequest struct {
    MenuID uint `json:"menuId" binding:"required"`
}

// レスポンス
type FavoriteResponse struct {
    Favorite Favorite `json:"favorite"`
}
```

### 2. 認証チェックとユーザー分離

- Auth0トークンからユーザーIDを取得
- そのユーザーのお気に入りとして追加
- 他ユーザーのデータへのアクセスを防止

### 3. 重複チェックとエラーハンドリング

- 同じユーザー・メニューの組み合わせの重複チェック
- 409 Conflict: 既にお気に入り済み
- 404 Not Found: 存在しないメニュー
- 500 Internal Server Error: データベースエラー

## API仕様

```
POST /v1/favorites
Authorization: Bearer <auth0_token>
Content-Type: application/json

Request Body:
{
  "menuId": 123
}

Response (201 Created):
{
  "favorite": {
    "favoriteId": 1,
    "userId": 1,
    "menuId": 123,
    "createdAt": "2024-01-01T00:00:00Z"
  }
}

Error Response (409 Conflict):
{
  "error": "Menu is already in favorites"
}

Error Response (404 Not Found):
{
  "error": "Menu not found"
}
```

## 受け入れ基準

- [ ] POST /v1/favorites エンドポイントが実装されている
- [ ] Auth0トークンからユーザーIDを取得できる
- [ ] 認証されたユーザーのお気に入りとして追加される
- [ ] 重複チェックが正しく動作する
- [ ] 存在しないメニューIDの場合404エラーを返す
- [ ] 適切なHTTPステータスコードを返す
- [ ] データベースエラーを適切に処理する

## 技術スタック

- Go
- Gin (HTTPフレームワーク)
- GORM
- Auth0 JWT検証

## 関連ファイル

- お気に入りAPI実装ファイル
- 認証ミドルウェア (Task 2で実装)
