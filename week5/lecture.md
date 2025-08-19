
# 第5週 座学テキスト

## 1. CRUDアプリ①：一覧・登録 (9/20)

CRUDは、ほとんどのアプリケーションの基本となるデータ操作、Create（作成）、Read（読み取り）、Update（更新）、Delete（削除）の頭文字を取ったものです。これを実装することで、アプリケーションの骨格が完成します。

### ファイル分割とコードの再利用

アプリケーションが複雑になるにつれて、1つのファイルに全てのコードを書くと、可読性やメンテナンス性が著しく低下します。機能ごとにファイルを分割し、共通の処理は部品化して再利用するのが基本です。

- **DB接続ファイル (`db_connect.php`)**: データベース接続処理を独立させ、必要な場所で `require_once` します。
- **関数ファイル (`functions.php`)**: よく使う処理（例えば `htmlspecialchars` のラッパー関数など）をまとめておき、これも `require_once` で読み込みます。
- **ヘッダー・フッター (`header.php`, `footer.php`)**: 全ページで共通のHTMLヘッダー（`<head>`タグやナビゲーション）やフッターを部品化し、`include` します。

**`require_once` vs `include`**
- `require_once`: ファイルが見つからない場合、致命的なエラー(Fatal Error)となり処理を停止します。DB接続など、それが無いと処理が続行不可能な場合に用います。
- `include`: ファイルが見つからなくても警告(Warning)を出すだけで、処理は続行します。HTMLの部品など、無くても致命的ではないが、あった方が良いものに使います。

### テンプレートの活用

PHPコードとHTMLコードが混在（スパゲッティコード）すると非常に見にくくなります。処理のロジックと見た目のテンプレートを分離する意識が重要です。

**基本的な分離の考え方:**
1.  PHPファイルの先頭で、データの取得やPOST処理などのロジックをすべて実行します。
2.  処理結果をPHP変数に格納します。
3.  ファイルの後半では、HTMLの構造に集中し、必要な場所で `<?= ... ?>` を使って変数の値を埋め込むだけにします。

```php
<?php
// --- ロジック部分 ---
require_once 'db_connect.php';
require_once 'functions.php';

// メモ一覧を取得
$memos = getAllMemos($dbh); // getAllMemosはDBから全件取得する自作関数

// --- テンプレート部分 ---
include 'header.php';
?>

<h1>メモ一覧</h1>
<ul>
    <?php foreach ($memos as $memo): ?>
        <li><?= h($memo['content']) ?></li>
    <?php endforeach; ?>
</ul>

<?php
include 'footer.php';
?>
```
> `<?php foreach(...): ?> ... <?php endforeach; ?>` のような代替構文を使うと、HTML内で制御構造が少し見やすくなります。

---

## 2. CRUDアプリ②：編集・削除 (9/21)

### `GET`パラメータを使った対象の指定

編集(Update)や削除(Delete)を行うには、「どのデータを対象にするか」をサーバーに伝える必要があります。一般的には、URLのクエリパラメータ（GETパラメータ）で対象のIDを渡します。

**例: 編集ページへのリンク**
```html
<a href="edit.php?id=5">編集</a>
```
このリンクをクリックすると、`edit.php` は `$_GET['id']` で `5` という値を受け取ることができます。このIDを使って、データベースから編集対象のデータを `SELECT` します。

### 編集(Update)処理の流れ

1.  **一覧ページ**: 各データに、IDを付与した編集ページへのリンク (`edit.php?id=...`) を設置します。
2.  **編集ページ (`edit.php`)**:
    a. GETでIDを受け取ります。IDがなければ不正なアクセスとして処理を中断します。
    b. 受け取ったIDを元に、DBから既存のデータを取得し、フォームの初期値として表示します。
    c. フォームには、更新対象のIDを `hidden` フィールドで含めておきます。
    d. フォームの送信先は `update.php` や、`edit.php` 自身に設定します。
3.  **更新処理ページ (`update.php`など)**:
    a. POSTで更新内容とIDを受け取ります。
    b. バリデーションを行います。
    c. `UPDATE`文を使い、指定されたIDのデータを更新します。
    d. 処理完了後、一覧ページにリダイレクトします。

### 削除(Delete)処理の流れ

削除はデータを消す破壊的な操作なので、ユーザーに本当に削除してよいか確認するステップを入れるのが親切です。

**方法A: 確認ページを挟む**
1.  一覧ページに削除リンク (`delete_confirm.php?id=...`) を設置。
2.  `delete_confirm.php` で「本当に削除しますか？」と尋ね、OKなら `delete.php` にIDをPOSTするフォームを置く。
3.  `delete.php` で `DELETE` 文を実行し、一覧にリダイレクト。

**方法B: JavaScriptで確認ダイアログを出す（より一般的）**
1.  一覧ページの削除ボタンに、JavaScriptの `confirm()` 関数を仕込みます。

```html
<form action="delete.php" method="post" onsubmit="return confirm('本当に削除しますか？');">
    <input type="hidden" name="id" value="<?= $memo['id'] ?>">
    <button type="submit">削除</button>
</form>
```
   - `onsubmit` イベントで `confirm` を呼び出し、`false` が返る（キャンセルが押される）とフォームの送信が中止されます。
2.  **削除処理ページ (`delete.php`)**:
    a. POSTでIDを受け取ります。
    b. `DELETE`文を使い、指定されたIDのデータを削除します。
    c. 処理完了後、一覧ページにリダイレクトします。
