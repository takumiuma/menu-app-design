# Task 1: バックエンドのデータモデルとマイグレーション設定

## 概要

Auth0認証を活用したユーザーお気に入り機能のためのGORMデータモデルを実装し、AutoMigrate機能でデータベーステーブルを自動作成する。

## 要件

- 要件4.2: ユーザー識別子をデータベースのusersテーブルに保存する
- 要件4.3: 既存ユーザーが再認証する際に既存のユーザーレコードを識別して使用する

## 実装内容

### 1. GORMのUserモデル定義

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

### 2. GORMのFavoriteモデル定義

```go
type Favorite struct {
    FavoriteID uint `gorm:"primaryKey;column:favorite_id"`
    UserID     uint `gorm:"not null;column:user_id;index"`
    MenuID     uint `gorm:"not null;column:menu_id;index"`
    CreatedAt  time.Time

    // リレーション
    User User `gorm:"foreignKey:UserID"`
}

func (Favorite) TableName() string {
    return "favorites"
}
```

### 3. AutoMigrate実行とインデックス設定

```go
// データベース初期化時
db.AutoMigrate(&User{}, &Favorite{})

// 複合ユニークインデックスの追加
db.Exec("CREATE UNIQUE INDEX IF NOT EXISTS idx_favorites_user_menu ON favorites(user_id, menu_id)")
```

## 受け入れ基準

- [ ] UserとFavoriteのGORMモデルが正しく定義されている
- [ ] AutoMigrate機能でテーブルが自動作成される
- [ ] user_id, menu_idの複合ユニークインデックスが設定されている
- [ ] Auth0 subのユニークインデックスが設定されている
- [ ] カスケード削除が適切に設定されている

## 技術スタック

- Go
- GORM
- PostgreSQL (想定)

## 関連ファイル

- データベースモデル定義ファイル
- マイグレーション実行ファイル
