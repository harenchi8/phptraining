
# 第12週 実践課題

**目的:**
JavaScriptのFetch APIを使い、LaravelのAPIと連携して非同期でデータを登録・表示する機能を実装します。また、作成したアプリケーションを本番環境にデプロイするための準備を行います。

---

## 課題1: Ajax投稿機能（非同期登録）の実装 (12/8)

**目的:**
投稿詳細ページに、ページをリロードせずにコメントを投稿・表示できる機能をAjaxで実装します。

**手順:**

1.  **コメント機能のバックエンド準備**:
    *   **モデルとマイグレーション作成**: `Comment`モデルとマイグレーションを作成します。
        ```bash
        php artisan make:model Comment -m
        ```
    *   **マイグレーション編集**: `comments`テーブルには、`post_id` (どの投稿へのコメントか), `user_id` (誰がコメントしたか), `body` (コメント本文) のカラムが必要です。
    *   `php artisan migrate` を実行します。
    *   **モデル関連付け**: `Post`モデルと`hasMany`、`User`モデルと`hasMany`、`Comment`モデルと`belongsTo`の関連をそれぞれ定義します。
    *   **APIの作成**: コメントを投稿するためのAPIエンドポイント (`POST /api/posts/{post}/comments`) と、特定の投稿のコメント一覧を取得するAPIエンドポイント (`GET /api/posts/{post}/comments`) を `routes/api.php` に作成します。対応するコントローラ (`Api/CommentController`) とロジックも実装します。APIリソース (`CommentResource`) も作成して出力を整形しましょう。

2.  **フロントエンドの実装 (`posts/show.blade.php`)**:
    *   **コメント一覧表示エリア**: `<ul id="comment-list"></ul>` のような、JavaScriptでコメントを挿入するための空の要素を用意します。
    *   **コメント投稿フォーム**: `<form id="comment-form"> ... </form>` を作成します。このフォームは `action` や `method` 属性は不要です。
    *   **JavaScriptの実装**: `<script>` タグをビューの末尾に追加し、以下の処理を記述します。
        1.  **CSRFトークンの準備**: `meta`タグからCSRFトークンを取得しておきます。
        2.  **コメント一覧の初期表示**: ページが読み込まれたら、`fetch`でコメント一覧API (`/api/posts/{post}/comments`) にGETリクエストを送り、返ってきたJSONデータを使ってコメント一覧のHTMLを組み立て、`#comment-list` の中身を書き換えます。
        3.  **投稿フォームのsubmitイベント**: `comment-form` が送信された際のデフォルトの動作（ページリロード）を `event.preventDefault()` でキャンセルします。
        4.  `fetch`でコメント投稿API (`/api/posts/{post}/comments`) にPOSTリクエストを送信します。`body`にはフォームに入力された内容をJSON形式で含めます。`headers`に`Content-Type`と`X-CSRF-TOKEN`を指定するのを忘れないように。
        5.  投稿に成功したら、返ってきた新しいコメントのデータを使ってDOMを操作し、コメント一覧の先頭に追加します。フォームの中身は空にします。
        6.  エラー処理も `catch` を使って実装しましょう。

**動作確認:**
- 投稿詳細ページを開くと、既存のコメントが（API経由で）表示されることを確認します。
- コメントを投稿すると、ページがリロードされることなく、自分のコメントが一覧の先頭に即座に追加されることを確認します。
- ページをリロードしても、投稿したコメントが消えずに表示されることを確認します。

---

## 課題2: デプロイ準備 (12/9)

**目的:**
アプリケーションをRender.comなどのPaaSにデプロイするために必要な設定と手順を理解し、準備を整えます。

**手順:**

1.  **Gitリポジトリの準備**:
    *   `laravel-training` プロジェクトのルートで、Gitリポジトリを初期化します。
        ```bash
        git init
        git add .
        git commit -m "Initial commit"
        ```
    *   Laravelがデフォルトで用意している `.gitignore` ファイルにより、`node_modules` や `.env`、`storage` ディレクトリ内の一部のファイルなど、リポジトリに含めるべきでないファイルが適切に除外されていることを確認します。

2.  **GitHubリポジトリの作成とプッシュ**:
    *   GitHub上で、新しい**プライベート**リポジトリを作成します。（コードを公開したくない場合）
    *   ローカルリポジトリにリモートリポジトリのURLを登録し、`main`ブランチをプッシュします。
        ```bash
        git remote add origin <GitHubリポジトリのURL>
        git branch -M main
        git push -u origin main
        ```

3.  **`.env`ファイルの確認と `.env.example` の更新**:
    *   `.env` ファイルには、ローカル開発用の設定が記述されています。
    *   `.env.example` ファイルは、そのアプリケーションが必要とする環境変数のリストを示すための雛形ファイルです。`.env` に新しい変数を追加したら、`.env.example` にもキー名とダミーの値を追加する癖をつけましょう。
    *   本番環境で必要となる環境変数（`DB_HOST`, `DB_DATABASE`など）が `.env.example` にリストアップされていることを確認します。

4.  **Procfileの確認（任意・参考）**:
    *   Herokuなどの一部のPaaSでは、`Procfile` というファイルで起動コマンドを定義します。Renderではダッシュボードで設定できますが、ファイルとして定義することも可能です。
    *   プロジェクトのルートに `Procfile` を作成し、以下のように記述します。
        ```
        web: php artisan serve --host 0.0.0.0 --port $PORT
        ```

5.  **Renderでのデプロイシミュレーション（頭の中で）**:
    *   Render.comにサインアップし、GitHubと連携します。
    *   「New Web Service」から、作成したGitHubリポジトリを選択します。
    *   `lecture.md` を参考に、以下の項目を設定するイメージを掴みます。
        - **Build Command**: `composer install --no-dev && npm install && npm run build` (本番では開発用のパッケージは不要なので `--no-dev` を付けるのが一般的)
        - **Start Command**: `php artisan serve --host 0.0.0.0 --port $PORT`
        - **Environment Variables**: `.env` の内容をキーと値に分けて一つずつ設定していく。特に `APP_ENV=production`, `APP_DEBUG=false` に変更する。
    *   Render上でデータベースも作成し、その接続情報を環境変数に設定します。
    *   デプロイ後、コンソールから `php artisan migrate` を実行する必要があることを覚えておきます。

この課題は、実際にデプロイまで行わなくても、手順を理解し、GitHubにコードをプッシュするところまでをゴールとします。実際に無料枠でデプロイに挑戦してみるのも大変良い学習になります。
