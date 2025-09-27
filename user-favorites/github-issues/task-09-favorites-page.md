# Task 9: お気に入りページの実装

## 概要

お気に入りメニューリストを表示する専用ページ `pages/favorites.vue` を作成する。空のリスト時のメッセージ表示と各アイテムからの削除機能を含む。

## 要件

- 要件3.1: ユーザーがお気に入りページにアクセスするとそのユーザーのお気に入りメニューアイテムのみを表示する
- 要件3.2: お気に入りリストが空の場合は「お気に入りがありません」等の適切なメッセージを表示する
- 要件3.3: お気に入りリストにアイテムがある場合は各アイテムの詳細情報（名前、ジャンル等）を表示する
- 要件3.4: お気に入りリストでアイテムを表示している際に各アイテムからお気に入り削除ができる
- 要件3.5: ユーザーが未認証の場合はログインページにリダイレクトする

## 実装内容

### 1. pages/favorites.vue ページ作成

```vue
<template>
  <v-container>
    <h1>お気に入りメニュー</h1>

    <!-- ローディング状態 -->
    <v-progress-circular v-if="loading" indeterminate />

    <!-- お気に入りリスト -->
    <v-row v-else-if="favorites.length > 0">
      <v-col v-for="favorite in favorites" :key="favorite.favoriteId" cols="12" md="6" lg="4">
        <v-card>
          <v-card-title>{{ favorite.menuName }}</v-card-title>
          <v-card-text>
            <div>ジャンル: {{ getGenreNames(favorite.genreIds) }}</div>
            <div>カテゴリ: {{ getCategoryNames(favorite.categoryIds) }}</div>
            <div class="text-caption">追加日: {{ formatDate(favorite.createdAt) }}</div>
          </v-card-text>
          <v-card-actions>
            <v-spacer />
            <FavoriteButton
              :menu-id="favorite.menuId"
              @favorite-changed="onFavoriteRemoved(favorite.favoriteId)"
            />
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>

    <!-- 空のお気に入りリスト -->
    <v-alert v-else type="info" class="mt-4"> お気に入りがありません </v-alert>
  </v-container>
</template>
```

### 2. 認証チェックとリダイレクト

- ページアクセス時にAuth0認証状態を確認
- 未認証の場合はログインページにリダイレクト
- 認証済みの場合はお気に入りリストを取得

### 3. お気に入りリストの表示

- useFavoriteStoreを使用してお気に入りリストを取得
- カード形式でメニュー情報を表示
- ジャンル・カテゴリ名の表示
- 追加日時の表示

### 4. 空のリスト時のメッセージ表示

- お気に入りが0件の場合の適切なメッセージ
- v-alertコンポーネントを使用

### 5. 各アイテムからのお気に入り削除機能

- FavoriteButtonコンポーネントを使用
- 削除時のリスト更新処理

## 実装詳細

### 認証チェック

```typescript
const { isAuthenticated } = useAuth0()
const router = useRouter()

onMounted(async () => {
  if (!isAuthenticated.value) {
    await router.push('/login')
    return
  }
  await loadFavorites()
})
```

### お気に入りリスト取得

```typescript
const favoriteStore = useFavoriteStore()
const loading = ref(true)

const loadFavorites = async () => {
  try {
    const token = await getAccessToken()
    await favoriteStore.loadFavorites(token)
  } catch (error) {
    // エラーハンドリング
  } finally {
    loading.value = false
  }
}
```

## 受け入れ基準

- [ ] pages/favorites.vue ファイルが作成されている
- [ ] 認証チェックが実装されている
- [ ] 未認証ユーザーはログインページにリダイレクトされる
- [ ] お気に入りリストが正しく表示される
- [ ] 空のリスト時に適切なメッセージが表示される
- [ ] メニュー名、ジャンル、カテゴリが表示される
- [ ] 追加日時が表示される
- [ ] FavoriteButtonが各アイテムに表示される
- [ ] お気に入り削除時にリストが更新される
- [ ] ローディング状態が表示される
- [ ] エラーハンドリングが実装されている

## 技術スタック

- Nuxt 3
- Vue 3
- Vuetify 3
- TypeScript
- Auth0
- Pinia

## 関連ファイル

- store/favorite.ts (Task 6で実装)
- components/FavoriteButton.vue (Task 7で実装)
- Auth0認証機能 (既存)
