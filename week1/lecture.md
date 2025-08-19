[カリキュラム一覧へ](../index.md) | [この週の課題へ](./assignment.md)
***
# 第1週 座学テキスト

## 1. PHPとJSの違い・環境構築 (8/23)

### PHPとは？
PHP (Hypertext Preprocessor) は、サーバーサイドで動作するスクリプト言語です。特にWeb開発に特化しており、HTMLに直接コードを埋め込んで動的なWebページを生成することを得意とします。

- **サーバーサイド言語**: ユーザーのブラウザ（クライアントサイド）で動くJavaScriptとは異なり、PHPはWebサーバー上で処理され、その結果（HTMLなど）だけがブラウザに返されます。
- **HTML埋め込み**: `<?php ... ?>` タグを使って、HTMLコードの中にPHPのロジックを記述できます。

---

### JavaScriptとの主な違い

| 項目 | PHP | JavaScript (ブラウザ) |
|:---|:---|:---|
| **実行環境** | Webサーバー | Webブラウザ |
| **主な用途** | サーバーでのデータ処理、DB連携、HTML生成 | HTMLの操作、ユーザーとの対話、非同期通信 |
| **変数宣言** | `$`で始まる (例: `$name`) | `let`, `const`, `var` (例: `let name`) |
| **文字列結合** | `.` (ドット) | `+` (プラス) |
| **連想配列** | `['key' => 'value']` | `{key: 'value'}` (オブジェクト) |
| **処理モデル** | 同期処理が基本 | 非同期処理が得意 (イベントループ) |

---

### PHPの基本構文

```php
<?php // PHPコードの開始

// 1. コメント
// これは一行コメントです
/*
  これは
  複数行コメントです
*/

// 2. 変数とデータ型
$name = "PHP"; // 文字列
$version = 8; // 数値
$isEasy = true; // 真偽値
$features = ["Web", "DB連携", "フレームワーク"]; // 配列

// 3. 配列 (インデックス配列と連想配列)
// インデックス配列
$colors = ["Red", "Green", "Blue"];
echo $colors[0]; // "Red"

// 連想配列
$user = [
  "name" => "山田",
  "age" => 30
];
echo $user["name"]; // "山田"

// 4. 制御構文 (if, for, foreach)
if ($version >= 8) {
  echo "PHP $version はモダンです。";
}

for ($i = 0; $i < count($colors); $i++) {
  echo $colors[$i];
}

foreach ($user as $key => $value) {
  echo "$key は $value です。";
}

// 5. 関数
function greet($name) {
  return "こんにちは、$name さん！";
}
echo greet("鈴木");

?>
```

---

### 実行環境の構築 (XAMPP)

XAMPP（ザンプ）は、Webサーバー(Apache)、データベース(MariaDB)、PHP、Perlをまとめてインストールできる開発環境パッケージです。

1.  **インストール**: [公式サイト](https://www.apachefriends.org/jp/index.html)からお使いのOSに合ったインストーラをダウンロードし、指示に従ってインストールします。
2.  **起動**: XAMPPコントロールパネルを起動し、「Apache」の「Start」ボタンを押します。
3.  **動作確認**:
    *   XAMPPのインストールフォルダ内にある `htdocs` ディレクトリが、Webサーバーのルートディレクトリになります。
    *   `htdocs` 内に `index.php` というファイルを作成し、以下の内容を記述します。
        ```php
        <?php
          phpinfo();
        ?>
        ```
    *   ブラウザで `http://localhost/index.php` にアクセスし、PHPの情報画面が表示されれば環境構築は成功です。

---

## 2. フォーム処理とHTML連携 (8/24)

### スーパーグローバル変数

スーパーグローバル変数は、PHPスクリプトのどこからでもアクセスできる、あらかじめ定義された特別な変数です。

- **`$_GET`**: URLのクエリ文字列（`?`以降）で渡されたデータを格納する連想配列。
- **`$_POST`**: HTTPのPOSTメソッドで送信されたデータを格納する連想配列。フォームの値を受け取る際に最もよく使います。
- **`$_SERVER`**: サーバーや実行環境の情報を持つ配列。
    - `$_SERVER['REQUEST_METHOD']`: 現在のページにアクセスされた際のメソッド (`GET`, `POST`など)。
    - `$_SERVER['REQUEST_URI']`: ページにアクセスするために指定されたURI。
- **`$_SESSION`**: セッション変数を格納する配列。ログイン状態の保持などに使います（第2週で詳述）。
- **`$_COOKIE`**: HTTPクッキーの値を格納する配列（第2週で詳述）。

> **補足**: `$_REQUEST` は `$_GET`, `$_POST`, `$_COOKIE` の内容をまとめたものですが、リクエストの送信元が不明確になるため、通常は `$_GET` や `$_POST` を明示的に使うことが推奨されます。

---

### HTMLとの連携

PHPの最大の強みは、HTMLとシームレスに連携できることです。

**1. HTML内にPHP変数を埋め込む**

`echo` を使ってHTMLタグや変数の値を出力します。

```php
<?php
  $title = "マイページ";
  $user_name = "田中";
?>
<!DOCTYPE html>
<html>
<head>
  <title><?php echo $title; ?></title>
</head>
<body>
  <h1>ようこそ、<?php echo $user_name; ?>さん</h1>
</body>
</html>
```

**短縮構文 `<?=`**

`<?php echo $variable; ?>` は `<?= $variable ?>` と省略して書くことができます。こちらの方がコードがスッキリします。

```html
<h1>ようこそ、<?= $user_name ?>さん</h1>
```

**2. フォーム (`<form>`) との連携**

HTMLの`<form>`タグは、ユーザーの入力をサーバーに送信するための重要な要素です。

- **`action`属性**: フォームのデータを送信する先のPHPファイル名を指定します。
- **`method`属性**: HTTPメソッドを指定します (`GET`または`POST`)。
- **`input`タグの`name`属性**: 送信されたデータを受け取る際のキー（連想配列のキー）になります。

**例: `login.html`**
```html
<form action="login.php" method="POST">
  <label>ユーザーID: <input type="text" name="user_id"></label>
  <label>パスワード: <input type="password" name="password"></label>
  <button type="submit">ログイン</button>
</form>
```

**例: `login.php`**
```php
<?php
  // POSTで送信されたデータを取得
  $userId = $_POST['user_id'];
  $password = $_POST['password'];

  echo "ユーザーID: $userId";
  // ここでDB照合などのログイン処理を行う
?>
```