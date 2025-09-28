# Task 7: FavoriteButtonコンポーネントの実装（更新版）

## 概要

再利用可能なお気に入りボタンコンポーネントを作成する。Pinia Storeから状態を取得し、お気に入りの追加/削除処理を行う。

## 変更点

- 個別のAPI呼び出しではなく、Storeの状態から判定
- `isFavorite` と `favoriteId` をStoreから取得
- より効率的な状態管理

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

const props = defineProps<Props>()
const emit = defineEmits<Emits>()

const favoriteStore = useFavoriteStore()
const { isAuthenticated, getAccessToken } = useAuth0()

// Storeから状態を取得
const isFavorite = computed(() => favoriteStore.isFavorite(props.menuId))
const favoriteId = computed(() => favoriteStore.getFavoriteId(props.menuId))

const loading = ref(false)

const toggleFavorite = async () => {
  if (!isAuthenticated.value) {
    showError('ログインが必要です')
    return
  }

  loading.value = true
  try {
    const token = await getAccessToken()

    if (isFavorite.value) {
      // 削除
      await favoriteStore.removeFromFavorites(favoriteId.value!, token)
      showSuccess('お気に入りから削除しました')
    } else {
      // 追加
      await favoriteStore.addToFavorites(props.menuId, token)
      showSuccess('お気に入りに追加しました')
    }

    emit('favoriteChanged', !isFavorite.value)
  } catch (error) {
    handleError(error)
  } finally {
    loading.value = false
  }
}
</script>
```

### 2. お気に入り状態の視覚的表示

- ハートアイコンの使用（mdi-heart / mdi-heart-outline）
- お気に入り済み: 赤色の塗りつぶしハート
- 未追加: グレーのアウトラインハート
- ローディング状態の表示

### 3. Storeベースの状態管理

- `favoriteStore.isFavorite(menuId)` で状態チェック
- `favoriteStore.getFavoriteId(menuId)` でfavoriteID取得
- リアクティブな状態更新

### 4. Props と Events

- Props: menuId (必須), size (オプション)
- Events: favoriteChanged (お気に入り状態変更時)

## 実装詳細

### 初期化時の状態取得

```typescript
// コンポーネントマウント時にお気に入り一覧を取得（必要に応じて）
onMounted(async () => {
  if (isAuthenticated.value && favoriteStore.favorites.length === 0) {
    try {
      const token = await getAccessToken()
      await favoriteStore.loadFavorites(token)
    } catch (error) {
      console.error('Failed to load favorites:', error)
    }
  }
})
```

### エラーハンドリング

```typescript
const handleError = (error: any) => {
  if (error.response?.status === 401) {
    showError('ログインが必要です')
  } else if (error.response?.status === 409) {
    showError('既にお気に入りに追加されています')
  } else {
    showError('操作に失敗しました')
  }
}
```

### 楽観的更新の活用

```typescript
// Storeで楽観的更新を行うため、ボタンの状態は即座に変更される
// エラー時はStoreでロールバック処理が実行される
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
- [ ] **Storeから状態を取得する（isFavorite, getFavoriteId）**
- [ ] **個別のAPI呼び出しは行わない**
- [ ] ローディング状態を表示する
- [ ] エラーハンドリングが実装されている
- [ ] TypeScriptの型定義が正しく設定されている

## パフォーマンス向上

- **状態チェック**: Setを使用したO(1)の高速チェック
- **リアクティブ更新**: 計算プロパティによる効率的な再描画
- **楽観的更新**: ユーザー体験の向上

## 技術スタック

- Vue 3
- Vuetify 3
- TypeScript
- Composition API
- Material Design Icons
- Pinia

## 関連ファイル

- store/favorite.ts (Task 6で実装)
- Auth0認証機能 (既存)

## 注意事項

- 状態はすべてPinia Storeから取得
- 初回表示時にお気に入り一覧の取得が必要な場合がある
