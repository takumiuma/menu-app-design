# Task 6: Piniaストアでのお気に入り状態管理

## 概要

お気に入りリストの状態管理を提供するPiniaストア `favorite.ts` を実装する。全件取得したお気に入りデータから効率的に状態を判定する仕組みを構築する。

## 変更点

- 全件取得したお気に入りデータから状態を計算
- Set を使用した高速な状態チェック

## 要件

- 要件1.3: メニューアイテムがお気に入りに追加されるとボタンの表示状態が変更される
- 要件2.3: お気に入りから削除されるとボタンの表示状態が「未追加」状態に戻る
- 要件3.3: お気に入りリストにアイテムがある場合は各アイテムの詳細情報を表示する

## 実装内容

### 1. favorite.ts ストア作成

```typescript
export const useFavoriteStore = defineStore('favorite', () => {
  const favorites = ref<FavoriteWithMenu[]>([])

  // 計算プロパティでお気に入りメニューIDのSetを作成（高速検索用）
  const favoriteMenuIds = computed(() =>
    new Set(favorites.value.map(f => f.menuId))
  )

  // お気に入り状態をチェック
  const isFavorite = (menuId: number): boolean => {
    return favoriteMenuIds.value.has(menuId)
  }

  // favoriteIdを取得（削除時に使用）
  const getFavoriteId = (menuId: number): number | undefined => {
    return favorites.value.find(f => f.menuId === menuId)?.favoriteId
  }

  // API呼び出し関数
  const loadFavorites = async (token: string): Promise<void>
  const addToFavorites = async (menuId: number, token: string): Promise<void>
  const removeFromFavorites = async (favoriteId: number, token: string): Promise<void>

  // ローカル状態更新用のヘルパー関数
  const addLocalFavorite = (favorite: FavoriteWithMenu): void => {
    favorites.value.push(favorite)
  }

  const removeLocalFavorite = (favoriteId: number): void => {
    const index = favorites.value.findIndex(f => f.favoriteId === favoriteId)
    if (index > -1) {
      favorites.value.splice(index, 1)
    }
  }

  return {
    favorites: readonly(favorites),
    favoriteMenuIds: readonly(favoriteMenuIds),
    isFavorite,
    getFavoriteId,
    loadFavorites,
    addToFavorites,
    removeFromFavorites,
    addLocalFavorite,
    removeLocalFavorite
  }
})
```

### 2. お気に入りリストの状態管理

- お気に入りメニューリストの保持
- 計算プロパティによる高速な状態チェック
- リアクティブな状態更新

### 3. 効率的な状態チェック機能

- `Set` を使用したO(1)の高速検索
- メニューIDからfavoriteIDの取得
- 状態変更時の即座な反映

## 実装詳細

### loadFavorites関数

```typescript
const loadFavorites = async (token: string): Promise<void> => {
  try {
    const { getFavorites } = useFavoriteService()
    const data = await getFavorites(token)
    favorites.value = data
  } catch (error) {
    console.error('Failed to load favorites:', error)
    throw error
  }
}
```

### addToFavorites関数

```typescript
const addToFavorites = async (menuId: number, token: string): Promise<void> => {
  try {
    const { addFavorite } = useFavoriteService()
    const response = await addFavorite(menuId, token)

    // ローカル状態を即座に更新（楽観的更新）
    // 注意: メニュー情報は別途取得が必要
    const menuInfo = await getMenuInfo(menuId) // 実装が必要
    addLocalFavorite({
      favoriteId: response.favoriteId,
      userId: 0, // 不要だが型のため
      menuId: menuId,
      menuName: menuInfo.menuName,
      genreIds: menuInfo.genreIds,
      categoryIds: menuInfo.categoryIds,
      createdAt: new Date().toISOString(),
    })
  } catch (error) {
    console.error('Failed to add favorite:', error)
    throw error
  }
}
```

### removeFromFavorites関数

```typescript
const removeFromFavorites = async (favoriteId: number, token: string): Promise<void> => {
  try {
    const { removeFavorite } = useFavoriteService()
    await removeFavorite(favoriteId, token)

    // ローカル状態を即座に更新
    removeLocalFavorite(favoriteId)
  } catch (error) {
    console.error('Failed to remove favorite:', error)
    // エラー時はローカル状態をロールバック
    await loadFavorites(token)
    throw error
  }
}
```

### FavoriteButtonでの使用例

```typescript
// components/FavoriteButton.vue
const favoriteStore = useFavoriteStore()

const isFavorite = computed(() => favoriteStore.isFavorite(props.menuId))
const favoriteId = computed(() => favoriteStore.getFavoriteId(props.menuId))

const toggleFavorite = async () => {
  if (isFavorite.value) {
    await favoriteStore.removeFromFavorites(favoriteId.value!, token.value)
  } else {
    await favoriteStore.addToFavorites(props.menuId, token.value)
  }
}
```

## 受け入れ基準

- [ ] store/favorite.ts ファイルが作成されている
- [ ] Composition APIスタイルでdefineStoreを使用している
- [ ] favorites配列が適切に管理されている
- [ ] favoriteMenuIds計算プロパティが実装されている
- [ ] isFavorite関数が高速に動作する（Set使用）
- [ ] getFavoriteId関数が実装されている
- [ ] loadFavorites関数が実装されている
- [ ] addToFavorites関数が実装されている
- [ ] removeFromFavorites関数が実装されている
- [ ] addLocalFavorite/removeLocalFavoriteヘルパー関数が実装されている
- [ ] useFavoriteServiceを適切に使用している
- [ ] エラーハンドリングが実装されている
- [ ] TypeScriptの型定義が正しく設定されている

## パフォーマンス考慮

- **Set使用**: O(1)の高速な状態チェック
- **計算プロパティ**: 依存関係の変更時のみ再計算
- **楽観的更新**: API成功を前提としたローカル状態の即座更新

## 技術スタック

- Nuxt 3
- Pinia
- TypeScript
- Composition API

## 関連ファイル

- composables/useFavoriteService.ts (Task 5で実装)
- GET /v1/favorites API (Task 4.3で実装)

## 注意事項

- メニュー情報の取得方法を別途検討が必要（addToFavorites時）
- 楽観的更新でUXを向上させるが、エラー時のロールバック処理も重要
