
# 第9週 座学テキスト

## 1. Laravel認証① (10/18)

Webアプリケーションの多くは、ユーザー登録やログインといった認証機能を必要とします。Laravelでは、これらの機能をゼロから構築する必要はありません。**スターターキット**と呼ばれるパッケージを導入するだけで、安全で高機能な認証システムを数分で実装できます。

### Laravel Breeze

Breezeは、Laravel公式が提供する、シンプルでカスタマイズしやすい認証スターターキットです。Tailwind CSSでスタイリングされた、基本的な認証機能（ログイン、登録、パスワードリセットなど）のルート、コントローラ、ビューをすべて自動で生成してくれます。

**Breezeのインストール手順:**

1.  **Composerでパッケージをインストール**
    ```bash
    composer require laravel/breeze --dev
    ```

2.  **Breezeのインストールコマンドを実行**
    ```bash
    php artisan breeze:install
    ```
    - 途中でどのスタック（Blade, React, Vue）を使うか尋ねられます。今回は `Blade` を選択します。
    - `Would you like to install dark mode support?` (ダークモード) -> `no`
    - `Which testing framework do you prefer?` (テスト) -> `PHPUnit` (デフォルト)

3.  **npmパッケージのインストールとビルド**
    - Breezeが生成するCSSやJSをコンパイルするために、Node.jsとnpmが必要です。
    - PCにインストールされていない場合は、[公式サイト](https://nodejs.org/ja)などからインストールしてください。
    ```bash
    # 必要なJSパッケージをインストール
    npm install

    # CSS/JSをビルド（コンパイル）
    npm run build
    ```

4.  **マイグレーションの実行**
    - Breezeはユーザー情報を格納する`users`テーブルや、パスワードリセット用のテーブルを必要とします。Laravelにデフォルトで含まれているマイグレーションを実行します。
    ```bash
    php artisan migrate
    ```

これだけで、認証に必要なすべての機能がアプリケーションに組み込まれます。

### Breezeによって追加されるもの

- **ルート (`routes/auth.php`)**: ログイン、ログアウト、ユーザー登録などのルートが定義されます。
- **コントローラ (`app/Http/Controllers/Auth/`)**: 認証処理を行うコントローラ群が生成されます。
- **ビュー (`resources/views/auth/`)**: ログイン画面や登録画面などのBladeビューが生成されます。
- **`users`テーブル**: `name`, `email`, `password` などのカラムを持つユーザーテーブルが作成されます。
- **トップページのナビゲーション**: ログイン状態に応じて「ログイン」「登録」や「ダッシュボード」「ログアウト」のリンクが自動で表示されるようになります。

`php artisan serve` と `npm run dev` (開発中にCSS/JSの変更を自動コンパイルする) を実行して、トップページにアクセスすると、右上に認証関連のリンクが追加されているのが確認できます。

---

## 2. Laravel認証② (10/19)

### Middleware (ミドルウェア)

ミドルウェアは、**リクエストがコントローラのアクションに到達する前に実行される処理層**です。複数のリクエストに共通する処理（フィルター）を記述するのに適しています。

**主な用途:**
- **認証チェック**: ユーザーがログインしているかを確認する。
- **認可制御**: ログインユーザーが特定のアクションを実行する権限を持っているか確認する。
- **ロギング**: アクセスログを記録する。
- **CORSヘッダーの付与**: APIなどでクロスオリジンリクエストを許可する。

Laravelでは、`app/Http/Kernel.php` ファイルでミドルウェアが管理されています。Breezeをインストールすると、`auth` というミドルウェアが使えるようになります。

### `auth`ミドルウェアによるアクセス制限

特定のルートやコントローラを「ログインしているユーザーのみアクセス可能」にするには、`auth`ミドルウェアを適用します。

**1. ルートに直接適用する方法 (`routes/web.php`)**

- **個別のルートに適用**
    ```php
    // このルートはログイン必須になる
    Route::get('/posts/create', [PostController::class, 'create'])->middleware('auth');
    ```

- **複数のルートにまとめて適用 (グループ化)**
    ```php
    Route::middleware('auth')->group(function () {
        Route::get('/posts/create', [PostController::class, 'create'])->name('posts.create');
        Route::post('/posts', [PostController::class, 'store'])->name('posts.store');
        // ... このグループ内のルートはすべてログイン必須になる
    });
    ```
    未ログインのユーザーがこれらのルートにアクセスしようとすると、自動的にログインページ (`/login`) にリダイレクトされます。

**2. コントローラのコンストラクタで適用する方法**

コントローラ全体、または特定のメソッドだけにミドルウェアを適用することもできます。

```php
<?php
// PostController.php

class PostController extends Controller
{
    public function __construct()
    {
        // このコントローラの全てのアクションにauthミドルウェアを適用
        $this->middleware('auth');

        // except()を使うと、特定のアクションだけ除外できる
        // 例: 投稿一覧(index)と詳細(show)は誰でも見れるようにする
        $this->middleware('auth')->except(['index', 'show']);
    }

    // ... アクションメソッド ...
}
```

### ログインユーザー情報の取得

コントローラやビューの中で、現在ログインしているユーザーの情報を取得したい場面は頻繁にあります。

- **`Auth`ファサードを使う方法**
    ```php
    use Illuminate\Support\Facades\Auth;

    // ログインしているかチェック
    if (Auth::check()) {
        // ログインユーザーのIDを取得
        $userId = Auth::id();

        // ログインユーザーのモデルインスタンスを取得
        $user = Auth::user();

        // ユーザー名を取得
        $name = $user->name;
    }
    ```

- **`$request`オブジェクトから取得する方法**
    ```php
    use Illuminate\Http\Request;

    public function store(Request $request)
    {
        $user = $request->user(); // ログインユーザーのモデルインスタンス
        // ...
    }
    ```

- **Bladeビューで使う方法**
    - Bladeの中では `@auth` ディレクティブが使えます。
    ```blade
    @auth
        // ユーザーがログインしている場合に表示
        <p>ようこそ、{{ Auth::user()->name }} さん</p>
    @endauth

    @guest
        // ユーザーがログインしていない場合に表示
        <p>ゲストさん、ようこそ！</p>
    @endguest
    ```
