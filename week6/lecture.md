
# 第6週 座学テキスト

## 1. アプリ構成とMVC導入 (9/27)

### なぜMVCが必要か？

第5週で作成したCRUDアプリでは、1つのPHPファイルにDB操作のロジックとHTMLの描画コードが混在していました。小規模なアプリならこれでも機能しますが、大規模になると以下のような問題が発生します。

- **可読性の低下**: ロジックと見た目が絡み合い、コードがどこで何をしているのか理解しにくくなる。
- **メンテナンス性の悪化**: デザインの変更がロジックに影響したり、その逆も然り。修正が困難になる。
- **再利用性の欠如**: コードが密結合しているため、特定の機能だけを別の場所で使い回すことが難しい。

これらの問題を解決するための設計パターンが **MVCモデル** です。

### MVCとは

MVCは、アプリケーションを以下の3つの役割に分割して構成する考え方です。

1.  **Model (モデル)**
    - **役割**: アプリケーションのデータと、それに関わる処理（ビジネスロジック）を担当。
    - **具体例**: データベースとのやり取り（CRUD操作）、データのバリデーションなど。
    - **原則**: データベースのテーブルと1対1で作成されることが多い（例: `memos`テーブル → `Memo`モデル）。

2.  **View (ビュー)**
    - **役割**: ユーザーに見える部分（UI）の生成を担当。
    - **具体例**: HTMLの生成、Modelから受け取ったデータの表示など。
    - **原則**: PHPのロジック（特にDB操作など）は含めず、データの表示に徹するべき。

3.  **Controller (コントローラ)**
    - **役割**: ModelとViewの橋渡し役。ユーザーからのリクエストを受け取り、それに応じてModelに処理を依頼し、結果をViewに渡して表示させる。
    - **具体例**: ユーザーのリクエスト（URL）に応じて、どのModelを呼び出し、どのViewを表示するかを決定する。

### MVCにおける処理の流れ

1.  ユーザーが特定のURLにアクセスする（リクエスト）。
2.  **ルーター**がURLを解釈し、対応する**Controller**のメソッドを呼び出す。
3.  **Controller**は、必要に応じて**Model**を呼び出し、データの取得や更新を依頼する。
4.  **Model**はデータベースとやり取りし、結果を**Controller**に返す。
5.  **Controller**は、Modelから受け取ったデータを**View**に渡す。
6.  **View**は、受け取ったデータを使ってHTMLを生成し、ユーザーに返す（レスポンス）。

### URLルーティング

MVCモデルでは、`index.php`や`edit.php`のような物理的なファイル名に直接アクセスするのではなく、`http://example.com/memos/edit/5` のような綺麗なURL（Pretty URL）を使います。

- **ルーター**: このURLを解析し、「`memos`というリソースのID `5` を編集する」という意味だと解釈して、適切なController（例: `MemoController`）の`edit`メソッドを呼び出す役割を担います。
- **フロントコントローラパターン**: 全てのリクエストを一旦単一のファイル（例: `index.php`）で受け取り、そこからルーターに処理を振り分ける設計が一般的です。

---

## 2. MVCアプリへの再構成 (9/28)

### View・Controller・Modelの分離

第5週のメモアプリをMVCの考え方に沿ってリファクタリング（内部構造の改善）してみましょう。

**ディレクトリ構成の例:**
```
/ (ルート)
|-- controllers/
|   |-- MemoController.php
|-- models/
|   |-- Memo.php
|-- views/
|   |-- index_view.php
|   |-- edit_view.php
|-- public/ (ドキュメントルート)
|   |-- index.php (フロントコントローラ)
|   |-- style.css
|-- db_connect.php
```
> `public`ディレクトリのみを外部に公開し、それ以外の重要なPHPファイル（DB接続情報やモデルなど）が直接アクセスされるのを防ぐ構成がセキュリティ上望ましいです。

### Modelの作成 (`models/Memo.php`)

- `memos`テーブルに関するDB操作（CRUD）をすべてこのファイルに集約します。
- `getAllMemos()`, `getMemoById($id)`, `createMemo($content)`, `updateMemo($id, $content)`, `deleteMemo($id)` のようなメソッドを持つクラスとして実装します。

```php
// models/Memo.php
class Memo {
    private $dbh;

    public function __construct($dbh) {
        $this->dbh = $dbh;
    }

    public function getAllMemos() {
        $stmt = $this->dbh->query("SELECT * FROM memos ORDER BY id DESC");
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    // ... 他のCRUDメソッド ...
}
```

### Controllerの作成 (`controllers/MemoController.php`)

- ユーザーのアクション（一覧表示、作成、編集、削除）に対応するメソッドを定義します。
- Modelを呼び出してデータを操作し、その結果をViewに渡します。

```php
// controllers/MemoController.php
require_once '../models/Memo.php';

class MemoController {
    // ...
    public function index() {
        $memoModel = new Memo($this->dbh);
        $memos = $memoModel->getAllMemos();
        // $memos をビューに渡して表示
        include '../views/index_view.php';
    }
    // ... 他のアクションに対応するメソッド ...
}
```

### Viewの作成 (`views/index_view.php`)

- 表示に特化したファイルです。PHPのロジックは最小限（`foreach`や`echo`）にします。
- Controllerから渡された変数を元にHTMLを組み立てます。

```php
<!-- views/index_view.php -->
<h1>メモ一覧</h1>
<ul>
    <?php foreach ($memos as $memo): // Controllerから渡された$memos変数 ?>
        <li>...</li>
    <?php endforeach; ?>
</ul>
```

### フロントコントローラと簡易ルーター (`public/index.php`)

- 全てのリクエストの入り口です。
- URLのパスを元に、どのControllerのどのメソッドを呼ぶかを決定する簡易的なルーティング処理を記述します。

このリファクタリングにより、各ファイルの責務が明確になり、コードの見通しが格段に良くなります。これは、Laravelのようなモダンなフレームワークが採用している構造の基礎となります。
