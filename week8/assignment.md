
# 第8週 実践課題

**目的:**
マイグレーションでテーブルを定義し、Eloquentモデルを通じてデータベースに投稿データをCRUDする一連の流れを実装します。さらに、フォームリクエストによる堅牢なバリデーションを導入します。

---

## 課題1: 投稿フォーム→DB保存→一覧表示 (10/11)

**目的:**
Artisanコマンドを駆使してマイグレーションとモデルを作成し、Eloquent ORMを使ってデータベースへの保存と読み出しを実装します。

**手順:**

1.  **`.env`ファイルの編集**:
    *   `laravel-training` プロジェクトの `.env` ファイルを開きます。
    *   `DB_*` で始まる行を、XAMPPのphpMyAdmin環境に合わせて設定します。通常はデータベース名を変えるだけでOKです。
        ```env
        DB_CONNECTION=mysql
        DB_HOST=127.0.0.1
        DB_PORT=3306
        DB_DATABASE=laravel_training_db # 新しいDB名
        DB_USERNAME=root
        DB_PASSWORD=
        ```
    *   phpMyAdminで、この `laravel_training_db` という名前の空のデータベースを作成しておきます。

2.  **モデルとマイグレーションの作成**:
    *   ターミナルで以下のArtisanコマンドを実行し、`Post`モデルと`posts`テーブル用のマイグレーションファイルを同時に作成します。
        ```bash
        php artisan make:model Post -m
        ```

3.  **マイグレーションファイルの編集**:
    *   `database/migrations/` に作成された `..._create_posts_table.php` ファイルを開きます。
    *   `up()` メソッドの中を編集し、投稿の「タイトル(title)」と「本文(body)」を保存するカラムを追加します。
        ```php
        public function up()
        {
            Schema::create('posts', function (Blueprint $table) {
                $table->id();
                $table->string('title', 255); // titleカラム (VARCHAR, 255文字)
                $table->text('body');      // bodyカラム (TEXT)
                $table->timestamps();
            });
        }
        ```

4.  **マイグレーションの実行**:
    *   ターミナルで以下のコマンドを実行し、データベースに `posts` テーブルを作成します。
        ```bash
        php artisan migrate
        ```
    *   phpMyAdminで `laravel_training_db` を確認し、`posts` テーブルが作成されていることを確認します。

5.  **ルーティングの定義 (`routes/web.php`)**:
    *   投稿一覧ページ (`/posts`) と投稿フォームページ (`/posts/create`)、投稿処理 (`/posts`) のルートを定義します。
        ```php
        use App\Http\Controllers\PostController;

        // 投稿一覧
        Route::get('/posts', [PostController::class, 'index'])->name('posts.index');
        // 投稿フォーム
        Route::get('/posts/create', [PostController::class, 'create'])->name('posts.create');
        // 投稿処理
        Route::post('/posts', [PostController::class, 'store'])->name('posts.store');
        ```
        > `.name()` はルートに名前をつけるメソッドで、後のリダイレクトなどで便利です。

6.  **コントローラとビューの作成**:
    *   `php artisan make:controller PostController` でコントローラを作成します。
    *   `PostController` に `index`, `create`, `store` の3つのメソッドを実装します。
    *   `store` メソッドでは、Eloquentを使ってDBにデータを保存します。（この時点ではバリデーションは不要）
        ```php
        // PostController.php の store メソッド内
        public function store(Request $request) // Requestクラスをuseする
        {
            $post = new Post();
            $post->title = $request->input('title');
            $post->body = $request->input('body');
            $post->save();

            return redirect()->route('posts.index');
        }
        ```
    *   `index` メソッドでは、`Post::all()` ですべての投稿を取得し、ビューに渡します。
    *   対応するビューファイル (`resources/views/posts/index.blade.php`, `resources/views/posts/create.blade.php`) を作成します。
    *   `create.blade.php` には投稿フォームを、`index.blade.php` には投稿一覧を表示するコードを記述します。

**動作確認:**
- `/posts/create` にアクセスしてフォームから投稿し、`/posts` の一覧に表示されることを確認します。

---

## 課題2: 入力チェック付き投稿実装 (10/12)

**目的:**
課題1で作成した投稿機能に、フォームリクエストを使ったバリデーションを追加します。

**手順:**

1.  **フォームリクエストの作成**:
    *   ターミナルで以下のコマンドを実行し、`StorePostRequest` を作成します。
        ```bash
        php artisan make:request StorePostRequest
        ```

2.  **フォームリクエストの編集 (`app/Http/Requests/StorePostRequest.php`)**:
    *   `authorize()` メソッドが `true` を返すように変更します。
    *   `rules()` メソッドに、`title` と `body` のバリデーションルールを定義します。
        ```php
        public function rules()
        {
            return [
                'title' => 'required|max:50', // 必須、50文字以内
                'body'  => 'required',        // 必須
            ];
        }
        ```

3.  **Postモデルの編集 (`app/Models/Post.php`)**:
    *   マスアサインメント（`Post::create()`）を使えるようにするため、`$fillable` プロパティを追加します。
        ```php
        protected $fillable = ['title', 'body'];
        ```

4.  **コントローラの修正 (`PostController.php`)**:
    *   `store` メソッドの引数の型を `Illuminate\Http\Request` から `App\Http\Requests\StorePostRequest` に変更します。
    *   `store` メソッドの中身を、`Post::create()` を使った、よりシンプルな形に書き換えます。
        ```php
        // use App\Http\Requests\StorePostRequest; を忘れないように
        public function store(StorePostRequest $request)
        {
            Post::create($request->validated());

            return redirect()->route('posts.index')->with('success', '投稿しました。');
        }
        ```
        > `$request->validated()` はバリデーション済みのデータを配列で返します。
        > `->with(...)` は、リダイレクト先に一時的なメッセージ（フラッシュメッセージ）を渡すメソッドです。

5.  **ビューの修正 (`resources/views/posts/create.blade.php`)**:
    *   フォームの上に、バリデーションエラーメッセージを表示するコードを追加します（`lecture.md`参照）。
    *   各`input`や`textarea`に、`old('title')` のように入力値を復元するコードを追加します。

6.  **フラッシュメッセージの表示 (`resources/views/posts/index.blade.php`)**:
    *   一覧ページの上部に、`session('success')` の中身があれば表示するコードを追加します。
        ```blade
        @if (session('success'))
            <div class="alert alert-success">
                {{ session('success') }}
            </div>
        @endif
        ```

**動作確認:**
- フォームを空のまま、またはタイトルを50文字以上で投稿しようとすると、エラーメッセージと共にフォーム画面に戻されることを確認します。
- その際、正常に入力した方の値はフォームに残っていることを確認します。
- 正常に投稿できた場合、一覧画面にリダイレクトされ、「投稿しました。」というメッセージが表示されることを確認します。
