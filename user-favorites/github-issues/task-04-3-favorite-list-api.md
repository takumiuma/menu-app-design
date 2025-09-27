# Task 4.3: お気に入り一覧取得APIの実装

## 概要

認証されたユーザーのお気に入りメニューリストを取得するAPIエンドポイントを実装する。メニュー情報との結合処理を含む。

## 要件

- 要件3.2: お気に入りリストが空の場合は適切なメッセージを表示する
- 要件3.3: お気に入りリストにアイテムがある場合は各アイテムの詳細情報を表示する

## 実装内容

### 1. GET /v1/favorites エンドポイント

```go
// レスポンス
type FavoriteWithMenu struct {
    FavoriteID uint      `json:"favoriteId"`
    UserID     uint      `json:"userId"`
    MenuID     uint      `json:"menuId"`
    MenuName   string    `json:"menuName"`
    GenreIDs   []uint    `json:"genreIds"`
    CategoryIDs []uint   `json:"categoryIds"`
    CreatedAt  time.Time `json:"createdAt"`
}

type FavoritesResponse struct {
    Favorites []FavoriteWithMenu `json:"favorites"`
}
```

### 2. ユーザー固有のお気に入りリスト取得

- Auth0トークンからユーザーIDを取得
- そのユーザーのお気に入りのみ取得
- 他ユーザーのお気に入りは完全に隔離

### 3. メニュー情報との結合処理

- favoritesテーブルとmenusテーブルをJOIN
- メニュー名、ジャンルID、カテゴリIDを含む
- 削除されたメニューの処理

## API仕様

```
GET /v1/favorites
Authorization: Bearer <auth0_token>

Response (200 OK):
{
  "favorites": [
    {
      "favoriteId": 1,
      "userId": 1,
      "menuId": 123,
      "menuName": "ハンバーガー",
      "genreIds": [1, 2],
      "categoryIds": [3, 4],
      "createdAt": "2024-01-01T00:00:00Z"
    },
    {
      "favoriteId": 2,
      "userId": 1,
      "menuId": 456,
      "menuName": "パスタ",
      "genreIds": [2],
      "categoryIds": [5],
      "createdAt": "2024-01-01T01:00:00Z"
    }
  ]
}

Response (200 OK - 空のリスト):
{
  "favorites": []
}
```

## 受け入れ基準

- [ ] GET /v1/favorites エンドポイントが実装されている
- [ ] Auth0トークンからユーザーIDを取得できる
- [ ] 認証ユーザーのお気に入りのみ取得する
- [ ] メニュー情報（名前、ジャンル、カテゴリ）が含まれる
- [ ] お気に入りが空の場合は空配列を返す
- [ ] 削除されたメニューを適切に処理する
- [ ] 作成日時順でソートされる
- [ ] データベースエラーを適切に処理する

## 技術スタック

- Go
- Gin (HTTPフレームワーク)
- GORM (JOIN処理)
- Auth0 JWT検証

## 関連ファイル

- お気に入りAPI実装ファイル
- 認証ミドルウェア (Task 2で実装)
