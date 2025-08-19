
# 第14週 座学テキスト

## 1. 最終課題③：機能実装① (11/22)

今週から、先週設計したアプリケーションの本格的な実装フェーズに入ります。計画的に、そして効率的に開発を進めるための手法を学びます。

### 実装計画の立て方

いきなり目についたところから作り始めるのではなく、機能の依存関係を考えて、実装する順番を計画することが重要です。

1.  **機能の洗い出しと優先順位付け**:
    - 先週作成した機能要件リストを再確認します。
    - 機能の重要度や、他の機能のベースとなるもの（例: ユーザー認証は多くの機能の前提となる）から優先順位を付けます。
    - この手法を **MoSCoW（モスクワ）分析** (Must-have, Should-have, Could-have, Won't-have) を使って整理するのも有効です。
        - **Must**: 必ず実装するコア機能 (例: CRUD, 認証)
        - **Should**: ぜひ実装したい主要機能 (例: 検索, いいね)
        - **Could**: あれば嬉しい追加機能 (例: タグ付け, ランキング)
        - **Won't**: 今回は実装しない機能

2.  **タスクの分割**:
    - 「記事投稿機能を実装する」という大きなタスクを、より具体的な小さなタスクに分割します。
    - **例: 記事投稿機能のタスク分割**
        1.  `posts`テーブルのマイグレーション作成
        2.  `Post`モデル作成
        3.  `PostController`作成 (リソースコントローラ)
        4.  `web.php`にリソースルートを追加
        5.  投稿作成画面(`create.blade.php`)のHTML作成
        6.  `store`メソッドのバリデーションと保存処理を実装
        7.  ...というように、1つあたり数時間で終わるレベルまで細分化します。

3.  **TrelloやGitHub Projectsを使ったタスク管理**:
    - 分割したタスクをカンバン方式のツールで管理すると、進捗が可視化できてモチベーション維持に繋がります。
    - 「ToDo」「Doing」「Done」のレーンを作り、タスクカードを移動させていきます。

### Gitによるブランチ戦略

チーム開発はもちろん、個人開発でもGitのブランチを効果的に使うことで、安全かつ効率的に開発を進められます。**Git-flow** を参考にしたシンプルなブランチ戦略がおすすめです。

- **`main` (または `master`) ブランチ**: 常に安定して動作する、本番環境のコードを管理するブランチ。直接コミットはしません。
- **`develop` ブランチ**: 開発のメインとなるブランチ。`main`から最初に分岐させます。開発中の最新のコードがここに集約されます。
- **フィーチャーブランチ (`feature/xxx`)**: 新しい機能を実装するためのブランチ。「記事投稿機能」や「ログイン機能」など、機能ごとに`develop`ブランチから分岐させます。
    - 例: `feature/post-crud`, `feature/user-authentication`
    - 機能が完成したら、`develop`ブランチにプルリクエスト（またはマージ）します。
    - マージが完了したら、フィーチャーブランチは削除します。

**開発フローの例:**
1.  `git checkout develop`
2.  `git pull origin develop` (最新の状態に更新)
3.  `git checkout -b feature/new-feature` (新機能用のブランチを作成して移動)
4.  (コーディング & コミット)
5.  `git checkout develop`
6.  `git merge feature/new-feature` (開発ブランチにマージ)
7.  `git push origin develop`

---

## 2. 最終課題④：機能実装② (11/23)

### デバッグ手法

開発にエラーはつきものです。エラーの原因を効率的に特定し、修正する「デバッグ」のスキルは非常に重要です。

- **エラーメッセージをよく読む**: Laravelのエラー画面（Ignition）は非常に高機能です。エラーメッセージ、発生場所、スタックトレース（どの関数呼び出しを辿ってエラーに至ったか）を注意深く読み解くだけで、原因のほとんどが特定できます。

- **`dd()` ヘルパー**: `die and dump` の略。引数に渡した変数の内容を整形して表示し、そこで処理を停止します。特定の時点での変数の状態を確認したい場合に非常に強力です。
    ```php
    public function store(Request $request)
    {
        dd($request->all()); // フォームから送られてきた全データを表示して停止
        // ...
    }
    ```

- **Logファサード**: 処理を止めずに変数の内容などをログファイル (`storage/logs/laravel.log`) に記録したい場合に使います。
    ```php
    use Illuminate\Support\Facades\Log;

    Log::info('ここにメッセージ', ['context' => $variable]);
    ```

- **Laravel Telescope**: 開発時にアプリケーションのリクエスト、例外、DBクエリ、ログなどをWeb UIで監視できる、公式のデバッグツールです。`composer require laravel/telescope --dev`, `php artisan telescope:install`, `php artisan migrate` で簡単に導入できます。

### テストの重要性

手動での動作確認（ブラウザをポチポチする）は重要ですが、機能が増えるたびに全てのパターンを確認するのは非効率で、確認漏れも発生します。

**自動テスト**は、コードでコードの動作を検証する仕組みです。一度テストコードを書いておけば、コマンド一つで何度でも同じ検証を正確に実行できます。

- **ユニットテスト (Unit Test)**: 関数やメソッドなど、ごく小さな単位（ユニット）が正しく動作するかを検証するテスト。
- **フィーチャーテスト (Feature Test)**: 複数のユニットを組み合わせた機能（例: APIエンドポイントへのリクエストからレスポンスまで）が、一連の流れとして正しく動作するかを検証するテスト。

Laravelでは、**PHPUnit** というテストフレームワークが標準で組み込まれています。

**フィーチャーテストの例:**
```bash
php artisan make:test PostCreationTest
```

```php
<?php
// tests/Feature/PostCreationTest.php
namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;
use App\Models\User;

class PostCreationTest extends TestCase
{
    use RefreshDatabase; // テスト実行ごとにDBをリフレッシュする

    public function test_authenticated_user_can_create_post()
    {
        $user = User::factory()->create(); // テスト用のユーザーを作成

        // このユーザーとしてログインした状態で /posts にPOSTリクエストを送る
        $response = $this->actingAs($user)->post('/posts', [
            'title' => 'テスト投稿',
            'body' => 'これはテストです。'
        ]);

        // postsテーブルにデータが保存されたかアサート（表明）
        $this->assertDatabaseHas('posts', [
            'title' => 'テスト投稿'
        ]);

        // /posts にリダイレクトされたかアサート
        $response->assertRedirect('/posts');
    }
}
```

`./vendor/bin/phpunit` コマンドでテストを実行できます。最終課題でテストコードを書くのは挑戦的な目標ですが、テストの概念を知っておくことは非常に重要です。
