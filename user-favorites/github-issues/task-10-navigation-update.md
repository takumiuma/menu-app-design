# Task 10: ナビゲーションメニューの更新

## 概要

TheHeader.vueにお気に入りページへのリンクを追加し、認証済みユーザーのみ表示するよう制御する。

## 要件

- 要件3.1: ユーザーがお気に入りページにアクセスできる
- 要件3.5: ユーザーが未認証の場合は適切にアクセス制御される

## 実装内容

### 1. TheHeader.vueのナビゲーション項目追加

```typescript
const ITEMS = [
  { title: 'ホーム', icon: 'mdi-food', link: '/' },
  { title: 'メニューリスト', icon: 'mdi-table-edit', link: '/menuList' },
  { title: 'お気に入り', icon: 'mdi-heart', link: '/favorites', authRequired: true },
]
```

### 2. 認証済みユーザーのみ表示する制御

- Auth0の認証状態を確認
- 認証済みユーザーのみお気に入りリンクを表示
- 未認証ユーザーには表示しない

### 3. ナビゲーションドロワーの更新

```vue
<template>
  <v-navigation-drawer
    v-model="drawer"
    :location="$vuetify.display.mobile ? 'top' : undefined"
    temporary
  >
    <v-list>
      <v-list-item v-for="item in filteredItems" :key="item.title" :to="item.link">
        <v-icon>{{ item.icon }}</v-icon>
        {{ item.title }}
      </v-list-item>
    </v-list>
    <!-- 既存のログイン/ログアウトボタン -->
  </v-navigation-drawer>
</template>
```

### 4. アイテムフィルタリングロジック

```typescript
const filteredItems = computed(() => {
  return ITEMS.filter((item) => {
    if (item.authRequired) {
      return isAuthenticated.value
    }
    return true
  })
})
```

## 実装詳細

### 認証状態の取得

- 既存のAuth0認証状態を活用
- isAuthenticated の値を使用
- リアクティブな表示制御

### アイコンの選択

- お気に入りページ: `mdi-heart`
- 既存のアイコンとの一貫性を保持

### レスポンシブ対応

- 既存のモバイル対応を維持
- ドロワーの位置制御を継承

## 受け入れ基準

- [ ] components/TheHeader.vue にお気に入りリンクが追加されている
- [ ] お気に入りリンクのアイコンが `mdi-heart` になっている
- [ ] 認証済みユーザーのみお気に入りリンクが表示される
- [ ] 未認証ユーザーにはお気に入りリンクが表示されない
- [ ] お気に入りリンクをクリックすると `/favorites` に遷移する
- [ ] 既存のナビゲーション項目が正常に動作する
- [ ] モバイル対応が維持されている
- [ ] 認証状態の変更に応じてリアクティブに更新される

## 技術スタック

- Nuxt 3
- Vue 3
- Vuetify 3
- TypeScript
- Auth0
- Vue Router

## 関連ファイル

- components/TheHeader.vue (既存)
- pages/favorites.vue (Task 9で実装)
- Auth0認証機能 (既存)
