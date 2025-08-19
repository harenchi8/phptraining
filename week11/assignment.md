
# 第11週 実践課題

**目的:**
Laravelのファイルストレージ機能を使ってプロフィール画像のアップロード機能を実装します。また、`routes/api.php` とAPIリソースを使って、外部アプリケーション向けのJSON形式のAPIを作成します。

---

## 課題1: プロフィール画像アップロード機能の作成 (11/1)

**目的:**
ユーザーが自分のプロフィール画像をアップロードし、それがページに表示される機能を実装します。

**手順:**

1.  **シンボリックリンクの作成**:
    *   まだ実行していない場合は、ターミナルで以下のArtisanコマンドを実行します。
        ```bash
        php artisan storage:link
        ```

2.  **`users`テーブルへのカラム追加**:
    *   ユーザーの画像ファイルパスを保存する `avatar_path` カラムを `users` テーブルに追加します。
        ```bash
        php artisan make:migration add_avatar_path_to_users_table --table=users
        ```
    *   作成されたマイグレーションファイルの `up` メソッドを編集します。
        ```php
        Schema::table('users', function (Blueprint $table) {
            $table->string('avatar_path')->nullable()->after('email');
        });
        ```
    *   `php artisan migrate` を実行します。

3.  **ルートの定義 (`routes/web.php`)**:
    *   プロフィール編集ページと、画像アップロード処理のルートを追加します。これらはログインユーザーのみが対象なので `auth` ミドルウェアグループの中に追加します。
        ```php
        use App\Http\Controllers\ProfileController;

        // Breezeが生成したルートの下あたりに追加
        Route::get('/profile/edit', [ProfileController::class, 'edit'])->name('profile.edit');
        Route::post('/profile/upload', [ProfileController::class, 'upload'])->name('profile.upload');
        ```

4.  **コントローラの作成と実装**:
    *   `php artisan make:controller ProfileController` でコントローラを作成します。
    *   `edit` メソッド: ログインユーザーの情報をビューに渡します。
    *   `upload` メソッド: `lecture.md` を参考に、画像のバリデーションとアップロード処理を実装します。保存先は `public` ディスクの `avatars` ディレクトリとします。保存後、ファイルパスを `users` テーブルの `avatar_path` カラムに保存します。

5.  **ビューの作成 (`resources/views/profile/edit.blade.php`)**:
    *   Breezeのダッシュボード (`dashboard.blade.php`) などを参考に、レイアウトを整えます。
    *   現在のプロフィール画像（もしあれば）を表示します。なければデフォルト画像を表示するなどの工夫をしましょう。
        ```blade
        @if(Auth::user()->avatar_path)
            <img src="{{ asset('storage/' . Auth::user()->avatar_path) }}" ...>
        @else
            <img src="/default_avatar.png" ...> {{-- public/にデフォルト画像を置く --}}
        @endif
        ```
    *   画像を選択するための `<input type="file">` を持つフォームを作成します。`enctype="multipart/form-data"` を忘れないように。

**動作確認:**
- プロフィール編集ページにアクセスし、フォームが表示されることを確認します。
- 画像ファイルを選択してアップロードし、「アップロードしました」といった成功メッセージと共にリダイレクトされ、ページ上の画像が更新されることを確認します。
- `storage/app/public/avatars` ディレクトリに画像ファイルが保存されていること、`public/storage` にシンボリックリンクが作成されていることを確認します。
- `users` テーブルの `avatar_path` カラムに、`avatars/xxxxxxxx.jpg` のようなパスが保存されていることを確認します。

---

## 課題2: 投稿一覧APIの作成 (11/2)

**目的:**
`routes/api.php` を使い、投稿の一覧と詳細をJSONで返すAPIエンドポイントを作成します。APIリソースを使って、出力するJSONのフォーマットを整形します。

**手順:**

1.  **API用コントローラの作成**:
    *   API用のコントローラは専用のディレクトリに置くと管理しやすいため、`Api` ディレクトリを指定して作成します。
        ```bash
        php artisan make:controller Api\PostController
        ```

2.  **APIルートの定義 (`routes/api.php`)**:
    *   投稿一覧 (`/posts`) と投稿詳細 (`/posts/{post}`) を取得するAPIルートを定義します。
        ```php
        use App\Http\Controllers\Api\PostController;

        Route::get('/posts', [PostController::class, 'index']);
        Route::get('/posts/{post}', [PostController::class, 'show']);
        ```

3.  **APIリソースの作成**:
    *   `Post` モデルのデータを整形するための `PostResource` を作成します。
        ```bash
        php artisan make:resource PostResource
        ```
    *   `app/Http/Resources/PostResource.php` を開き、`toArray` メソッドを編集して、APIで返したい形式の配列を返すようにします。（`lecture.md`参照）
        - `id`, `title`, `body`, `user_name`, `created_at` などを返すようにしてみましょう。

4.  **コントローラの編集 (`Api/PostController.php`)**:
    *   `index` メソッド: 全ての投稿を取得し、`PostResource::collection()` でラップして返します。
    *   `show` メソッド: ルートモデルバインディングで受け取った `$post` を `new PostResource($post)` でラップして返します。

**動作確認:**

- **PostmanなどのAPIクライアントツール、またはWebブラウザで**、以下のURLにアクセスします。
    - **一覧API**: `http://127.0.0.1:8000/api/posts`
    - **詳細API**: `http://127.0.0.1:8000/api/posts/1` (存在する投稿IDを指定)
- ブラウザに、整形されたJSONデータが表示されることを確認します。
- JSONのキーが、`PostResource` で定義した通りの名前になっていることを確認します。
- `data` というキーの中に、オブジェクトまたはオブジェクトの配列が格納されていることを確認します。（LaravelのAPIリソースのデフォルトの形式です）
