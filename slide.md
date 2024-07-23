class: center, middle

# 3章
# HTTP/1.0のセマンティクス:ブラウザの基本機能の裏側

---

# 目次

- フォームやファイルの送付
- ブラウザが期待する言語のコンテンツや画像フォーマットのファイルを取得
- 認証を使って、ユーザー固有のコンテンツを表示する仕組み
- クッキーを使い、アクセスのたびにログイン操作せずに済む仕組み
- プロキシを使った外部キャッシュやフィルタリングの導入
- リファラー
- 検索エンジン向けのアクセス制御
- ユーザーエージェント

---


# フォームの送信

## シンプルなフォーム
```
<form method="POST">
  <input name="title">
  <input name="author">
  <input type="submit">
</form>
```

`$ curl --httpl.0 -d title="The Art of Community" -d author="Jono Bacon" http://localhost: 18888`
---

## フォームを使ったファイルの送信
### マルチパートフォーム形式
```
<form method="POST" enctype="multipart/form-data">
  <input type="file" name="file">
  <input type="submit">
</form>
```
---

### 境界を指定して、ファイルのデータを送信
「ヘッダーフィールド＋空行＋コンテンツ」<br>

----WebKitFormBoundaryyOYfbccgoID172j7 <br>
Content-Disposition: form-data; name="title"<br><br>
The Art of Community<br>
-----WebKitFormBoundaryyOYfbccgoID172j7<br>
Content-Disposition: form-data; name="author"<br><br>
Jono Bacon<br>
------WebKitFormBoundaryyOYfbccgoID172j7--

---

<!-- ## フォームを利用したリダイレクト

```
<DOCTYPE html>
<html>
<body onload="document. forms [0].submit()">
  <form action=" リダイレクトしたい先" method="post">
    <input type="hidden" name="data" value="送りたいメッセージ" />
    <input type="submit" value="Continue"/>
  </form>
</body>
``` -->

---

# クッキー
サービス側がクライアント（ブラウザ）に保存を指示

名前（キー）=値の形式で保存
```
LAST_ACCESS_DATE=Jul/31/2016
LAST_ACCESS_TIME=12:04
```

---

- -c/--cookie-jarオプションで指定したファイルにサーバーから受け取ったクッキーを保存
- -b/--cookieオプションで指定したファイルから読み込んでクッキーを送信
  - -b "name=value"でクッキーを追加

`$ curl -http1.0 -c cookie.txt -b cookie.txt -b "name=value" http://example.com/helloworld`

---

<img src="images/5B572AE7-99AF-48B1-8A71-8AD14B758EE2_1_102_a.jpeg" alt="cookie" width="500px">

---

## 制約

| 属性       | 説明                                                                                                                                          | 省略時の動作                                                      |
|------------|---------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| **Expires、Max-Age** | **Expires**: `Med, 89 Jun 2821 10:18:14 GRT` 形式の日時で有効期限を設定<br>**Max-Age**: 現在時刻から指定秒数後に無効化 | 省略時: ブラウザを閉じた瞬間に消えるセッションクッキー           |
| **Domain** | クッキー送信先のサーバーを指定                                                                                                              | クッキーを発行したサーバー                                       |
| **Path**   | クッキー送信対象のサーバーのパスを指定                                                                                                       | クッキーを発行したサーバーのパス                                 |
| **Secure** | HTTPS接続時のみクッキーを送信<br>DNSハッキングによる不正送信リスクを軽減 | HTTP接続時: クッキーは送信されず流出を防止 |
| **HttpOnly** | JavaScriptからクッキーを隠す<br>セキュリティ向上: クロスサイトスクリプティングなどのリスク軽減 | なし                                                              |
| **SameSite** | 同じサイトのドメインに対してのみクッキーを送信<br>設定可能な値: `None`、`Lax`、`Strict` | なし                                                              |

---

## オリジン
同じオリジン
- http://example.com:80: HTTPはデフォルトで80番ポートを使うので同一
- http://example.com/news： パス違いはOK

すべて別のオリジン
- http://www.example.com:example.comと、www.example.comは別のドメイン
- https://example.com： スキームが違う
- http://example.com:8080：ポートが違う

---

## 認証
### Basic認証
- ユーザー名とパスワードを、base64エンコーディング(可逆変換)したもの
- SSL/TLS通信を使っていない状態で通信を傍受されると、通信内容から簡単にユーザー名とパスワードが漏洩
- base64（ユーザー名＋"；"+パスワード）

`$ curl --httpl.0 --basic -u user:pass http://localhost:18888`

- 次のようなフィールドが付与
`Authorization: "Basic dXNlcjpwYXNz"`

---

### Digest認証
- ハッシュ関数
- 401 Unauthorized というステータスでレスポンスが帰ってきます。以下のようなフィールドが付与
`ww-Authenticate: Digest realm="エリア名"，nonce="1234567890"， algorithm=SHA-256, qOp="auth"`
