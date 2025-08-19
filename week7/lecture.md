
# 第7週 座学テキスト

## 1. Laravel導入① (10/4)

### Laravelとは

Laravelは、現在世界で最も人気のあるPHPのWebアプリケーションフレームワークです。MVCアーキテクチャをベースにしており、Webアプリケーション開発で頻繁に必要となる機能（ルーティング、DB操作、認証、テンプレートエンジンなど）が最初から豊富に用意されています。これにより、開発者は面倒な定型作業から解放され、アプリケーション固有のロジック開発に集中できます。

**主な特徴:**
- **MVCアーキテクチャ**: 第6週で学んだMVCモデルを基本構造として採用。
- **Eloquent ORM**: 直感的なコードでデータベースを操作できるオブジェクトリレーショナルマッパー。
- **Artisan CLI**: コマンドラインからコードの自動生成やDB管理ができる強力なツール。
- **Bladeテンプレートエンジン**: PHPコードをHTML内に綺麗に記述できる、シンプルで高機能なテンプレートエンジン。
- **豊富なエコシステム**: 認証機能(Breeze/Jetstream)、決済(Cashier)、管理画面(Nova)など、公式・サードパーティ製のパッケージが多数存在する。

### Composerによるプロジェクト作成

Laravelは、PHPのパッケージ管理ツールである **Composer** を使ってインストールします。ComposerがPCにインストールされていない場合は、[公式サイト](https://getcomposer.org/)から先にインストールしておいてください。

**Laravelプロジェクトの新規作成コマンド:**
```bash
composer create-project laravel/laravel aplication-name
```
- `aplication-name` の部分に、作成したいアプリケーションのディレクトリ名を指定します。
- このコマンドを実行すると、指定したディレクトリにLaravelプロジェクトの雛形がすべてダウンロード・設置されます。

### Artisan CLI (アーティザン)

Artisanは、Laravelに付属するコマンドラインインターフェースです。プロジェクトのルートディレクトリで `php artisan` コマンドを実行することで、様々なタスクを自動化できます。

- **開発サーバーの起動**: `php artisan serve`
    - これを実行すると、`http://127.0.0.1:8000` で開発用のWebサーバーが起動します。
- **コマンド一覧の表示**: `php artisan list`
- **コントローラやモデルの雛形作成**: `php artisan make:controller`, `php artisan make:model` など（後述）。

### Laravelのディレクトリ構造

`composer create-project` で作成されたディレクトリの中身は以下のようになっています。特に重要なものを抜粋します。

- **`app/`**: アプリケーションの心臓部。ControllerやModelはここに配置されます。
- **`config/`**: データベース接続情報やアプリケーションの設定ファイル群。
- **`database/`**: マイグレーションファイル（テーブル定義）やシーダー（初期データ）を配置。
- **`public/`**: Webサーバーのドキュメントルート。`index.php`（フロントコントローラ）やCSS/JSファイルが配置される。
- **`resources/views/`**: Bladeテンプレートファイル（View）を配置。
- **`routes/`**: URLのルーティング定義ファイル。
    - **`web.php`**: 通常のWebブラウザからのアクセスに対するルートを定義します。
- **`.env`**: 環境変数ファイル。DB接続情報など、環境ごとに異なる設定を記述します。このファイルはGitなどのバージョン管理に含めてはいけません。（`.env.example`が雛形として提供されます）

---

## 2. Laravel導入② (10/5)

### ルーティング (`routes/web.php`)

Laravelでは、URLとそれに対応する処理（主にControllerのメソッド）の関連付けを `routes/web.php` ファイルに定義します。

**基本的なルーティング:**

- **クロージャ（無名関数）を使う方法**
    ```php
    // routes/web.php
    use Illuminate\Support\Facades\Route;

    // GETリクエストで / (ルート) にアクセスがあった場合、クロージャ内の処理を実行
    Route::get('/', function () {
        return 'Hello World';
    });

    // /greeting にアクセスがあった場合、ビューファイル welcome.blade.php を表示
    Route::get('/greeting', function () {
        return view('welcome');
    });
    ```

- **コントローラを使う方法（こちらが基本）**
    ```php
    // routes/web.php
    use App\Http\Controllers\PostController;

    // /posts にアクセスがあった場合、PostControllerのindexメソッドを呼び出す
    Route::get('/posts', [PostController::class, 'index']);
    ```

### Bladeテンプレートエンジン (`resources/views/`)

Bladeは、Laravelのシンプルで強力なテンプレートエンジンです。通常のPHPファイルと異なり、`.blade.php` という拡張子を持ちます。

**主な構文:**

- **変数の表示**: `{{ $variable }}`
    - 自動的に `htmlspecialchars` が適用され、XSS対策が施されます。
- **制御構文**:
    - `@if`, `@elseif`, `@else`, `@endif`
    - `@foreach`, `@endforeach`
    - `@for`, `@endfor`
    - `@isset`, `@endisset` / `@empty`, `@endempty`
- **レイアウトの継承**:
    - `@extends('layouts.app')`: `layouts/app.blade.php` という親レイアウトを継承します。
    - `@section('content') ... @endsection`: 親レイアウトの `@yield('content')` の部分にコンテンツを埋め込みます。
    - これにより、全ページ共通のヘッダーやフッターを効率的に管理できます。

### コントローラ (`app/Http/Controllers/`)

コントローラは、ルーティングとビジネスロジック（Model）を繋ぎ、最終的にViewを返す役割を担います。

**コントローラの作成コマンド:**
```bash
php artisan make:controller PostController
```
このコマンドで、`app/Http/Controllers/PostController.php` が自動生成されます。

**コントローラの例:**
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class PostController extends Controller
{
    // 投稿一覧を表示するメソッド
    public function index()
    {
        // (本来はここでModelからデータを取得する)
        $posts = [
            ['title' => 'Laravel入門'],
            ['title' => 'PHPの基礎']
        ];

        // view()ヘルパー関数でビューを返す
        // 'posts.index' は resources/views/posts/index.blade.php を指す
        // 第2引数でビューにデータを渡す
        return view('posts.index', ['posts' => $posts]);
    }
}
```

**対応するView (`resources/views/posts/index.blade.php`):**
```blade
<h1>投稿一覧</h1>
<ul>
    @foreach ($posts as $post)
        <li>{{ $post['title'] }}</li>
    @endforeach
</ul>
```
