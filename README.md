FORMAT: 1A
HOST: https://openapi.freshlive.tv

# OpenFRESH

OpenFRESHは生放送動画配信プラットフォーム [FRESH!](https://freshlive.tv) を便利に利用するための開発者向けプロジェクトです。

## 現在β公開中
現在、OpenFRESH APIはβ公開中であり、FRESH!の配信用アカウントを保持しているユーザーにのみ開放されています。
エンドポイントは `https://openapi.freshlive.tv` です。

また、β公開中はAPIのスキーマが変わることもありますのでご注意ください。


## Group 認証

OpenFRESH APIを利用するには、まずアクセストークンを取得する必要があり、アクセストークンの取得はFRESH!のアカウント（現在は配信主アカウントのみ）のログインアカウントを利用する必要がある。

## OAuth [/v1/oauth/token]

### アクセストークンの取得 [POST]
OpenFRESH APIを利用するためのアクセストークンを取得する。JSONレスポンスに含まれる`token`の内容がアクセストークンとなる。

+ Request (application/json)

    + Attributes

        + username: your_account (required)
        + password: your_password (required)

+ Response 200 (application/json)

    + Attributes
        + meta:
        + data:
            + token: `eyJhbGciOiJIUzI1NiJ9.eyJpZCI6IjE2IiwiZXhwIjoxNTE5OTc3Mjk1LCJzdGF0dXMiOiJicm9hZGNhc3QifQ.sTyTldvZ9H95ZsJfzMDUH2YgNGd3wYKV926tR90OjA2` (string, required) - アクセストークン
            + expiredAt: `2018-03-03T22:07:45.038+09:00` (string, required) - アクセストークンの有効期限


## Group アクセストークンの利用

### リクエストヘッダ

OpenFRESH APIへのリクエストは基本的にアクセストークンが必須となっている（アクセストークン取得APIを除いて）。`Authorization` リクエストヘッダを必ず付与するようにし、前述で取得したアクセストークンを以下のように設定する。

```bash
$ curl -X GET \
    -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJpZCI6IjE2IiwiZXhwIjoxNTE5OTc3Mjk1LCJzdGF0dXMiOiJicm9hZGNhc3QifQ.sTyTldvZ9H95ZsJfzMDUH2YgNGd3wYKV926tR90OjA2' \
    'https://openapi.freshlive.tv/v1/comments?programId=91633&limit=10&sinceMillisecond=0'
```


## Group コメント

## Comments [/v1/comments]

### コメントの取得 [GET]

番組のコメントを取得します。

+ Request
    + Headers
        Authorization: Bearer 取得したアクセストークン

+ Parameters

    + programId: 11111 (number, required) - 番組ID
    + limit: 50 (number) - 取得するコメントの件数
        + Default: 20
    + sinceMillisecond: 100000 (number) - コメントを取得する番組開始からの時間（ミリ秒）
        + Default: 0
    + order: asc (string) - ソート順
        + Default: asc
        + Members
            + asc
            + desc

+ Response 200 (application/json)

    + Attributes
        + meta:
            + nowMillisecond: 36892765 (number, optional) - 番組開始からの経過時間（ミリ秒） アーカイブの場合はnull
            + latestMillisecond: 29754707 (number, optional) - 取得したコメントデータのうち、最も最近なコメントのmillisecond
        + data:
            + id: `9eeff70f-6f0e-40b0-ae94-7ae34c38d2ae` (string, required) - コメントを識別する一意なID
            + type: user (string, required) - 投稿したユーザーの種別
            + freshId: 9999999 (string, required) - FRESH ID（ユーザーを識別するID）
            + programId: 11111 (string, required) - 番組ID
            + millisecond: 10000 (number, required) - 番組開始から対象のコメント時間までの時間（ミリ秒）
            + raw: こんにちはこんにちは @フレッシュ (string, required) - コメント本文
            + text: こんにちはこんにちは (string, required) - コメント本文から `@xxx` のようなメタトークンを取り除いたもの
            + atmark: (array, required) - コメントに `@xxx` のような文字列が含まれている場合、それを取り出した内容
                + フレッシュ
            + createdAt: `2017-03-09T06:16:45+09:00` (string, required) - コメントの投稿時間

+ Response 404 (application/json)

    + Body

{
  "error": {
    "status": 404,
    "message": "programId=100000 is not found.",
    "errorData": {
    }
  }
}

### コメントの投稿 [POST]

**配信中**の番組にコメントを投稿します。アーカイブになってしまった番組にコメントを投稿することは出来ません。

+ Request (application/json)

    + Headers
        Authorization: Bearer 取得したアクセストークン

    + Attributes

        + programId: 11111 (string,required) - コメントを投稿する番組ID
        + comment: こんにちはこんにちは (required) - コメント内容

+ Response 201 (application/json)

    + Body

        {}

+ Response 403 (application/json)

{
  "error": {
    "status": 403,
    "message": "this program is not onair.",
    "errorData": {
    }
  }
}


## Group 留意事項

### アクセストークンの扱い

アクセストークンは[JWT(Json Web Token)](https://jwt.io/)形式で発行され、一度発行されたアクセストークンは1年間有効です。JWTは改竄に強くセキュアですが、生成されたアクセストークンは第3者に不正利用されないように各々のユーザーで保管しておく必要があります。

### RateLimit

OpenFRESH APIにRateLimit（時間単位でのAPIコール数制限）が設定されています。現在は1ユーザーあたり**60回/1分**の制限を設けており、それを越えるAPIの利用をした場合は`HTTP 429 Too Many Requests`を返します。
RateLimitに達した場合は少し待ってからAPIリクエストをし、直近1分間のリクエスト数が60回を超えないような制御をする必要があります。



