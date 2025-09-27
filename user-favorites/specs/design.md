# 設計文書

## 概要

Auth0認証機能を活用したユーザーお気に入り機能の設計文書です。既存のMenuAppアーキテクチャに統合し、認証されたユーザーが個人のお気に入りメニューリストを管理できる機能を提供します。

## アーキテクチャ

### システム構成

```
Frontend (Nuxt 3)
├── Pages
│   ├── index.vue (お気に入りボタン追加)
│   ├── menuList.vue (お気に入りボタン追加)
│   └── favorites.vue (新規作成)
├── Components
│   ├── FavoriteButton.vue (新規作成)
│   └── TheHeader.vue (ナビゲーション追加)
├── Composables
│   └── useFavoriteService.ts (新規作成)
├── Store
│   └── favorite.ts (新規作成)
└── Auth0 Integration (既存)

Backend API
├── /v1/users (新規エンドポイント)
├── /v1/favorites (新規エンドポイント)
└── Database
    ├── users テーブル (新規)
    └── favorites テーブル (新規)
```

### データフロー

1. **認証フロー**: Auth0 → フロントエンド → バックエンド（ユーザー登録/識別）
2. **お気に入り操作**: フロントエンド → API → データベース → レスポンス
3. **状態管理**: Pinia Store → コンポーネント → UI更新

## コンポーネントとインターフェース

### 1. データモデル

#### User Interface

```typescript
interface User {
  userId: number
  auth0Sub: string // Auth0のユーザー識別子
  createdAt: string
  updatedAt: string
}
```

#### Favorite Interface

```typescript
interface Favorite {
  favoriteId: number
  userId: number
  menuId: number
  createdAt: string
}
```

#### FavoriteWithMenu Interface

```typescript
interface FavoriteWithMenu {
  favoriteId: number
  userId: number
  menuId: number
  menuName: string
  genreIds: number[]
  categoryIds: number[]
  createdAt: string
}
```

### 2. API エンドポイント

#### ユーザー管理

- `POST /v1/users` - ユーザー作成/取得
  - Request: `{ auth0Sub: string }`
  - Response: `{ user: User }`

#### お気に入り管理

- `GET /v1/favorites` - ユーザーのお気に入り一覧取得
  - Headers: `Authorization: Bearer <token>`
  - Response: `{ favorites: FavoriteWithMenu[] }`
  - **認証チェック**: トークンからユーザーIDを取得し、そのユーザーのお気に入りのみ返却

- `POST /v1/favorites` - お気に入り追加
  - Headers: `Authorization: Bearer <token>`
  - Request: `{ menuId: number }`
  - Response: `{ favorite: Favorite }`
  - **認証チェック**: トークンからユーザーIDを取得し、そのユーザーのお気に入りとして追加

- `DELETE /v1/favorites/:favoriteId` - お気に入り削除
  - Headers: `Authorization: Bearer <token>`
  - Response: `{ success: boolean }`
  - **認証チェック**: 削除対象のお気に入りが認証ユーザーのものかチェック

- `GET /v1/favorites/check/:menuId` - お気に入り状態確認
  - Headers: `Authorization: Bearer <token>`
  - Response: `{ isFavorite: boolean, favoriteId?: number }`
  - **認証チェック**: トークンからユーザーIDを取得し、そのユーザーのお気に入り状態のみ確認

### 3. フロントエンドコンポーネント

#### FavoriteButton.vue

```vue
<template>
  <v-btn
    :icon="isFavorite ? 'mdi-heart' : 'mdi-heart-outline'"
    :color="isFavorite ? 'red' : 'grey'"
    @click="toggleFavorite"
    :loading="loading"
  />
</template>
```

**Props:**

- `menuId: number` - 対象メニューID
- `size?: string` - ボタンサイズ

**Events:**

- `@favoriteChanged` - お気に入り状態変更時

#### favorites.vue (新規ページ)

```vue
<template>
  <v-container>
    <h1>お気に入りメニュー</h1>
    <v-row v-if="favorites.length > 0">
      <v-col v-for="favorite in favorites" :key="favorite.favoriteId">
        <!-- メニューカード表示 -->
      </v-col>
    </v-row>
    <v-alert v-else type="info"> お気に入りがありません </v-alert>
  </v-container>
</template>
```

