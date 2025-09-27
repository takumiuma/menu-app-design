# Task 11: エラーハンドリングとユーザーフィードバック

## 概要

認証エラー、ネットワークエラー、成功/失敗時の適切なフィードバック表示を実装し、ユーザーエクスペリエンスを向上させる。

## 要件

- 要件1.4: お気に入り追加処理が完了するとユーザーに成功メッセージを表示する
- 要件2.4: お気に入り削除処理が完了するとユーザーに削除完了メッセージを表示する
- 要件3.5: ユーザーが未認証の場合はログインページにリダイレクトする

## 実装内容

### 1. 認証エラー時のログインページリダイレクト

```typescript
// composables/useErrorHandler.ts
export const useErrorHandler = () => {
  const router = useRouter()

  const handleAuthError = async (error: any) => {
    if (error.response?.status === 401) {
      await router.push('/login')
      showError('ログインが必要です')
    }
  }

  return { handleAuthError }
}
```

### 2. ネットワークエラー時のエラーメッセージ表示

- 接続エラー
- タイムアウトエラー
- サーバーエラー (500系)
- 適切なエラーメッセージの表示

### 3. 成功/失敗時の適切なフィードバック表示

```typescript
// composables/useNotification.ts
export const useNotification = () => {
  const showSuccess = (message: string) => {
    // Vuetifyのスナックバーまたはアラートで成功メッセージ表示
  }

  const showError = (message: string) => {
    // エラーメッセージ表示
  }

  const showInfo = (message: string) => {
    // 情報メッセージ表示
  }

  return { showSuccess, showError, showInfo }
}
```

### 4. FavoriteButtonでのエラーハンドリング強化

```typescript
// components/FavoriteButton.vue
const toggleFavorite = async () => {
  if (!isAuthenticated.value) {
    showError('ログインが必要です')
    return
  }

  loading.value = true
  try {
    if (isFavorite.value) {
      await favoriteStore.removeFromFavorites(favoriteId.value!, token.value)
      showSuccess('お気に入りから削除しました')
    } else {
      await favoriteStore.addToFavorites(props.menuId, token.value)
      showSuccess('お気に入りに追加しました')
    }
    emit('favoriteChanged', !isFavorite.value)
  } catch (error) {
    handleError(error)
  } finally {
    loading.value = false
  }
}
```

### 5. お気に入りページでのエラーハンドリング

```typescript
// pages/favorites.vue
const loadFavorites = async () => {
  loading.value = true
  try {
    const token = await getAccessToken()
    await favoriteStore.loadFavorites(token)
  } catch (error) {
    if (error.response?.status === 401) {
      await router.push('/login')
    } else {
      showError('お気に入りリストの取得に失敗しました')
    }
  } finally {
    loading.value = false
  }
}
```

## エラーメッセージ一覧

### 成功メッセージ

- "お気に入りに追加しました"
- "お気に入りから削除しました"

### エラーメッセージ

- "ログインが必要です" (401)
- "お気に入りの追加に失敗しました"
- "お気に入りの削除に失敗しました"
- "お気に入りリストの取得に失敗しました"
- "ネットワークエラーが発生しました"
- "既にお気に入りに追加されています" (409)

### 情報メッセージ

- "お気に入りがありません"

## 受け入れ基準

- [ ] composables/useErrorHandler.ts が作成されている
- [ ] composables/useNotification.ts が作成されている
- [ ] 401エラー時にログインページにリダイレクトされる
- [ ] ネットワークエラー時に適切なメッセージが表示される
- [ ] お気に入り追加成功時に成功メッセージが表示される
- [ ] お気に入り削除成功時に成功メッセージが表示される
- [ ] 重複追加時に適切なエラーメッセージが表示される
- [ ] FavoriteButtonでエラーハンドリングが実装されている
- [ ] お気に入りページでエラーハンドリングが実装されている
- [ ] Vuetifyのスナックバーまたはアラートを使用している
- [ ] エラーメッセージが日本語で表示される

## 技術スタック

- Nuxt 3
- Vue 3
- Vuetify 3 (v-snackbar / v-alert)
- TypeScript
- Composables

## 関連ファイル

- components/FavoriteButton.vue (Task 7で実装)
- pages/favorites.vue (Task 9で実装)
- store/favorite.ts (Task 6で実装)
- composables/useFavoriteService.ts (Task 5で実装)
