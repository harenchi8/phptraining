
# 第6週 実践課題

**目的:**
第5週で作成したCRUD機能を持つメモアプリを、MVC（Model-View-Controller）の設計パターンに基づいてリファクタリング（構造改善）します。これにより、コードの可読性、保守性、再利用性を高めることの重要性を学びます。

---

### 事前準備: ディレクトリ構成の作成

まず、以下のようなディレクトリ構造を `week6` ディレクトリの中に作成してください。

```
/week6
|-- controllers/
|-- models/
|-- views/
|-- index.php       (フロントコントローラ兼ルーター)
|-- db_connect.php  (第5週のものをコピー)
```

`memos`テーブルと`php_training_db`データベースは第5週で作成したものをそのまま利用します。

---

## 課題: メモアプリのMVCへのリファクタリング

### Step 1: Modelの作成

**`models/Memo.php`** を作成し、メモに関するデータベース操作をすべてこのファイルに集約します。

**要件:**
- `Memo`というクラスを定義します。
- コンストラクタでPDOのデータベースハンドル（`$dbh`）を受け取り、プロパティに保持します。
- 以下のpublicメソッドを実装してください。内部ではプリペアドステートメントを使用します。
    - `findAll()`: 全てのメモをIDの降順で取得します。
    - `findById($id)`: 指定されたIDのメモを1件取得します。
    - `create($content)`: 新しいメモを作成します。
    - `update($id, $content)`: 指定されたIDのメモを更新します。
    - `delete($id)`: 指定されたIDのメモを削除します。

### Step 2: Viewの作成

ユーザーに表示されるHTML部分を `views` ディレクトリに分離します。

1.  **`views/index.php` (一覧・新規登録フォーム画面)**
    *   第5週の `index.php` のHTML部分をベースに作成します。
    *   PHPのロジックは、Controllerから渡される変数（例: `$memos`, `$errors`）を使って表示する `foreach` や `if` 程度に留めます。
    *   新規登録フォームの `action` は `index.php?action=create` のように、フロントコントローラに処理を依頼する形に変更します。
    *   編集・削除のリンクやフォームも同様に `index.php?action=edit&id=...` や `index.php?action=delete` のように変更します。

2.  **`views/edit.php` (編集画面)**
    *   第5週の `edit.php` のHTML部分をベースに作成します。
    *   Controllerから渡される変数（例: `$memo`）を使って、編集対象のメモをフォームに表示します。
    *   更新フォームの `action` は `index.php?action=update` のように変更します。

### Step 3: Controllerの作成

**`controllers/MemoController.php`** を作成し、ユーザーのリクエストとModel・Viewの橋渡し役を担わせます。

**要件:**
- `MemoController` というクラスを定義します。
- コンストラクタでDBハンドルを受け取ります。
- 以下のpublicメソッドを実装します。
    - `index()`: `Memo`モデルの`findAll()`を呼び出して全メモを取得し、`views/index.php` を `include` して表示します。
    - `create()`: POSTされた内容をバリデーションし、`Memo`モデルの`create()`を呼び出します。完了後、一覧ページにリダイレクトします。
    - `edit()`: GETで渡されたIDを元に `Memo`モデルの`findById()`を呼び出し、取得したメモデータと共に `views/edit.php` を `include` して表示します。
    - `update()`: POSTされた内容とIDをバリデーションし、`Memo`モデルの`update()`を呼び出します。完了後、一覧ページにリダイレクトします。
    - `delete()`: POSTされたIDを元に `Memo`モデルの`delete()`を呼び出します。完了後、一覧ページにリダイレクトします。

### Step 4: フロントコントローラ（ルーター）の作成

**`week6/index.php`** を作成し、すべてのリクエストをここで受け付け、適切なControllerのアクションに振り分けます。

**要件:**
- `db_connect.php` と `controllers/MemoController.php` を `require_once` します。
- `$_GET['action']` の値を見て、呼び出すべき `MemoController` のメソッドを決定します。
    - `action`が `create` なら `create()` メソッドを呼ぶ。
    - `action`が `edit` なら `edit()` メソッドを呼ぶ。
    - ...というように、`if`文や`switch`文で分岐させます。
    - `action`が指定されていない、または存在しない場合は、デフォルトとして `index()` メソッドを呼び出します。
- `MemoController` のインスタンスを生成し、対応するメソッドを実行します。

**動作確認:**
- `http://localhost/week6/index.php` にアクセスして、第5週と全く同じようにメモの登録、一覧表示、編集、削除ができることを確認します。
- 内部の構造は大きく変わりましたが、ユーザーから見た動作は同じであることがリファクタリングのポイントです。
- 各ファイルがそれぞれの責務（Model:DB操作, View:表示, Controller:橋渡し）に集中し、コードの見通しが良くなったことを実感してください。
