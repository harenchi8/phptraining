
# 第10週 実践課題

**目的:**
これまでに作成した投稿機能を、リソースコントローラとリソースルートを使って、よりLaravelらしいRESTfulな構成にリファクタリングします。CRUDの全機能（詳細表示、編集、更新、削除）を実装し、ポリシーによる認可制御を追加してアプリケーションを完成させます。

---

## 課題1: リソースコントローラへのリファクタリング (10/25)

**目的:**
既存の`PostController`をリソースコントローラに置き換え、ルート定義をシンプルにします。

**手順:**

1.  **リソースコントローラの作成**:
    *   既存の `PostController` は一旦そのままで、練習として新しいリソースコントローラを作成します。（もし既存のものを置き換えたい場合は、先にバックアップを取ってください）
        ```bash
        php artisan make:controller PostResourceController --resource --model=Post
        ```
        > `--model=Post` を付けると、ルートモデルバインディングが適用された雛形が生成され便利です。

2.  **ルート定義の変更 (`routes/web.php`)**:
    *   これまで個別に定義していた投稿関連のルートをコメントアウトし、代わりに `Route::resource` を1行追加します。
        ```php
        use App\Http\Controllers\PostResourceController;

        // ...
        // Route::get('/posts', ...)->name('posts.index');
        // Route::get('/posts/create', ...)->name('posts.create');
        // ... これまでのルートをコメントアウト ...

        Route::resource('posts', PostResourceController::class)->middleware('auth')->except(['index', 'show']);
        ```
        > 認証が不要な `index` (一覧) と `show` (詳細) は `except` でミドルウェアの対象外にしています。

3.  **コントローラロジックの移行**:
    *   既存の `PostController` から、`index`, `create`, `store` メソッドの中身を、新しく作成した `PostResourceController` の対応するメソッドにコピーします。
    *   `PostResourceController` の `store` メソッドの引数は、`StorePostRequest` に変更するのを忘れないように。

4.  **ビューの `route()` ヘルパーを修正**:
    *   `posts` ディレクトリ内の各Bladeファイルで、`action` や `href` に指定している `route()` ヘルパーの記述が、リソースルートの名前に合っているか確認・修正します。
        - 例: `route('posts.store')`, `route('posts.create')` など。

**動作確認:**
- `php artisan route:list` を実行し、`posts.index`, `posts.store` など7つのルートが `PostResourceController` に紐付いていることを確認します。
- これまで通り、投稿の一覧表示と新規作成・保存ができることを確認します。

---

## 課題2: 詳細・編集・削除機能と認可の実装 (10/26)

**目的:**
CRUDの残り機能（Read, Update, Delete）を実装し、ポリシーを使って自分の投稿しか編集・削除できないように制御します。

**手順:**

1.  **詳細表示機能 (`show`) の実装**:
    *   `PostResourceController` の `show(Post $post)` メソッドを実装します。ルートモデルバインディングにより `$post` には既にデータが入っているので、それをビューに渡すだけです。
    *   `resources/views/posts/show.blade.php` を作成し、投稿のタイトルと本文、投稿者名などを表示します。
    *   一覧画面 (`index.blade.php`) の各投稿に、詳細ページへのリンク (`route('posts.show', $post)`) を追加します。

2.  **編集・更新機能 (`edit`, `update`) の実装**:
    *   `edit(Post $post)` メソッド: `show` と同様に、取得した `$post` を `posts.edit` ビューに渡します。
    *   `update(StorePostRequest $request, Post $post)` メソッド: バリデーション済みのデータで `$post` を更新し、詳細ページか一覧ページにリダイレクトします。
    *   `resources/views/posts/edit.blade.php` を作成し、編集フォームを設置します。`@method('PUT')` を忘れないように。
    *   詳細ページ (`show.blade.php`) に、編集ページへのリンクを追加します。

3.  **削除機能 (`destroy`) の実装**:
    *   `destroy(Post $post)` メソッド: `$post` を削除し、一覧ページにリダイレクトします。
    *   詳細ページ (`show.blade.php`) に、削除ボタンを設置します。`@method('DELETE')` とJavaScriptの確認ダイアログを忘れずに。

4.  **ポリシーの作成と登録**:
    *   Artisanコマンドで `PostPolicy` を作成します。
        ```bash
        php artisan make:policy PostPolicy --model=Post
        ```
    *   `app/Policies/PostPolicy.php` を開き、`update` と `delete` メソッドを実装します。ロジックは「ログインユーザーのIDと投稿の`user_id`が一致するか」です。
        ```php
        // PostPolicy.php
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
        // deleteも同様
        ```
    *   `app/Providers/AuthServiceProvider.php` を開き、`$policies` プロパティに `Post` モデルと `PostPolicy` を関連付ける記述を追加します。
        ```php
        protected $policies = [
            Post::class => PostPolicy::class,
        ];
        ```

5.  **コントローラでの認可処理**:
    *   `PostResourceController` の `edit`, `update`, `destroy` の各メソッドの冒頭で、`authorize` メソッドを呼び出します。
        ```php
        public function update(StorePostRequest $request, Post $post)
        {
            $this->authorize('update', $post);
            // ... 更新処理
        }
        // edit, destroyも同様
        ```

6.  **ビューでの表示制御**:
    *   詳細ページや一覧ページで、誰にでも編集・削除ボタンが見えるのは不親切です。`@can` ディレクティブを使って、権限のあるユーザーにしか表示されないようにします。
        ```blade
        {{-- show.blade.php --}}
        @can('update', $post)
            <a href="{{ route('posts.edit', $post) }}">編集</a>
        @endcan

        @can('delete', $post)
            <form ...>@csrf @method('DELETE') ... </form>
        @endcan
        ```

**動作確認:**
- 自分で作成した投稿の詳細ページでは「編集」「削除」が表示され、他人が作成した投稿のページでは表示されないことを確認します。
- 他人の投稿の編集・削除URLに直接アクセスしようとすると、「403 This action is unauthorized.」エラーが表示されることを確認します。
- 編集・更新・削除機能が正しく動作することを確認します。
