# Task 8.2: メニューリストページ（menuList.vue）への統合

## 概要

メニューリストページ（menuList.vue）のメニューリスト表示部分にFavoriteButtonコンポーネントを追加し、認証状態に応じたボタン表示制御を実装する。

## 要件

- 要件1.1: 認証済みの状態でメニューアイテムを表示している際にお気に入りボタン（ハートアイコン等）を表示する
- 要件1.5: ユーザーが未認証の場合はお気に入りボタンは表示されない

## 実装内容

### 1. FavoriteButtonコンポーネントの統合

- メニューリスト表示部分にFavoriteButtonを追加
- 各メニューアイテムのmenuIdを渡す
- テーブル形式のレイアウトに適したサイズで配置

### 2. 認証状態に応じたボタン表示制御

- Auth0の認証状態を確認
- 認証済みユーザーのみボタンを表示
- 未認証ユーザーには表示しない

### 3. テーブルレイアウトでの配置

- 既存のメニュー管理インターフェースに統合
- アクション列にFavoriteButtonを追加
- 編集・削除ボタンとの適切な配置

## 実装例

```vue
<template>
  <v-container>
    <v-data-table :headers="headers" :items="menus" class="elevation-1">
      <!-- 既存のスロット -->

      <template v-slot:item.actions="{ item }">
        <v-btn icon @click="editMenu(item)">
          <v-icon>mdi-pencil</v-icon>
        </v-btn>
        <v-btn icon @click="deleteMenu(item)">
          <v-icon>mdi-delete</v-icon>
        </v-btn>
        <FavoriteButton
          v-if="isAuthenticated"
          :menu-id="item.menuId"
          size="small"
          @favorite-changed="onFavoriteChanged"
        />
      </template>
    </v-data-table>
  </v-container>
</template>

<script setup lang="ts">
// Auth0認証状態の取得
// FavoriteButtonコンポーネントのインポート
// テーブルヘッダーの調整
// イベントハンドリング
</script>
```

## 受け入れ基準

- [ ] pages/menuList.vue にFavoriteButtonが統合されている
- [ ] データテーブルのアクション列にFavoriteButtonが表示される
- [ ] 認証済みユーザーのみボタンが表示される
- [ ] 未認証ユーザーにはボタンが表示されない
- [ ] menuIdが正しくFavoriteButtonに渡される
- [ ] favoriteChangedイベントを適切に処理する
- [ ] 既存の編集・削除ボタンとの配置が適切
- [ ] テーブルレイアウトが崩れない
- [ ] smallサイズでボタンが表示される
- [ ] Auth0認証状態を正しく取得している

## 技術スタック

- Nuxt 3
- Vue 3
- Vuetify 3 (v-data-table)
- TypeScript
- Auth0

## 関連ファイル

- pages/menuList.vue (既存)
- components/FavoriteButton.vue (Task 7で実装)
- Auth0認証機能 (既存)