### 4. Composable

#### useFavoriteService.ts

```typescript
export const useFavoriteService = () => {
  const { axios } = useAxiosService()

  const addFavorite = async (menuId: number, token: string): Promise<Favorite>
  const removeFavorite = async (favoriteId: number, token: string): Promise<void>
  const getFavorites = async (token: string): Promise<FavoriteWithMenu[]>
  const checkFavoriteStatus = async (menuId: number, token: string): Promise<{isFavorite: boolean, favoriteId?: number}>

  return {
    addFavorite,
    removeFavorite,
    getFavorites,
    checkFavoriteStatus
  }
}
```

### 5. Pinia Store

#### favorite.ts

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

## データモデル

### データベーステーブル設計

#### GORM モデル定義

##### User モデル

```go
type User struct {
    UserID    uint   `gorm:"primaryKey;column:user_id"`
    Auth0Sub  string `gorm:"uniqueIndex;not null;column:auth0_sub"`
    CreatedAt time.Time
    UpdatedAt time.Time

    // リレーション
    Favorites []Favorite `gorm:"foreignKey:UserID;constraint:OnDelete:CASCADE"`
}
```

##### Favorite モデル

```go
type Favorite struct {
    FavoriteID uint `gorm:"primaryKey;column:favorite_id"`
    UserID     uint `gorm:"not null;column:user_id;index"`
    MenuID     uint `gorm:"not null;column:menu_id;index"`
    CreatedAt  time.Time

    // リレーション
    User User `gorm:"foreignKey:UserID"`
}

// 複合ユニークインデックス
func (Favorite) TableName() string {
    return "favorites"
}
```

#### AutoMigrate実行

```go
// データベース初期化時
db.AutoMigrate(&User{}, &Favorite{})

// 複合ユニークインデックスの追加
db.Exec("CREATE UNIQUE INDEX IF NOT EXISTS idx_favorites_user_menu ON favorites(user_id, menu_id)")
```

### リレーションシップ

- `users` 1:N `favorites` (一人のユーザーは複数のお気に入りを持つ)
- `favorites` N:1 `menus` (複数のお気に入りは一つのメニューを参照)

## エラーハンドリング

### フロントエンド

1. **認証エラー**: ログインページへリダイレクト
2. **ネットワークエラー**: ユーザーにエラーメッセージ表示
3. **重複追加エラー**: 既にお気に入り済みメッセージ表示
4. **削除エラー**: 削除失敗メッセージ表示

### バックエンド

1. **認証トークン無効**: 401 Unauthorized
2. **ユーザー未登録**: 自動ユーザー作成
3. **重複お気に入り**: 409 Conflict
4. **存在しないメニュー**: 404 Not Found
5. **データベースエラー**: 500 Internal Server Error

## テスト戦略

### ユニットテスト

- **FavoriteButton.vue**: お気に入り状態の表示・切り替え
- **useFavoriteService.ts**: API通信ロジック
- **favorite.ts**: ストア状態管理

### 統合テスト

- **お気に入り追加フロー**: 認証 → API呼び出し → 状態更新
- **お気に入り削除フロー**: 削除 → API呼び出し → 状態更新
- **お気に入りページ**: データ取得 → 表示

### E2Eテスト

- **認証済みユーザーのお気に入り操作**
- **お気に入りページの表示・操作**
- **未認証ユーザーのアクセス制御**

## セキュリティ考慮事項

### 認証・認可

- Auth0トークンによるAPI認証
- **バックエンド認証チェック**: 全てのお気に入り関連APIでトークン検証を実装
- **ユーザー分離**: ユーザーは自分のお気に入りのみアクセス可能（他ユーザーのデータは完全に隔離）
- **トークン検証**: JWTトークンからAuth0 subを抽出し、データベースのユーザーIDと照合
- トークン有効期限の適切な管理

### データ保護

- Auth0 subのみ保存（個人情報最小化）
- SQLインジェクション対策
- CORS設定の適切な管理

### プライバシー

- ユーザーのお気に入り情報は他ユーザーから非表示
- データ削除時の完全削除保証
