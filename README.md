OpenAPI形式で記述しています。

DockerでモックサーバとSwagger UIを立ち上げることができます

# 起動方法
```
> cd swagger
> npm run resolve   # 初回起動時のみ必要
> docker-compose up -d
> docker-compose ps

NAMES               PORTS                            COMMAND                  NETWORKS
swagger-nginx       80/tcp, 0.0.0.0:8084->8084/tcp   "nginx -g 'daemon of…"   swagger_link
swagger-api         0.0.0.0:8083->8000/tcp           "/usr/local/bin/apis…"   swagger_link
swagger-ui          80/tcp, 0.0.0.0:8082->8080/tcp   "sh /usr/share/nginx…"   api-docs_default
swagger                                              "docker-entrypoint.s…"   api-docs_default
```

# API概要

## リクエスト

APIとの全ての通信にはHTTPSプロトコルを利用します。アクセス先のホストには、Connthassのデータを利用する場合には connthass.com を利用します
## パラメータ

Connthass APIへのリクエストには、GET、POST、PUT、PATCH、DELETEの5種類のHTTPメソッドを利用します。多くのAPIへのリクエストにはパラメータを含められますが、GETリクエストにパラメータを含める場合にはURIクエリを利用し、それ以外の場合にはリクエストボディを利用します。パラメータには、ページネーション用途など任意で渡すものと、投稿時の本文など必須のものが存在します。APIドキュメントには、各APIごとに送信可能なパラメータが記載されています。
## 利用制限

認証している状態ではユーザごとに1時間に1000回まで、認証していない状態ではIPアドレスごとに1時間に60回までリクエストを受け付けます。

## ステータスコード

200、201、204、400、401、403、404、500の8種類のステータスコードを利用します。GETまたはPATCHリクエストに対しては200を、POSTリクエストに対しては201を、PUTまたはDELETEリクエストに対しては204を返します。但し、エラーが起きた場合にはその他のステータスコードの中から適切なものを返します。
## データ形式

APIとのデータの送受信にはJSONを利用します。JSONをリクエストボディに含める場合、リクエストのContent-Typeヘッダにapplication/jsonを指定してください。但し、GETリクエストにバラメータを含める場合にはURIクエリを利用します。また、PUTリクエストまたはDELETEリクエストに対してはレスポンスボディが返却されません。日時を表現する場合には、ISO 8601形式の文字列を利用します。

GET /api/v1/items?page=1&per_page=20 HTTP/1.1

## エラーレスポンス

エラーの場合は、対応するHTTPコードと、エラーメッセージをJSONで返します。下記はAPIトークンが間違っていた場合のエラー例です。

HTTPレスポンスヘッダ

```
HTTP/1.1 401 Unauthorized
Content-Type: application/json
```

HTTPレスポンスボディ
```
{
  "errors": ["Invalid API token"]
}
```
エラーの場合はerrorsというキーに配列形式で、エラーメッセージが返却されます。

## ページネーション

一部の配列を返すAPIでは、全ての要素を一度に返すようにはなっておらず、代わりにページを指定できるようになっています。これらのAPIには、1から始まるページ番号を表すpageパラメータと、1ページあたりに含まれる要素数を表すper_pageパラメータを指定することができます。pageの初期値は1、pageの最大値は100に設定されています。また、per_pageの初期値は20、per_pageの最大値は100に設定されています。

ページを指定できるAPIでは、Linkヘッダ を含んだレスポンスを返します。Linkヘッダには、最初のページと最後のページへのリンクに加え、存在する場合には次のページと前のページへのリンクが含まれます。個々のリンクにはそれぞれ、first、last、next、prevという値を含んだrel属性が紐付けられます。

```
Link: <https://connthass.com/api/v1/events?page=1>; rel="first",
      <https://connthass.com/api/v1/events?page=1>; rel="prev",
      <https://connthass.com/api/v1/events?page=3>; rel="next",
      <https://connthass.com/api/v1/events?page=6>; rel="last"
```

また、ページを指定できるAPIでは、要素の合計数が Total-Count レスポンスヘッダに含まれます。

```
Total-Count: 6
```

## 認証認可

ConnthassのGETリクエスト以外のAPIを利用するには、アクセストークンをリクエストに含める必要があります。

アクセストークンは、以下のようにX-API-KEYをキーにしてリクエストヘッダに含められます。

```
X-API-KEY:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcd
```