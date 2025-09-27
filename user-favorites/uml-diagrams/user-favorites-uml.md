# ユーザーお気に入り機能 UML図

## 1. クラス図 (Class Diagram)

```mermaid
classDiagram
    %% データベースモデル
    class User {
        +uint UserID
        +string Auth0Sub
        +time.Time CreatedAt
        +time.Time UpdatedAt
        +[]Favorite Favorites
    }

    class Favorite {
        +uint FavoriteID
        +uint UserID
        +uint MenuID
        +time.Time CreatedAt
        +User User
    }

    class Menu {
        +uint MenuID
        +string MenuName
        +[]uint GenreIDs
        +[]uint CategoryIDs
    }

    %% フロントエンドインターフェース
    class FavoriteWithMenu {
        +number favoriteId
        +number userId
        +number menuId
        +string menuName
        +number[] genreIds
        +number[] categoryIds
        +string createdAt
    }

    %% API層
    class FavoriteAPI {
        +addFavorite(menuId: number, token: string) Promise~Favorite~
        +removeFavorite(favoriteId: number, token: string) Promise~void~
        +getFavorites(token: string) Promise~FavoriteWithMenu[]~
        +checkFavoriteStatus(menuId: number, token: string) Promise~object~
    }

    %% サービス層
    class FavoriteService {
        +findOrCreateUser(auth0Sub: string) User
        +addFavorite(userID: uint, menuID: uint) Favorite
        +removeFavorite(favoriteID: uint, userID: uint) bool
        +getFavoritesByUser(userID: uint) []FavoriteWithMenu
        +checkFavoriteStatus(userID: uint, menuID: uint) bool
    }

    %% フロントエンドコンポーネント
    class FavoriteButton {
        +number menuId
        +string size
        +boolean isFavorite
        +boolean loading
        +toggleFavorite() void
        +checkStatus() void
    }

    class FavoriteStore {
        +FavoriteWithMenu[] favorites
        +Map favoriteStatus
        +loadFavorites(token: string) Promise~void~
        +addToFavorites(menuId: number, token: string) Promise~void~
        +removeFromFavorites(favoriteId: number, token: string) Promise~void~
        +checkFavoriteStatus(menuId: number, token: string) Promise~boolean~
    }

    %% 認証
    class AuthMiddleware {
        +validateToken(token: string) bool
        +extractAuth0Sub(token: string) string
        +findOrCreateUser(auth0Sub: string) User
    }

    %% リレーション
    User ||--o{ Favorite : "1対多"
    Menu ||--o{ Favorite : "1対多"
    Favorite }o--|| User : "多対1"
    Favorite }o--|| Menu : "多対1"

    FavoriteAPI --> FavoriteService : "使用"
    FavoriteButton --> FavoriteStore : "使用"
    FavoriteStore --> FavoriteAPI : "使用"
    AuthMiddleware --> User : "作成・取得"
    FavoriteService --> User : "使用"
    FavoriteService --> Favorite : "作成・削除・取得"
```

## 2. シーケンス図 - お気に入り追加フロー

```mermaid
sequenceDiagram
    participant U as User
    participant FB as FavoriteButton
    participant FS as FavoriteStore
    participant API as FavoriteAPI
    participant AM as AuthMiddleware
    participant DB as Database

    U->>FB: クリック（お気に入り追加）
    FB->>FB: loading = true
    FB->>FS: addToFavorites(menuId, token)
    FS->>API: addFavorite(menuId, token)

    API->>AM: トークン検証
    AM->>AM: JWTトークン検証
    AM->>AM: Auth0 sub抽出
    AM->>DB: ユーザー検索/作成
    DB-->>AM: User
    AM-->>API: userID設定

    API->>DB: お気に入り追加
    alt 成功
        DB-->>API: Favorite作成
        API-->>FS: { favoriteId: 1 }
        FS->>FS: ローカル状態更新
        FS-->>FB: 成功
        FB->>FB: loading = false
        FB->>FB: isFavorite = true
        FB-->>U: 成功メッセージ表示
    else 重複エラー
        DB-->>API: 409 Conflict
        API-->>FS: エラー
        FS-->>FB: エラー
        FB->>FB: loading = false
        FB-->>U: "既にお気に入り済み"
    end
```

## 3. シーケンス図 - お気に入りページ表示フロー

```mermaid
sequenceDiagram
    participant U as User
    participant FP as FavoritesPage
    participant FS as FavoriteStore
    participant API as FavoriteAPI
    participant AM as AuthMiddleware
    participant DB as Database

    U->>FP: ページアクセス
    FP->>FP: 認証チェック
    alt 未認証
        FP-->>U: ログインページリダイレクト
    else 認証済み
        FP->>FS: loadFavorites(token)
        FS->>API: getFavorites(token)

        API->>AM: トークン検証
        AM->>AM: JWTトークン検証
        AM->>AM: Auth0 sub抽出
        AM->>DB: ユーザー検索/作成
        DB-->>AM: User
        AM-->>API: userID設定

        API->>DB: お気に入り一覧取得（JOIN）
        DB-->>API: FavoriteWithMenu[]
        API-->>FS: favorites
        FS->>FS: 状態更新
        FS-->>FP: 完了

        alt お気に入りあり
            FP-->>U: お気に入りリスト表示
        else お気に入りなし
            FP-->>U: "お気に入りがありません"
        end
    end
```

