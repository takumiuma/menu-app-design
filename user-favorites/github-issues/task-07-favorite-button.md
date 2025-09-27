# Task 7: FavoriteButtonコンポーネントの実装

## 概要

再利用可能なお気に入りボタンコンポーネントを作成する。お気に入り状態の視覚的表示とクリック時の追加/削除処理を含む。

## 要件

- 要件1.1: 認証済みの状態でメニューアイテムを表示している際にお気に入りボタン（ハートアイコン等）を表示する
- 要件1.2: ユーザーがお気に入りボタンをクリックするとメニューアイテムをお気に入りリストに追加する
- 要件1.3: メニューアイテムがお気に入りに追加されるとボタンの表示状態が変更される（塗りつぶし等）
- 要件2.1: お気に入り済みのメニューアイテムを表示している際にお気に入りボタンは「追加済み」状態で表示される
- 要件2.2: ユーザーがお気に入り済みのボタンをクリックするとメニューアイテムをお気に入りリストから削除する
- 要件2.3: お気に入りから削除されるとボタンの表示状態が「未追加」状態に戻る

## 実装内容

### 1. FavoriteButton.vue コンポーネント作成

```vue
<template>
  <v-btn
    :icon="isFavorite ? 'mdi-heart' : 'mdi-heart-outline'"
    :color="isFavorite ? 'red' : 'grey'"
    @click="toggleFavorite"
    :loading="loading"
    :size="size"
  />
</template>

<script setup lang="ts">
interface Props {
  menuId: number
  size?: string
}

interface Emits {
  favoriteChanged: [isFavorite: boolean]
}
</script>
```

### 2. お気に入り状態の視覚的表示

- ハートアイコンの使用（mdi-heart / mdi-heart-outline）
- お気に入り済み: 赤色の塗りつぶしハート
- 未追加: グレーのアウトラインハート
- ローディング状態の表示

### 3. クリック時のお気に入り追加/削除処理

- Auth0認証状態の確認
- お気に入り状態の取得
- 追加/削除の切り替え処理
- エラーハンドリング

### 4. Props と Events

- Props: menuId (必須), size (オプション)
- Events: favoriteChanged (お気に入り状態変更時)

## 実装詳細

### toggleFavorite関数

```typescript
const toggleFavorite = async () => {
  if (!isAuthenticated.value) return

  loading.value = true
  try {
    if (isFavorite.value) {
      await favoriteStore.removeFromFavorites(favoriteId.value!, token.value)
    } else {
      await favoriteStore.addToFavorites(props.menuId, token.value)
    }
    emit('favoriteChanged', !isFavorite.value)
  } catch (error) {
    // エラーハンドリング
  } finally {
    loading.value = false
  }
}
```

## 受け入れ基準

- [ ] components/FavoriteButton.vue ファイルが作成されている
- [ ] Vuetifyのv-btnコンポーネントを使用している
- [ ] ハートアイコン（mdi-heart / mdi-heart-outline）を使用している
- [ ] お気に入り状態に応じて色とアイコンが変更される
- [ ] menuId propsを受け取る
- [ ] size propsを受け取る（オプション）
- [ ] favoriteChanged イベントを発火する
- [ ] Auth0認証状態を確認する
- [ ] useFavoriteStoreを使用してお気に入り操作を行う
- [ ] ローディング状態を表示する
- [ ] エラーハンドリングが実装されている
- [ ] TypeScriptの型定義が正しく設定されている

## 技術スタック

- Vue 3
- Vuetify 3
- TypeScript
- Composition API
- Material Design Icons

## 関連ファイル

- store/favorite.ts (Task 6で実装)
- Auth0認証機能 (既存)
