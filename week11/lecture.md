
# 第11週 座学テキスト

## 1. ファイルアップロード (11/1)

Laravelでは、画像などのファイルアップロードを簡単かつ安全に処理する仕組みが提供されています。

### ファイルシステムの基本

Laravelのファイルシステムは、`config/filesystems.php` で設定します。デフォルトでいくつかの「ディスク」が定義されています。

- **`local`**: `storage/app/` ディレクトリを指します。外部から直接アクセスできないプライベートなファイルを保存するのに適しています。
- **`public`**: `storage/app/public/` ディレクトリを指します。プロフィール画像など、外部に公開したいファイルを保存するためのディスクです。
- **`s3`**: Amazon S3などのクラウドストレージサービス。設定すれば、ローカルファイルと同じように扱えます。

### 公開ディスク (`public`) の使い方

`storage/app/public` ディレクトリに保存したファイルをWebブラウザからアクセスできるようにするには、**シンボリックリンク**を作成する必要があります。以下のArtisanコマンドを一度だけ実行します。

```bash
php artisan storage:link
```

これにより、`public/storage` というシンボリックリンクが作成され、`storage/app/public` ディレクトリを指すようになります。つまり、`storage/app/public/images/avatar.jpg` というファイルには、`http://example.com/storage/images/avatar.jpg` というURLでアクセスできるようになります。

### ファイルアップロード処理の流れ

1.  **フォームの準備**
    - ファイルをアップロードするフォームには、必ず `enctype="multipart/form-data"` 属性を追加する必要があります。
    ```html
    <form action="..." method="POST" enctype="multipart/form-data">
        @csrf
        <input type="file" name="avatar">
        <button type="submit">アップロード</button>
    </form>
    ```

2.  **コントローラでの処理**
    - アップロードされたファイルは、`$request->file('input_name')` で `UploadedFile` オブジェクトとして取得できます。
    - `store()` メソッドを使うと、ファイルを指定したディスクに保存し、そのパスを返すことができます。

```php
<?php
// UserController.php

public function upload(Request $request)
{
    // バリデーション (画像ファイルであること、サイズ制限など)
    $request->validate([
        'avatar' => 'required|image|max:2048', // 必須, 画像形式, 2MBまで
    ]);

    if ($request->hasFile('avatar')) {
        // 'avatars' ディレクトリにファイルを保存し、そのパスを取得
        // (例: 'avatars/xxxxxxxx.jpg')
        $path = $request->file('avatar')->store('avatars', 'public');

        // 取得したパスをデータベースに保存する
        $user = $request->user();
        $user->avatar_path = $path;
        $user->save();
    }

    return back()->with('success', '画像をアップロードしました。');
}
```

3.  **ビューでの表示**
    - `asset()` ヘルパーとシンボリックリンクのパスを組み合わせて画像を表示します。
    ```blade
    <img src="{{ asset('storage/' . $user->avatar_path) }}" alt="アバター">
    ```

---

## 2. API開発入門 (11/2)

### APIとは

API (Application Programming Interface) は、あるソフトウェアが他のソフトウェアと情報をやり取りするための「窓口」や「接続仕様」のことです。Web APIの文脈では、主にサーバーが外部のプログラム（例: スマートフォンアプリ、JavaScriptで動作するフロントエンド）に対して、特定の形式（主にJSON）でデータを提供するための仕組みを指します。

### APIルーティング (`routes/api.php`)

Laravelでは、API用のルートは `routes/api.php` に記述します。ここに記述されたルートは、自動的にURLの先頭に `/api/` というプレフィックスが付きます。

- 例: `routes/api.php` に `Route::get('/posts', ...)` と書くと、実際のURLは `http://example.com/api/posts` となります。
- `api.php` のルートは、デフォルトで `api` ミドルウェアグループが適用され、セッションやCookieを使わないステートレスな認証（APIトークンなど）を前提としています。

### JSONレスポンス

APIでは、HTMLを返すのではなく、データをJSON形式で返すのが一般的です。Laravelのコントローラで配列やEloquentのコレクション（モデルの集合）をそのまま `return` すると、自動的にJSON形式に変換してレスポンスを返してくれます。

```php
<?php
// app/Http/Controllers/Api/PostController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    public function index()
    {
        $posts = Post::with('user')->latest()->get();

        // Eloquentコレクションを返すと、自動でJSONに変換される
        return $posts;
    }
}
```

### Eloquent APIリソース

EloquentモデルをJSONに変換する際、どのカラムを含めるか、カラム名を変更するか、リレーションを追加するかなど、出力形式を細かく制御したい場合があります。そのために **APIリソース** という変換層を利用します。

**APIリソースの作成:**
```bash
php artisan make:resource PostResource
```
- `app/Http/Resources/PostResource.php` が作成されます。

**APIリソースの編集:**
```php
<?php
// app/Http/Resources/PostResource.php

class PostResource extends JsonResource
{
    public function toArray($request)
    {
        // JSONとして返したいデータをここで定義する
        return [
            'id' => $this->id,
            'post_title' => $this->title, // 'title'を'post_title'にリネーム
            'content' => $this->body,
            'author_name' => $this->user->name, // リレーション先のデータも追加
            'created' => $this->created_at->format('Y-m-d'),
        ];
    }
}
```

**コントローラでの利用:**
- `PostResource` を使ってEloquentモデルをラップします。
- 1件の場合は `new PostResource($post)`、複数件の場合は `PostResource::collection($posts)` を使います。

```php
<?php
// app/Http/Controllers/Api/PostController.php
use App\Http\Resources\PostResource;

class PostController extends Controller
{
    public function index()
    {
        $posts = Post::with('user')->latest()->get();
        // コレクションをリソースでラップして返す
        return PostResource::collection($posts);
    }

    public function show(Post $post)
    {
        // 単一モデルをリソースでラップして返す
        return new PostResource($post);
    }
}
```
これにより、APIのレスポンス形式をモデルやコントローラのロジックから分離し、柔軟に管理することができます。
