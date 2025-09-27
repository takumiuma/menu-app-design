# Task 4.2: お気に入り削除APIの実装

## 概要

認証されたユーザーがお気に入りからメニューアイテムを削除するAPIエンドポイントを実装する。削除権限チェック（自分のお気に入りのみ削除可能）を含む。

## 要件

- 要件2.2: ユーザーがお気に入り済みのボタンをクリックするとメニューアイテムをお気に入りリストから削除する
- 要件2.3: お気に入りから削除されるとボタンの表示状態が「未追加」状態に戻る
- 要件2.4: お気に入り削除処理が完了するとユーザーに削除完了メッセージを表示する

## 実装内容

### 1. DELETE /v1/favorites/:favoriteId エンドポイント

```go
// レスポンス
type DeleteFavoriteResponse struct {
    Success bool `json:"success"`
}
```

### 2. 削除権限チェック

- Auth0トークンからユーザーIDを取得
- 削除対象のお気に入りが認証ユーザーのものかチェック
- 他ユーザーのお気に入りは削除不可

### 3. エラーハンドリング

- 401 Unauthorized: 認証トークン無効
- 403 Forbidden: 他ユーザーのお気に入りを削除しようとした場合
- 404 Not Found: 存在しないお気に入りID
- 500 Internal Server Error: データベースエラー

## API仕様

```
DELETE /v1/favorites/:favoriteId
Authorization: Bearer <auth0_token>

Response (200 OK):
{
  "success": true
}

Error Response (403 Forbidden):
{
  "error": "You can only delete your own favorites"
}

Error Response (404 Not Found):
{
  "error": "Favorite not found"
}
```

## 受け入れ基準

- [ ] DELETE /v1/favorites/:favoriteId エンドポイントが実装されている
- [ ] Auth0トークンからユーザーIDを取得できる
- [ ] 削除対象のお気に入りが認証ユーザーのものかチェックする
- [ ] 他ユーザーのお気に入りを削除しようとした場合403エラーを返す
- [ ] 存在しないお気に入りIDの場合404エラーを返す
- [ ] 削除成功時にsuccess: trueを返す
- [ ] データベースエラーを適切に処理する

## 技術スタック

- Go
- Gin (HTTPフレームワーク)
- GORM
- Auth0 JWT検証

## 関連ファイル

- お気に入りAPI実装ファイル
- 認証ミドルウェア (Task 2で実装)
