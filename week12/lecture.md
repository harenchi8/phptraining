
# 第12週 座学テキスト

## 1. JSと非同期連携 (12/8)

これまでのWebアプリケーションは、フォームを送信するたびにページ全体がリロードされる同期的なものでした。現代的なUI/UXでは、ページをリロードせずにサーバーと通信し、画面の一部だけを更新する**非同期通信 (Ajax)** が広く使われています。

### Ajaxとは

Ajax (Asynchronous JavaScript + XML) は、JavaScriptを使ってサーバーと非同期にデータをやり取りする技術の総称です。（現在ではデータの形式としてXMLよりJSONが主流です。）

**Ajaxの利点:**
- **UXの向上**: ページ遷移なしでデータを更新できるため、ユーザーはスムーズで高速な操作感を体験できる。
- **サーバー負荷の軽減**: HTML全体を再生成する必要がなく、必要なデータ（JSON）だけを送受信するため、通信量が削減される。

### Fetch API

Fetch APIは、ブラウザに組み込まれている、非同期通信を行うための標準的なインターフェースです。Promiseベースで設計されており、直感的でモダンな書き方ができます。

**GETリクエストの例:**
```javascript
fetch('/api/posts')
    .then(response => {
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        return response.json(); // レスポンスをJSONとして解析
    })
    .then(data => {
        console.log(data); // 取得したデータを使ってDOM操作などを行う
    })
    .catch(error => {
        console.error('Fetch error:', error);
    });
```

**POSTリクエストの例 (CSRF対策込み):**
LaravelのAPIと連携する場合、特に認証が必要なエンドポイントではCSRFトークンの送信が必要です。

```javascript
// BladeのmetaタグにCSRFトークンを埋め込んでおく
// <meta name="csrf-token" content="{{ csrf_token() }}">

const csrfToken = document.querySelector('meta[name="csrf-token"]').getAttribute('content');

const postData = {
    title: '新しい投稿',
    body: '本文'
};

fetch('/api/posts', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRF-TOKEN': csrfToken // CSRFトークンをヘッダーに含める
    },
    body: JSON.stringify(postData) // 送信するデータをJSON文字列に変換
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Error:', error));
```

### SPAとの違い

- **マルチページアプリケーション (MPA)**: これまで作ってきたLaravelアプリのように、画面遷移のたびにサーバーから新しいHTMLを取得する伝統的なWebアプリ。
- **シングルページアプリケーション (SPA)**: 最初に単一のHTMLページを読み込み、その後はJavaScriptがAPIと非同期通信を行い、DOMを動的に書き換えることで画面遷移を実現するWebアプリ。ReactやVue.jsなどのフレームワークを使って構築されることが多い。

今回行うAjaxは、MPAに部分的に非同期処理を取り入れてUI/UXを向上させる、という位置づけです。

---

## 2. デプロイ準備 (12/9)

デプロイとは、開発環境（自分のPC）で作成したアプリケーションを、インターネット上の誰もがアクセスできる本番環境（公開サーバー）に配置し、公開するプロセスです。

### `.env`ファイルと環境変数

`.env`ファイルには、データベースのパスワードなど、環境ごとに異なる設定や、公開してはいけない機密情報を記述します。このファイルはGitリポジトリに含めてはいけません。

本番環境では、サーバー側で環境変数を設定する機能（例: HerokuのConfig Vars, RenderのEnvironment Variables）を使い、`.env`ファイルと同じ内容をセットします。Laravelは、環境変数がセットされていれば、`.env`ファイルよりもそちらを優先して読み込みます。

**本番環境で特に重要な設定:**
- **`APP_ENV`**: `production` に設定します。これにより、エラーメッセージが詳細に表示されなくなり、パフォーマンスが最適化されます。
- **`APP_DEBUG`**: `false` に設定します。`true`のままだと、エラー発生時にデバッグ情報が外部に漏洩し、非常に危険です。
- **`APP_KEY`**: `php artisan key:generate` で生成したキーを設定します。暗号化などに使われるため、開発環境と本番環境では異なるキーを使うのが望ましいです。
- **`DB_*`**: 本番環境用のデータベース接続情報を設定します。
- **`APP_URL`**: アプリケーションの公開URLを設定します。

### デプロイ先の選択肢

近年では、PaaS (Platform as a Service) と呼ばれるクラウドサービスを利用するのが主流です。サーバーのOSやミドルウェアの管理をサービス側が行ってくれるため、開発者はアプリケーションのコードをデプロイすることに集中できます。

- **Render**: Gitリポジトリと連携し、`git push` をトリガーに自動でビルドとデプロイを行ってくれるサービス。小規模なアプリなら無料枠で運用可能。データベースサービスも提供している。
- **Heroku**: PaaSの草分け的存在。Renderと同様にGit連携での自動デプロイが可能。
- **Vercel / Netlify**: 主にフロントエンド（SPA）のホスティングに強いサービスですが、サーバーレス関数を使えばPHPも動かせます。

### 一般的なデプロイフロー (Renderの例)

1.  **コードをGitHubにプッシュ**: LaravelプロジェクトをGitHubのプライベートリポジトリにプッシュします。
2.  **Renderに登録**: Renderのアカウントを作成し、GitHubアカウントと連携します。
3.  **Web Serviceの作成**: Renderのダッシュボードから「New Web Service」を選択し、デプロイしたいGitHubリポジトリを選択します。
4.  **設定**:
    - **Name**: サービス名（URLの一部になる）
    - **Environment**: `PHP`
    - **Build Command**: `composer install && npm install && npm run build`
    - **Start Command**: `php artisan serve --host 0.0.0.0 --port $PORT`
5.  **環境変数の設定**: `.env`ファイルの内容をRenderの「Environment」タブにコピー＆ペーストします。`APP_DEBUG=false` など、本番用の値に書き換えるのを忘れないように。
6.  **データベースの作成**: RenderでPostgreSQLなどのデータベースサービスを別途作成し、その接続情報を環境変数に設定します。
7.  **デプロイ**: 設定を保存すると、自動で初回デプロイが開始されます。
8.  **マイグレーションの実行**: デプロイ完了後、Renderのコンソール（Shell）機能を使って `php artisan migrate` を実行し、本番DBにテーブルを作成します。
