# Task 5: フロントエンドのお気に入りサービス実装

## 概要

Auth0トークンを使用してお気に入り関連のAPI通信を行うコンポーザブル `useFavoriteService.ts` を実装する。

## 要件

- 要件1.4: お気に入り追加処理が完了するとユーザーに成功メッセージを表示する
- 要件2.4: お気に入り削除処理が完了するとユーザーに削除完了メッセージを表示する

## 実装内容

### 1. useFavoriteService.ts コンポーザブル作成

```typescript
export const useFavoriteService = () => {
  const { axios } = useAxiosService()

  const addFavorite = async (menuId: number, token: string): Promise<Favorite>
  const removeFavorite = async (favoriteId: number, token: string): Promise<void>
  const getFavorites = async (token: string): Promise<FavoriteWithMenu[]>
  const checkFavoriteStatus = async (menuId: number, token: string): Promise<{isFavorite: boolean, favoriteId?: number}>

  return {
    addFavorite,
    removeFavorite,
    getFavorites,
    checkFavoriteStatus
  }
}
```

### 2. Auth0トークンを使用したAPI通信機能

- Authorization Bearerヘッダーにトークンを設定
- 既存のuseAxiosServiceを活用
- snake_case ↔ camelCase変換の活用

### 3. エラーハンドリングとレスポンス処理

- 401 Unauthorized: 認証エラー
- 403 Forbidden: 権限エラー
- 404 Not Found: リソース未発見
- 409 Conflict: 重複エラー
- 500 Internal Server Error: サーバーエラー

## TypeScript インターフェース

```typescript
interface User {
  userId: number
  auth0Sub: string
  createdAt: string
  updatedAt: string
}

interface Favorite {
  favoriteId: number
  userId: number
  menuId: number
  createdAt: string
}

interface FavoriteWithMenu {
  favoriteId: number
  userId: number
  menuId: number
  menuName: string
  genreIds: number[]
  categoryIds: number[]
  createdAt: string
}
```

## 受け入れ基準

- [ ] composables/useFavoriteService.ts ファイルが作成されている
- [ ] addFavorite関数が実装されている（POST /v1/favorites）
- [ ] removeFavorite関数が実装されている（DELETE /v1/favorites/:id）
- [ ] getFavorites関数が実装されている（GET /v1/favorites）
- [ ] checkFavoriteStatus関数が実装されている（GET /v1/favorites/check/:menuId）
- [ ] Auth0トークンをAuthorizationヘッダーに設定している
- [ ] 適切なエラーハンドリングが実装されている
- [ ] TypeScriptの型定義が正しく設定されている
- [ ] 既存のuseAxiosServiceを活用している

## 技術スタック

- Nuxt 3
- TypeScript
- Axios
- Composables

## 関連ファイル

- composables/useAxiosService.ts (既存)
- バックエンドAPI (Task 2-4で実装)
