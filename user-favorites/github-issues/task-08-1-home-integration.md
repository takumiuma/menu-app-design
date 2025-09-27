# Task 8.1: ホームページ（index.vue）への統合

## 概要

ホームページ（index.vue）のメニューアイテム表示部分にFavoriteButtonコンポーネントを追加し、認証状態に応じたボタン表示制御を実装する。

## 要件

- 要件1.1: 認証済みの状態でメニューアイテムを表示している際にお気に入りボタン（ハートアイコン等）を表示する
- 要件1.5: ユーザーが未認証の場合はお気に入りボタンは表示されない

## 実装内容

### 1. FavoriteButtonコンポーネントの統合

- メニューアイテム表示部分にFavoriteButtonを追加
- 各メニューアイテムのmenuIdを渡す
- 適切なサイズとレイアウトで配置

### 2. 認証状態に応じたボタン表示制御

- Auth0の認証状態を確認
- 認証済みユーザーのみボタンを表示
- 未認証ユーザーには表示しない

### 3. レイアウトとデザインの調整

- 既存のデザインに馴染むよう配置
- Vuetifyのグリッドシステムを活用
- レスポンシブ対応

## 実装例

```vue
<template>
  <v-container>
    <!-- 既存のメニューフィルタリング機能 -->

    <v-row>
      <v-col v-for="menu in filteredMenus" :key="menu.menuId" cols="12" md="6" lg="4">
        <v-card>
          <v-card-title>{{ menu.menuName }}</v-card-title>
          <v-card-text>
            <!-- メニュー詳細情報 -->
          </v-card-text>
          <v-card-actions>
            <!-- 既存のアクション -->
            <v-spacer />
            <FavoriteButton
              v-if="isAuthenticated"
              :menu-id="menu.menuId"
              @favorite-changed="onFavoriteChanged"
            />
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup lang="ts">
// Auth0認証状態の取得
// FavoriteButtonコンポーネントのインポート
// イベントハンドリング
</script>
```

## 受け入れ基準

- [ ] pages/index.vue にFavoriteButtonが統合されている
- [ ] 各メニューアイテムに対してFavoriteButtonが表示される
- [ ] 認証済みユーザーのみボタンが表示される
- [ ] 未認証ユーザーにはボタンが表示されない
- [ ] menuIdが正しくFavoriteButtonに渡される
- [ ] favoriteChangedイベントを適切に処理する
- [ ] 既存のレイアウトを崩さない
- [ ] レスポンシブデザインが維持される
- [ ] Auth0認証状態を正しく取得している

## 技術スタック

- Nuxt 3
- Vue 3
- Vuetify 3
- TypeScript
- Auth0

## 関連ファイル

- pages/index.vue (既存)
- components/FavoriteButton.vue (Task 7で実装)
- Auth0認証機能 (既存)
