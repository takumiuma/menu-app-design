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
  const getFavorites = async (token: string): Promise<Favorite[]>

  return {
    addFavorite,
    removeFavorite,
    getFavorites
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

## 実装詳細

### addFavorite関数

```typescript
const addFavorite = async (menuId: number, token: string): Promise<{ favoriteId: number }> => {
  try {
    const response = await axios.post(
      '/v1/favorites',
      { menuId },
      {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      }
    )
    return { favoriteId: response.data.favoriteId }
  } catch (error) {
    console.error('Failed to add favorite:', error)
    throw error
  }
}
```

### removeFavorite関数

```typescript
const removeFavorite = async (favoriteId: number, token: string): Promise<void> => {
  try {
    await axios.delete(`/v1/favorites/${favoriteId}`, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    })
  } catch (error) {
    console.error('Failed to remove favorite:', error)
    throw error
  }
}
```

### getFavorites関数

```typescript
const getFavorites = async (token: string): Promise<Favorite[]> => {
  try {
    const response = await axios.get('/v1/favorites', {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    })
    return response.data.favorites
  } catch (error) {
    console.error('Failed to get favorites:', error)
    throw error
  }
}
```

## TypeScript インターフェース

```typescript

interface Favorite {
  favoriteId: number
  menuId: number
}

```

## API仕様

### POST /v1/favorites

```typescript
// Request
{
  menu_id: number
}

// Response
{
  favorite_id: number
  menu_id: number
}
```

### DELETE /v1/favorites/:favoriteId

```typescript
// Response
// 204 No Content または { success: true }
```

### GET /v1/favorites

```typescript
// Response
{
  favorites: Favorite[]
}
```

## 受け入れ基準

- [ ] composables/useFavoriteService.ts ファイルが作成されている
- [ ] addFavorite関数が実装されている（POST /v1/favorites）
- [ ] removeFavorite関数が実装されている（DELETE /v1/favorites/:id）
- [ ] getFavorites関数が実装されている（GET /v1/favorites）
- [ ] **checkFavoriteStatus関数は実装されていない**
- [ ] Auth0トークンをAuthorizationヘッダーに設定している
- [ ] **addFavoriteのレスポンスは{ favoriteId: number }形式**
- [ ] 適切なエラーハンドリングが実装されている
- [ ] TypeScriptの型定義が正しく設定されている
- [ ] 既存のuseAxiosServiceを活用している

## エラーハンドリング例

```typescript
const handleApiError = (error: any) => {
  if (error.response?.status === 401) {
    throw new Error('認証が必要です')
  } else if (error.response?.status === 403) {
    throw new Error('権限がありません')
  } else if (error.response?.status === 404) {
    throw new Error('リソースが見つかりません')
  } else if (error.response?.status === 409) {
    throw new Error('既にお気に入りに追加されています')
  } else {
    throw new Error('操作に失敗しました')
  }
}
```

## 技術スタック

- Nuxt 3
- TypeScript
- Axios
- Composables

## 関連ファイル

- composables/useAxiosService.ts (既存)
- バックエンドAPI (Task 2, 4.1-4.3で実装)

## 注意事項

- 状態チェックはPinia Storeで行う
- レスポンスは必要最小限の情報のみ含む
- エラーハンドリングは呼び出し元（Store）で行う
