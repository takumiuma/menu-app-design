# Task 6: Piniaストアでのお気に入り状態管理

## 概要

お気に入りリストの状態管理とお気に入り状態のキャッシュ機能を提供するPiniaストア `favorite.ts` を実装する。

## 要件

- 要件1.3: メニューアイテムがお気に入りに追加されるとボタンの表示状態が変更される
- 要件2.3: お気に入りから削除されるとボタンの表示状態が「未追加」状態に戻る
- 要件3.3: お気に入りリストにアイテムがある場合は各アイテムの詳細情報を表示する

## 実装内容

### 1. favorite.ts ストア作成

```typescript
export const useFavoriteStore = defineStore('favorite', () => {
  const favorites = ref<FavoriteWithMenu[]>([])
  const favoriteStatus = ref<Map<number, {isFavorite: boolean, favoriteId?: number}>>(new Map())

  const loadFavorites = async (token: string): Promise<void>
  const addToFavorites = async (menuId: number, token: string): Promise<void>
  const removeFromFavorites = async (favoriteId: number, token: string): Promise<void>
  const checkFavoriteStatus = async (menuId: number, token: string): Promise<boolean>

  return {
    favorites: readonly(favorites),
    favoriteStatus: readonly(favoriteStatus),
    loadFavorites,
    addToFavorites,
    removeFromFavorites,
    checkFavoriteStatus
  }
})
```

### 2. お気に入りリストの状態管理

- お気に入りメニューリストの保持
- リストの追加・削除・更新
- リアクティブな状態更新

### 3. お気に入り状態のキャッシュ機能

- メニューIDごとのお気に入り状態をMapで管理
- API呼び出し回数の削減
- 状態の一貫性保持

## 実装詳細

### loadFavorites関数

- useFavoriteServiceを使用してお気に入りリストを取得
- favoritesとfavoriteStatusを更新
- エラーハンドリング

### addToFavorites関数

- useFavoriteServiceを使用してお気に入りを追加
- ローカル状態を即座に更新
- エラー時のロールバック処理

### removeFromFavorites関数

- useFavoriteServiceを使用してお気に入りを削除
- ローカル状態を即座に更新
- エラー時のロールバック処理

### checkFavoriteStatus関数

- キャッシュから状態を確認
- キャッシュにない場合はAPIから取得
- 状態をキャッシュに保存

## 受け入れ基準

- [ ] store/favorite.ts ファイルが作成されている
- [ ] Composition APIスタイルでdefineStoreを使用している
- [ ] favorites配列が適切に管理されている
- [ ] favoriteStatusマップが適切に管理されている
- [ ] loadFavorites関数が実装されている
- [ ] addToFavorites関数が実装されている
- [ ] removeFromFavorites関数が実装されている
- [ ] checkFavoriteStatus関数が実装されている
- [ ] useFavoriteServiceを適切に使用している
- [ ] エラーハンドリングが実装されている
- [ ] TypeScriptの型定義が正しく設定されている

## 技術スタック

- Nuxt 3
- Pinia
- TypeScript
- Composition API

## 関連ファイル

- composables/useFavoriteService.ts (Task 5で実装)
- 既存のPiniaストア設定
