
# 第4週 座学テキスト

## 1. SQLとDB接続基礎 (9/13)

### データベース(DB)とSQLとは

- **データベース (Database)**: 大量のデータを整理・保存し、効率的にアクセスできるようにしたシステムです。代表的なものにMySQL, PostgreSQL, SQLiteなどがあります。
- **リレーショナルデータベース (RDB)**: データをExcelのような行と列からなる「テーブル」の形式で管理するデータベースです。テーブル同士を関連（リレーション）付けることができます。
- **SQL (Structured Query Language)**: データベースを操作するための言語です。データの取得、追加、更新、削除など、すべての操作はSQL文を使って行います。

### phpMyAdminによるDB操作

phpMyAdminは、Webブラウザを通じてMySQLデータベースを視覚的に管理できるツールです。XAMPPに同梱されています。

- **アクセス**: XAMPPコントロールパネルのMySQL行にある「Admin」ボタン、またはブラウザで `http://localhost/phpmyadmin` にアクセスします。
- **主な操作**:
    - **データベースの作成**: `データベース`タブから新しいデータベースを作成します。
    - **テーブルの作成**: 作成したデータベースを選択し、テーブル名とカラム（列）を定義します。カラムにはデータ型（`INT`=数値, `VARCHAR`=文字列, `TEXT`=長文, `DATETIME`=日時など）や制約（`PRIMARY KEY`=主キー, `AUTO_INCREMENT`=自動連番など）を設定します。
    - **データの操作**: `SQL`タブでSQL文を直接実行したり、`挿入`タブからデータを追加したりできます。

### 基本的なSQL文

SQLの基本は4つの操作「**CRUD**」です (Create, Read, Update, Delete)。

1.  **`SELECT` (Read: 読み取り)**: テーブルからデータを取得します。
    ```sql
    -- usersテーブルから全てのカラムの全データを取得
    SELECT * FROM users;

    -- usersテーブルからnameとemailカラムだけ取得
    SELECT name, email FROM users;

    -- idが1のユーザーを取得
    SELECT * FROM users WHERE id = 1;

    -- ageが30以上のユーザーを、idの降順（大きい順）で取得
    SELECT * FROM users WHERE age >= 30 ORDER BY id DESC;
    ```

2.  **`INSERT` (Create: 作成)**: テーブルに新しい行を追加します。
    ```sql
    INSERT INTO users (name, email, age) VALUES ('山田太郎', 'yamada@test.com', 30);
    ```

3.  **`UPDATE` (Update: 更新)**: 既存の行のデータを更新します。**`WHERE`句を忘れると全ての行が更新されてしまうので、絶対に忘れないこと！**
    ```sql
    UPDATE users SET age = 31, email = 'new-yamada@test.com' WHERE id = 1;
    ```

4.  **`DELETE` (Delete: 削除)**: 既存の行を削除します。**`WHERE`句を忘れると全ての行が削除されてしまうので、絶対に忘れないこと！**
    ```sql
    DELETE FROM users WHERE id = 1;
    ```

---

## 2. PDOとSQLインジェクション対策 (9/14)

### PHPからDBへの接続

PHPからデータベースに接続するには、**PDO (PHP Data Objects)** という拡張機能を利用するのが現在の標準です。PDOは、MySQL, PostgreSQLなど、さまざまな種類のデータベースに対して同じ関数でアクセスできるという利点があります。

### SQLインジェクションとは

最も危険な脆弱性の一つです。ユーザーの入力値をSQL文に直接組み込んでしまうことで、攻撃者が意図しないSQL文を実行できてしまう問題です。

**危険なコードの例:**
```php
<?php
// ユーザーが入力したIDを直接SQL文に埋め込んでいる
$id = $_POST['id'];
$sql = "SELECT * FROM users WHERE id = $id"; // ← 非常に危険！
?>
```
もし、攻撃者がIDとして `1 OR 1=1` という文字列を入力すると、SQL文は `SELECT * FROM users WHERE id = 1 OR 1=1` となり、`WHERE`句が常に真になるため、テーブルの全データが漏洩してしまいます。

### プリペアドステートメントによる対策

SQLインジェクションを防ぐには、**プリペアドステートメント**を利用します。

- **仕組み**: SQL文の「型（テンプレート）」と、後から埋め込む「値」を別々にデータベースに送信する方式です。値はあくまで「データ」として扱われ、SQL文の一部として解釈されることがないため、安全です。

### PDOを使ったDB操作

1.  **DB接続 (`new PDO`)**
    ```php
    $dsn = 'mysql:dbname=your_database;host=localhost;charset=utf8';
    $user = 'root';
    $password = ''; // XAMPPのデフォルトは空

    try {
        $dbh = new PDO($dsn, $user, $password);
        // エラー発生時に例外を投げるように設定
        $dbh->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    } catch (PDOException $e) {
        die("DB接続エラー: " . $e->getMessage());
    }
    ```

2.  **データ取得 (SELECT)**
    ```php
    // 1. SQL文を準備 (値の部分は「:プレースホルダ」にする)
    $sql = 'SELECT * FROM users WHERE id = :id';
    $stmt = $dbh->prepare($sql);

    // 2. 値をバインド
    $stmt->bindValue(':id', 1, PDO::PARAM_INT);

    // 3. 実行
    $stmt->execute();

    // 4. 結果を取得
    $user = $stmt->fetch(PDO::FETCH_ASSOC); // 1件取得 (連想配列形式)
    // $users = $stmt->fetchAll(PDO::FETCH_ASSOC); // 全件取得

    print_r($user);
    ```

3.  **データ登録 (INSERT)**
    ```php
    $sql = 'INSERT INTO users (name, email) VALUES (:name, :email)';
    $stmt = $dbh->prepare($sql);

    // 値をバインド
    $stmt->bindValue(':name', '鈴木花子', PDO::PARAM_STR);
    $stmt->bindValue(':email', 'suzuki@test.com', PDO::PARAM_STR);

    // 実行
    $stmt->execute();

    echo "登録が完了しました。";
    ```
`UPDATE`や`DELETE`も同様に、プリペアドステートメントを使って安全に実行します。
