
# 第14週 実践課題

**目的:**
先週の設計と準備に基づき、アプリケーションのコアとなる機能の実装を本格的に進めます。Gitのブランチ戦略を実践し、計画的に開発を進める経験を積みます。

---

## 課題: メイン機能の実装 (11/22 - 11/23)

この2日間で、先週自分で定義した「**Must-have（必ず実装するコア機能）**」をすべて完成させることを目標とします。

**実装する機能の例:**
- ユーザー認証（Breezeで実装済み）
- メインとなるリソースのCRUD機能（例: ブログアプリなら記事のCRUD、タスク管理アプリならタスクのCRUD）
- CRUD機能に付随する認可処理（自分の投稿しか編集・削除できないなど）
- ファイルアップロード機能（もしコア機能に含めている場合）

**開発の進め方:**

1.  **タスク管理ツールの準備**:
    *   もし未使用なら、TrelloやGitHub Projectsなどのカンバンツールに、今週実装するタスクをカードとして登録しましょう。「機能要件リスト」を元に、「DB設計」「ルーティング」「コントローラ作成」「ビュー作成」「バリデーション実装」のように、できるだけ細かく分割するのがコツです。

2.  **Gitブランチの作成**:
    *   まず、開発のベースとなる `develop` ブランチを作成します。
        ```bash
        # mainブランチからdevelopブランチを作成して移動
        git checkout -b develop
        ```
    *   実装する機能ごとに、`develop` ブランチからフィーチャーブランチを切ります。
        ```bash
        # 例: 記事のCRUD機能を実装するためのブランチを作成
        git checkout -b feature/post-crud
        ```

3.  **機能ごとの実装サイクル**:
    *   `feature/post-crud` ブランチで、記事のCRUD機能の実装に集中します。
    *   **コントローラ → ルート → ビュー** の順、またはその逆など、自分がやりやすい流れで実装を進めてください。
    *   `php artisan make:controller PostController --resource --model=Post` や `php artisan make:policy PostPolicy --model=Post` など、Artisanコマンドを積極的に活用して効率化を図りましょう。
    *   こまめにコミットする習慣をつけましょう。1つのコミットは1つの意味のある変更単位にすると、後から見返したときに分かりやすくなります。
        ```bash
        # 良いコミットメッセージの例
        git commit -m "feat: Add post index page"
        git commit -m "fix: Correct validation rule for post title"
        ```
        > `feat:`は新機能、`fix:`はバグ修正、`refactor:`はリファクタリングなど、接頭辞を付けるとより明確になります。

4.  **`develop`ブランチへのマージ**:
    *   一つの機能（例: 記事CRUD）が完成し、手動での動作確認が終わったら、`develop` ブランチにマージします。
        ```bash
        # developブランチに移動
        git checkout develop

        # developブランチを最新の状態にする（今回は不要だが習慣として）
        git pull origin develop

        # フィーチャーブランチをマージ
        git merge feature/post-crud

        # GitHubにプッシュ
        git push origin develop
        ```
    *   マージ後、不要になったフィーチャーブランチは削除しても構いません (`git branch -d feature/post-crud`)。

5.  **次の機能へ**:
    *   `develop` ブランチから、次の機能のフィーチャーブランチ（例: `feature/comment-function`）を新たに切り、同じサイクルを繰り返します。

**デバッグのヒント:**
- 「Undefined variable」エラーが出たら、コントローラからビューに正しく変数を渡しているか確認しましょう。
- 「404 Not Found」エラーが出たら、`routes/web.php` の定義が正しいか、`action`や`href`のURLが合っているか確認しましょう。`php artisan route:list` も役立ちます。
- 「MassAssignmentException」が出たら、モデルの `$fillable` プロパティに、`create()` や `update()` で使いたいカラム名がすべて含まれているか確認しましょう。
- 意図した動作にならないときは、`dd()` を使って、怪しい箇所の直前の変数の状態を確認するのが最も効果的です。

この課題は、2日間で多くの機能を実装する必要があり、最も大変なパートです。エラーが頻発すると思いますが、エラーメッセージを読み解き、`dd()` を駆使して一つずつ解決していくプロセスそのものが、エンジニアとしての重要なスキルアップに繋がります。焦らず、着実に進めていきましょう。
