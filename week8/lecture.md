
# 第8週 座学テキスト

## 1. Eloquent ORM (10/11)

### ORMとは

ORM (Object-Relational Mapping) は、データベースのテーブルと、プログラミング言語のオブジェクトを対応付け（マッピング）する技術です。ORMを使うと、SQL文を直接書かなくても、オブジェクトを操作するような直感的なコードでデータベースを扱うことができます。Laravelに搭載されているORMが **Eloquent (エロクエント)** です。

**Eloquentの利点:**
- **直感的**: SQLを意識せず、PHPのコードとして自然にDB操作を記述できる。
- **生産性向上**: `INSERT`や`UPDATE`などの定型的なSQL文を書く手間が省ける。
- **保守性向上**: データベースの種類（MySQL, PostgreSQLなど）が変わっても、PHP側のコードをほとんど変更する必要がない。

### マイグレーション (Migration)

マイグレーションは、**PHPのコードでデータベースのテーブル構造を管理する**仕組みです。SQLで直接テーブルを作成・変更する代わりに、マイグレーションファイルに変更履歴を記述していきます。

**利点:**
- チーム開発において、全員のDB構造を簡単に同期できる。
- テーブルの作成や変更の履歴がバージョン管理（Git）できる。

**マイグレーションファイルの作成:**
```bash
# postsテーブルを作成するためのマイグレーションファイルを作成
php artisan make:migration create_posts_table
```
- `database/migrations/` にファイルが作成されます。
- `up()` メソッドにテーブルを作成する処理、`down()` メソッドにテーブルを削除する処理を記述します。

**`up()`メソッドの例:**
```php
// database/migrations/xxxx_xx_xx_xxxxxx_create_posts_table.php
public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id(); // 自動増分するBIGINTの主キー (id)
        $table->string('title'); // VARCHAR
        $table->text('body'); // TEXT
        $table->timestamps(); // created_at と updated_at カラムを自動作成
    });
}
```

**マイグレーションの実行:**
- `.env`ファイルに正しいデータベース接続情報を設定した後、以下のコマンドを実行します。
```bash
php artisan migrate
```
- このコマンドで、まだ実行されていないマイグレーションファイルが実行され、DBにテーブルが作成されます。

### モデル (Model)

Eloquentでは、データベースの各テーブルに対応する「モデル」クラスを作成します。モデルを通じて、そのテーブルのデータを操作します。

**モデルの作成:**
```bash
# Postモデルを作成 (-m オプションで対応するマイグレーションファイルも同時に作成)
php artisan make:model Post -m
```
- `app/Models/Post.php` が作成されます。
- モデル名はテーブル名の単数形（`posts`テーブル → `Post`モデル）にするのが規約です。

### EloquentによるCRUD操作

`Post`モデルを使ってCRUD操作を行う例です。

```php
<?php
// PostController.php 内での操作例
use App\Models\Post; // Postモデルをインポート

// Read: 全件取得
$posts = Post::all();

// Read: IDで1件取得
$post = Post::find(1);

// Create: 新規作成
$post = new Post();
$post->title = '新しい投稿';
$post->body = '投稿の本文です。';
$post->save(); // DBに保存

// Update: 更新
$post = Post::find(1);
$post->title = '更新された投稿';
$post->save(); // DBに保存

// Delete: 削除
$post = Post::find(1);
$post->delete();

// Mass Assignment（マスアサインメント）による作成
// Postモデル側で$fillableプロパティの設定が必要
Post::create([
    'title' => 'マスアサインメントによる投稿',
    'body' => '本文です。'
]);
```

---

## 2. バリデーション制御 (10/12)

ユーザーからの入力値を検証するバリデーションは、アプリケーションの品質とセキュリティを保つために不可欠です。Laravelには強力なバリデーション機能が組み込まれています。

### Requestバリデーション

Laravelでのバリデーションは、主に **フォームリクエスト (Form Request)** を使う方法が推奨されます。バリデーションロジックをコントローラから分離できるため、コードがスッキリします。

**フォームリクエストの作成:**
```bash
php artisan make:request StorePostRequest
```
- `app/Http/Requests/StorePostRequest.php` が作成されます。

**フォームリクエストの編集:**
```php
<?php
// app/Http/Requests/StorePostRequest.php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    // このリクエストの認可設定。今回は誰でもOKなのでtrue。
    public function authorize()
    {
        return true;
    }

    // バリデーションルールを定義する。
    public function rules()
    {
        return [
            'title' => 'required|max:255', // 必須、かつ255文字以内
            'body' => 'required', // 必須
        ];
    }

    // (任意) バリデーションメッセージをカスタマイズ
    public function messages()
    {
        return [
            'title.required' => 'タイトルは必須です。',
            'body.required'  => '本文は必須です。',
        ];
    }
}
```

### コントローラでの利用

コントローラのメソッドで、引数の型を `Illuminate\Http\Request` から、作成したフォームリクエストクラス (`StorePostRequest`) に変更（タイプヒンティング）するだけで、自動的にバリデーションが実行されます。

```php
<?php
// PostController.php
use App\Http\Requests\StorePostRequest; // 作成したフォームリクエストをuse
use App\Models\Post;

class PostController extends Controller
{
    // ...

    // 引数の型を StorePostRequest に変更
    public function store(StorePostRequest $request)
    {
        // バリデーションが失敗した場合、この行より下は実行されず、
        // 自動的に直前のページにリダイレクトされ、エラーメッセージが表示される。

        // バリデーションを通過した場合のみ、以下の処理が実行される。
        Post::create($request->validated());

        return redirect()->route('posts.index')->with('success', '投稿が完了しました。');
    }
}
```

### Bladeでのエラーメッセージ表示

バリデーションエラーが発生すると、Laravelは自動的にエラー情報（`$errors`変数）をViewに渡します。これを使ってエラーメッセージを表示できます。

```blade
<!-- create.blade.php -->

{{-- $errorsに何かしらのエラーがあれば表示 --}}
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<form action="..." method="post">
    @csrf
    <div>
        <label>タイトル</label>
        <input type="text" name="title" value="{{ old('title') }}">
        {{-- 特定のフィールドのエラー表示 --}}
        @error('title')
            <div class="alert alert-danger">{{ $message }}</div>
        @enderror
    </div>
    ...
</form>
```
- `$errors->any()`: 何か1つでもエラーがあるかチェック。
- `$errors->all()`: 全てのエラーメッセージを配列で取得。
- `@error('field_name')`: 特定のフィールドのエラーがある場合のみ表示。
- `old('field_name')`: バリデーション失敗時に、直前に入力した値を再表示するヘルパー関数。
