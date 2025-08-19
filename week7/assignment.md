
# 第7週 実践課題

**目的:**
Composerを使ってLaravelプロジェクトを新規作成し、Artisanコマンド、ルーティング、コントローラ、Bladeビューの基本的な使い方を学び、Laravel開発の第一歩を踏み出します。

---

## 課題1: Laravelプロジェクトの作成とHello World (10/4)

**目的:**
Laravelのインストールと、最も基本的なルーティング・ビューの表示を体験します。

**手順:**

1.  **Composerのインストール確認**:
    *   ターミナル（コマンドプロンプト）で `composer -V` を実行し、バージョン情報が表示されることを確認します。エラーが出る場合は、先にComposerをインストールしてください。

2.  **Laravelプロジェクトの作成**:
    *   `PWORKS`ディレクトリ（あるいは任意の作業ディレクトリ）に移動し、以下のコマンドを実行して `laravel-training` という名前のプロジェクトを作成します。
        ```bash
        composer create-project laravel/laravel laravel-training
        ```
    *   インストールには数分かかります。

3.  **プロジェクトの起動**:
    *   作成されたプロジェクトディレクトリに移動します。
        ```bash
        cd laravel-training
        ```
    *   Artisanコマンドで開発サーバーを起動します。
        ```bash
        php artisan serve
        ```

4.  **動作確認**:
    *   ブラウザで `http://127.0.0.1:8000` または `http://localhost:8000` にアクセスし、Laravelのウェルカムページが表示されることを確認します。

5.  **ルーティングの変更 (Hello World)**:
    *   `routes/web.php` ファイルを開きます。
    *   既存の `/` のルートをコメントアウトするか、以下のように書き換えて、「Hello World」という文字列が返るように変更します。
        ```php
        use Illuminate\Support\Facades\Route;

        Route::get('/', function () {
            return 'Hello World from Laravel!';
        });
        ```

6.  **再度の動作確認**:
    *   ブラウザをリロードし、画面が「Hello World from Laravel!」に変わっていることを確認します。

---

## 課題2: ControllerとBladeを活用したお知らせ一覧の仮表示 (10/5)

**目的:**
Artisanコマンドでコントローラを作成し、ルートからコントローラを呼び出し、コントローラから渡されたデータをBladeビューで表示するという、Laravelの基本的なMVCの流れを実装します。

**手順:**

1.  **コントローラの作成**:
    *   ターミナルで（`php artisan serve`は動かしたままでOK、別のターミナルを開いて）以下のArtisanコマンドを実行し、`NewsController` を作成します。
        ```bash
        php artisan make:controller NewsController
        ```
    *   `app/Http/Controllers/NewsController.php` が作成されたことを確認します。

2.  **コントローラの編集**:
    *   `NewsController.php` を開き、`index` というpublicメソッドを追加します。
    *   `index` メソッド内では、お知らせのダミーデータを配列で作成し、`news.index` というビューにそのデータを渡して返すように記述します。
        ```php
        <?php

        namespace App\Http\Controllers;

        use Illuminate\Http\Request;

        class NewsController extends Controller
        {
            public function index()
            {
                // ダミーのお知らせデータ
                $news_list = [
                    ['id' => 1, 'title' => 'Laravelの学習を開始しました', 'published_at' => '2025-10-05'],
                    ['id' => 2, 'title' => '週末課題のお知らせ', 'published_at' => '2025-10-04'],
                    ['id' => 3, 'title' => 'システムメンテナンスのお知らせ', 'published_at' => '2025-10-01'],
                ];

                return view('news.index', ['news_list' => $news_list]);
            }
        }
        ```

3.  **ルーティングの設定**:
    *   `routes/web.php` を開き、`/news` というURLへのGETリクエストを `NewsController` の `index` メソッドに紐付けるルート定義を追加します。
        ```php
        use App\Http\Controllers\NewsController; // use文を追加

        // ... 他のルート ...

        Route::get('/news', [NewsController::class, 'index']);
        ```

4.  **Bladeビューの作成**:
    *   `resources/views/` ディレクトリの中に、`news` という新しいディレクトリを作成します。
    *   `resources/views/news/` の中に `index.blade.php` というファイルを作成します。
    *   `index.blade.php` を編集し、コントローラから渡された `$news_list` を `@foreach` でループして、お知らせ一覧をHTMLのテーブルで表示するように記述します。
        ```blade
        <!DOCTYPE html>
        <html lang="ja">
        <head>
            <meta charset="UTF-8">
            <title>お知らせ一覧</title>
        </head>
        <body>
            <h1>お知らせ一覧</h1>
            <table border="1">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>タイトル</th>
                        <th>公開日</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach ($news_list as $news)
                        <tr>
                            <td>{{ $news['id'] }}</td>
                            <td>{{ $news['title'] }}</td>
                            <td>{{ $news['published_at'] }}</td>
                        </tr>
                    @endforeach
                </tbody>
            </table>
        </body>
        </html>
        ```

5.  **動作確認**:
    *   ブラウザで `http://127.0.0.1:8000/news` にアクセスします。
    *   コントローラで定義したダミーデータが、Bladeビューで作成したテーブルに正しく表示されていることを確認します。