## 4. アクティビティ図 - お気に入り機能全体フロー

```mermaid
flowchart TD
    A[ユーザーログイン] --> B{認証成功?}
    B -->|No| C[ログインページ]
    B -->|Yes| D[メニューページ表示]

    D --> E{お気に入りボタン表示}
    E --> F[ユーザーがボタンクリック]

    F --> G{現在の状態}
    G -->|未追加| H[お気に入り追加API呼び出し]
    G -->|追加済み| I[お気に入り削除API呼び出し]

    H --> J[認証ミドルウェア]
    I --> J

    J --> K[トークン検証]
    K --> L{トークン有効?}
    L -->|No| M[401エラー]
    L -->|Yes| N[Auth0 sub抽出]

    N --> O[ユーザー検索/作成]
    O --> P{追加 or 削除?}

    P -->|追加| Q[重複チェック]
    Q --> R{既存?}
    R -->|Yes| S[409エラー]
    R -->|No| T[お気に入り作成]

    P -->|削除| U[権限チェック]
    U --> V{自分のお気に入り?}
    V -->|No| W[403エラー]
    V -->|Yes| X[お気に入り削除]

    T --> Y[成功レスポンス]
    X --> Y
    Y --> Z[フロントエンド状態更新]
    Z --> AA[UI更新]
    AA --> AB[成功メッセージ表示]

    M --> AC[エラーメッセージ表示]
    S --> AC
    W --> AC
```

## 5. コンポーネント図 - システム構成

```mermaid
graph TB
    subgraph "Frontend (Nuxt 3)"
        subgraph "Pages"
            P1[index.vue]
            P2[menuList.vue]
            P3[favorites.vue]
        end

        subgraph "Components"
            C1[FavoriteButton.vue]
            C2[TheHeader.vue]
        end

        subgraph "Composables"
            CO1[useFavoriteService.ts]
            CO2[useAxiosService.ts]
        end

        subgraph "Store"
            S1[favorite.ts]
        end

        subgraph "Auth"
            A1[Auth0 Integration]
        end
    end

    subgraph "Backend API"
        subgraph "Middleware"
            M1[AuthMiddleware]
        end

        subgraph "Endpoints"
            E1[POST /v1/favorites]
            E2[DELETE /v1/favorites/:id]
            E3[GET /v1/favorites]
            E4[GET /v1/favorites/check/:menuId]
        end

        subgraph "Services"
            SV1[FavoriteService]
            SV2[UserService]
        end
    end

    subgraph "Database"
        DB1[(users)]
        DB2[(favorites)]
        DB3[(menus)]
    end

    subgraph "External"
        EX1[Auth0]
    end

    %% 接続関係
    P1 --> C1
    P2 --> C1
    P3 --> C1
    C1 --> S1
    S1 --> CO1
    CO1 --> CO2
    CO2 --> E1
    CO2 --> E2
    CO2 --> E3
    CO2 --> E4

    E1 --> M1
    E2 --> M1
    E3 --> M1
    E4 --> M1

    M1 --> SV2
    E1 --> SV1
    E2 --> SV1
    E3 --> SV1
    E4 --> SV1

    SV1 --> DB1
    SV1 --> DB2
    SV1 --> DB3
    SV2 --> DB1

    A1 --> EX1
    M1 --> EX1
```

## 6. ER図 - データベース設計

```mermaid
erDiagram
    users {
        uint user_id PK
        string auth0_sub UK
        timestamp created_at
        timestamp updated_at
    }

    favorites {
        uint favorite_id PK
        uint user_id FK
        uint menu_id FK
        timestamp created_at
    }

    menus {
        uint menu_id PK
        string menu_name
        json genre_ids
        json category_ids
        timestamp created_at
        timestamp updated_at
    }

    users ||--o{ favorites : "1対多"
    menus ||--o{ favorites : "1対多"
```

## 7. 状態遷移図 - お気に入りボタンの状態

```mermaid
stateDiagram-v2
    [*] --> Loading : 初期化
    Loading --> NotFavorite : 未お気に入り
    Loading --> Favorite : お気に入り済み

    NotFavorite --> Adding : クリック
    Adding --> Favorite : 追加成功
    Adding --> NotFavorite : 追加失敗
    Adding --> Error : ネットワークエラー

    Favorite --> Removing : クリック
    Removing --> NotFavorite : 削除成功
    Removing --> Favorite : 削除失敗
    Removing --> Error : ネットワークエラー

    Error --> NotFavorite : リトライ成功
    Error --> Favorite : リトライ成功
    Error --> Error : リトライ失敗
```

これらのUML図により、ユーザーお気に入り機能の全体像を多角的に理解できます。各図は異なる観点からシステムを表現しており、開発・保守・拡張時の参考資料として活用できます。
