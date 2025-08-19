
# 第10週 座学テキスト

## 1. CRUD実践① Resource Controller (10/25)

Webアプリケーション開発において、CRUD（作成、読み取り、更新、削除）操作は非常によくあるパターンです。Laravelでは、この定型的なCRUDルートとコントローラを効率的に作成するための仕組みとして **リソースコントローラ (Resource Controller)** が用意されています。

### RESTfulな設計思想

リソースコントローラは、**REST (Representational State Transfer)** という設計原則に基づいています。RESTでは、アプリケーション内のすべてのもの（投稿、ユーザー、商品など）を「リソース」として捉え、HTTPメソッド（`GET`, `POST`, `PUT/PATCH`, `DELETE`）を使ってリソースを操作します。

| URI | メソッド | アクション | ルート名 |
|:---|:---|:---|:---|
| `/posts` | `GET` | `index` | `posts.index` | 投稿の一覧を表示 |
| `/posts/create` | `GET` | `create` | `posts.create` | 投稿の作成フォームを表示 |
| `/posts` | `POST` | `store` | `posts.store` | 新しい投稿を保存 |
| `/posts/{post}` | `GET` | `show` | `posts.show` | 特定の投稿を表示 |
| `/posts/{post}/edit`| `GET` | `edit` | `posts.edit` | 特定の投稿の編集フォームを表示 |
| `/posts/{post}` | `PUT/PATCH`| `update` | `posts.update` | 特定の投稿を更新 |
| `/posts/{post}` | `DELETE` | `destroy` | `posts.destroy` | 特定の投稿を削除 |

この規約に従うことで、URLとメソッドを見るだけで、そのリクエストが何をするものなのかが明確になります。

### リソースコントローラの作成

Artisanコマンドで `--resource` (または `-r`) オプションを付けてコントローラを作成すると、上記の7つのアクションに対応するメソッドの雛形がすべて入ったコントローラが生成されます。

```bash
php artisan make:controller PhotoController --resource
```

### リソースルートの定義

`routes/web.php` に、たった1行追加するだけで、リソースコントローラの全アクションに対応する7つのルートをまとめて定義できます。

```php
// routes/web.php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class);
```
- `php artisan route:list` コマンドを実行すると、`photos.index`, `photos.store` といった名前のルートが自動で生成されているのが確認できます。

もし、7つのアクションのうち一部（例えば `show`）だけが不要な場合は、`except()` メソッドで除外できます。

```php
Route::resource('photos', PhotoController::class)->except(['show']);
```
逆に、特定のアクションだけを定義したい場合は `only()` メソッドを使います。

```php
Route::resource('photos', PhotoController::class)->only(['index', 'store']);
```

---

## 2. CRUD実践② 詳細・編集・削除 (10/26)

リソースコントローラのアクションを実装していきましょう。

### ルートモデルバインディング

`show`, `edit`, `update`, `destroy` の各メソッドでは、URLに含まれるID（例: `/posts/5` の `5`）を使って、対象のモデルインスタンスを取得する必要があります。

Laravelには **ルートモデルバインディング** という強力な機能があり、このプロセスを自動化してくれます。

**通常の書き方:**
```php
public function show($id)
{
    $post = Post::findOrFail($id); // IDで検索し、見つからなければ404エラー
    return view('posts.show', ['post' => $post]);
}
```

**ルートモデルバインディングを使った書き方:**
- コントローラのメソッドの引数で、IDの代わりにモデルクラスをタイプヒントします。変数名もルートセグメント（`{post}`）と一致させます。
```php
// ルート定義: Route::get('/posts/{post}', ...)

public function show(Post $post) // 引数を (Post $post) に変更
{
    // Laravelが自動的にURLのIDに一致するPostモデルを検索し、見つけて$postに注入してくれる。
    // 見つからなければ自動で404エラーを返す。
    // よって、自分で検索するコードは不要になる。
    return view('posts.show', ['post' => $post]);
}
```
これにより、コントローラのコードがよりシンプルでクリーンになります。`edit`, `update`, `destroy` メソッドでも同様に利用できます。

### HTMLフォームとHTTPメソッド

HTMLの`<form>`タグは、`GET`と`POST`メソッドしかネイティブでサポートしていません。しかし、RESTfulな設計では`PUT`, `PATCH`, `DELETE`メソッドも使います。

Laravelでは、`POST`メソッドのフォーム内に、`@method`ディレクティブを使って本来のHTTPメソッドを指定することで、この問題を解決します。

**編集フォーム (`edit.blade.php`) の例:**
```blade
<form action="{{ route('posts.update', $post) }}" method="POST">
    @csrf
    @method('PUT') {{-- ここでPUTメソッドを指定 --}}

    {{-- ...フォームの中身... --}}

    <button type="submit">更新</button>
</form>
```

**削除フォームの例:**
```blade
<form action="{{ route('posts.destroy', $post) }}" method="POST">
    @csrf
    @method('DELETE') {{-- ここでDELETEメソッドを指定 --}}

    <button type="submit">削除</button>
</form>
```

### 認可 (Authorization)

認証(Authentication)が「誰であるか」を確認するのに対し、認可(Authorization)は「その操作を実行する権限があるか」を確認することです。

例えば、「自分の投稿は編集・削除できるが、他人の投稿はできない」といった制御が認可にあたります。

Laravelでは、**ポリシー (Policy)** クラスを使って認可ロジックをカプセル化するのが一般的です。

**ポリシーの作成:**
```bash
php artisan make:policy PostPolicy --model=Post
```
- `app/Policies/PostPolicy.php` が作成されます。
- `--model=Post` を付けると、`Post`モデルに対する基本的なメソッドの雛形が生成されます。

**ポリシーの編集:**
```php
// app/Policies/PostPolicy.php
class PostPolicy extends Policy
{
    // 投稿を更新できるか
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }

    // 投稿を削除できるか
    public function delete(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
}
```

**コントローラでの利用:**
- `authorize` メソッドを使って、ポリシーのメソッドを呼び出します。
```php
// PostController.php
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post); // ここで認可チェック
    // 認可されなければ、自動的に403 (Forbidden) エラーが返される

    // ...更新処理...
}
```
