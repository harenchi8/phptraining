
# 第9週 実践課題

**目的:**
Laravel Breezeを導入して、高機能な認証システムをアプリケーションに組み込みます。さらに、ミドルウェアを使って特定のページへのアクセスをログインユーザーのみに制限する方法を学びます。

---

## 課題1: Breeze認証の導入とUI実装 (10/18)

**目的:**
数ステップのコマンドで、実際に動作するユーザー登録・ログイン・ログアウト機能を実装します。

**手順:**

1.  **Node.jsとnpmのインストール確認**:
    *   ターミナルで `node -v` と `npm -v` を実行し、バージョン情報が表示されることを確認します。エラーが出る場合は、先にNode.jsをインストールしてください。

2.  **Laravel Breezeのインストール**:
    *   `laravel-training` プロジェクトのルートディレクトリで、以下のコマンドを順番に実行します。
        ```bash
        # 1. ComposerでBreezeパッケージをダウンロード
        composer require laravel/breeze --dev

        # 2. Breezeをインストール (Bladeスタックを選択)
        php artisan breeze:install
        ```
    *   途中の質問には以下のように答えます。
        - `Which stack would you like to install?` -> `blade`
        - `Would you like to install dark mode support?` -> `no`
        - `Which testing framework do you prefer?` -> `phpunit` (EnterキーでOK)

3.  **フロントエンドのビルド**:
    *   引き続きターミナルで、以下のコマンドを実行します。
        ```bash
        # 3. 必要なnpmパッケージをインストール
        npm install

        # 4. CSS/JSをビルド
        npm run build
        ```

4.  **データベースのマイグレーション**:
    *   Breezeが必要とする `users` テーブルなどをデータベースに作成します。
        ```bash
        # 5. マイグレーションを実行
        php artisan migrate
        ```

5.  **動作確認**:
    *   `php artisan serve` で開発サーバーを起動します。
    *   ブラウザで `http://127.0.0.1:8000` にアクセスします。
    *   画面右上に「Log in」と「Register」のリンクが表示されていることを確認します。
    *   **ユーザー登録**: 「Register」リンクから、新しいユーザーを登録してみます。登録後、「Dashboard」という画面にリダイレクトされることを確認します。
    *   **ログアウト**: 右上のユーザー名をクリックし、「Log Out」を選択します。トップページに戻ることを確認します。
    *   **ログイン**: 「Log in」リンクから、先ほど登録した情報でログインできることを確認します。

---

## 課題2: Middlewareによる認可制御 (10/19)

**目的:**
第8週で作成した投稿機能（作成、保存）を、ログインしているユーザーしか利用できないようにアクセス制限をかけます。また、投稿をユーザー情報と紐付けます。

**手順:**

1.  **投稿テーブル(`posts`)の修正**:
    *   投稿がどのユーザーによって作成されたかを記録するため、`posts`テーブルに`user_id`カラムを追加します。
    *   新しいマイグレーションファイルを作成します。
        ```bash
        php artisan make:migration add_user_id_to_posts_table --table=posts
        ```
    *   作成されたマイグレーションファイル (`database/migrations/...`) の `up` メソッドを編集します。
        ```php
        public function up()
        {
            Schema::table('posts', function (Blueprint $table) {
                // created_atカラムの後にuser_idカラムを追加
                $table->foreignId('user_id')->after('id')->constrained()->onDelete('cascade');
            });
        }
        ```
        > `foreignId`は外部キー制約付きの`user_id`カラムを作成します。`onDelete('cascade')`は、ユーザーが削除されたらそのユーザーの投稿も一緒に削除する設定です。
    *   `down` メソッドも編集して、ロールバック時にカラムを削除するようにします。
        ```php
        public function down()
        {
            Schema::table('posts', function (Blueprint $table) {
                $table->dropForeign(['user_id']);
                $table->dropColumn('user_id');
            });
        }
        ```
    *   `php artisan migrate` を実行してテーブル構造を更新します。

2.  **モデルの関連付け**:
    *   `app/Models/User.php` に、一人のユーザーが多くの投稿を持つ(hasMany)関係を定義します。
        ```php
        // User.php
        public function posts()
        {
            return $this->hasMany(Post::class);
        }
        ```
    *   `app/Models/Post.php` に、一つの投稿は一人のユーザーに属する(belongsTo)関係を定義します。
        ```php
        // Post.php
        public function user()
        {
            return $this->belongsTo(User::class);
        }
        ```

3.  **ルートの保護 (`routes/web.php`)**:
    *   投稿の作成(create)と保存(store)のルートを `auth` ミドルウェアで保護します。
        ```php
        Route::middleware('auth')->group(function () {
            Route::get('/posts/create', [PostController::class, 'create'])->name('posts.create');
            Route::post('/posts', [PostController::class, 'store'])->name('posts.store');
        });

        // 誰でも見れるルート
        Route::get('/posts', [PostController::class, 'index'])->name('posts.index');
        ```

4.  **コントローラの修正 (`PostController.php`)**:
    *   `store` メソッドを修正し、ログインしているユーザーのIDを `user_id` として保存するようにします。
        ```php
        // storeメソッド内
        public function store(StorePostRequest $request)
        {
            $post = new Post($request->validated());
            $post->user_id = $request->user()->id; // ログインユーザーのIDを取得してセット
            $post->save();

            return redirect()->route('posts.index')->with('success', '投稿しました。');
        }
        ```

5.  **ビューの修正 (`posts/index.blade.php`)**:
    *   投稿一覧に、投稿者の名前も表示するように変更します。
        ```blade
        @foreach ($posts as $post)
            <tr>
                <td>{{ $post->id }}</td>
                <td>{{ $post->title }}</td>
                <td>{{ $post->user->name }}</td> {{-- リレーションを使ってユーザー名を表示 --}}
                <td>{{ $post->created_at }}</td>
            </tr>
        @endforeach
        ```
    *   `PostController`の`index`メソッドで、`with('user')` を使ってN+1問題を解消しておきましょう（Eager Loading）。
        ```php
        // indexメソッド内
        $posts = Post::with('user')->latest()->get();
        return view('posts.index', compact('posts'));
        ```

**動作確認:**
- ログアウト状態で `/posts/create` にアクセスしようとすると、ログインページにリダイレクトされることを確認します。
- ログイン状態で投稿を作成し、一覧ページで自分の名前が表示されることを確認します。
- phpMyAdminで `posts` テーブルを開き、`user_id` カラムに正しいユーザーIDが記録されていることを確認します。
