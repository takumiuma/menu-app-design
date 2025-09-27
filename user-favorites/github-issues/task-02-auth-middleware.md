# Task 2: Auth0トークン検証ミドルウェアの実装

## 概要

Auth0のJWTトークンを検証し、Auth0 subからユーザーIDを取得するミドルウェアを実装する。全てのお気に入り関連APIで認証チェックを行う。

## 要件

- 要件4.1: Auth0から提供されるユーザー識別子（sub）を取得する
- 要件4.4: 個人識別情報（PII）は最小限に留める
- 要件4.5: 適切なセキュリティ対策（暗号化、アクセス制御等）を実装する

## 実装内容

### 1. JWTトークン検証機能

- Authorization Bearerヘッダーからトークンを抽出
- Auth0の公開鍵でJWTトークンを検証
- トークンの有効期限チェック

### 2. Auth0 subからユーザーID取得機能

- JWTトークンからAuth0 subを抽出
- データベースでAuth0 subに対応するユーザーIDを検索
- ユーザーが存在しない場合の処理

### 3. 認証エラー時のレスポンス処理

- 401 Unauthorized: トークンが無効または期限切れ
- 403 Forbidden: ユーザーが見つからない
- 適切なエラーメッセージの返却

## API仕様

```go
// ミドルウェア関数
func AuthMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // トークン検証ロジック
        // ユーザーID取得ロジック
        // コンテキストにユーザーIDを設定
    }
}

// 使用例
router.Use(AuthMiddleware())
router.GET("/v1/favorites", getFavorites)
```

## 受け入れ基準

- [ ] Authorization Bearerヘッダーからトークンを正しく抽出できる
- [ ] Auth0の公開鍵でJWTトークンを検証できる
- [ ] JWTトークンからAuth0 subを抽出できる
- [ ] Auth0 subからデータベースのユーザーIDを取得できる
- [ ] 認証エラー時に適切なHTTPステータスコードを返す
- [ ] ミドルウェアがGinフレームワークで動作する

## 技術スタック

- Go
- Gin (HTTPフレームワーク)
- JWT-Go (JWTライブラリ)
- Auth0

## 関連ファイル

- 認証ミドルウェアファイル
- Auth0設定ファイル
