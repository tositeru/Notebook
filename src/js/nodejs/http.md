# Nodejs HTTP

Node.jsにおけるHTTPサーバーの使い方についてメモしていく

Node.jsベースのHTTPサーバーのフレームワークの裏側ではこのようなコードがあることを覚えておく。

[Anatomy of an HTTP Transaction](https://nodejs.org/en/docs/guides/anatomy-of-an-http-transaction/)

## 基本

サーバーの作成にはNode.jsの`http`モジュールを使う。

`request`と`response`がHTTPリクエストとレスポンスに対応している。

- **request**
  - [http.IncomingMessage](https://nodejs.org/api/http.html#http_class_http_incomingmessage)
  - [stream.Readable](https://nodejs.org/api/stream.html#stream_class_stream_readable)
- **response**
  - [http.ServerResponse](https://nodejs.org/api/http.html#http_class_http_serverresponse)
  - [stream.Writable](https://nodejs.org/api/stream.html#stream_class_stream_writable)

```js
const http = require('http')

const server = http.createServer((request, response) => {
  //リクエストの基本情報を取得
  const {method, url, headers} = request

  //リクエストの本体を解析する
  //on('data')とon('end')を組み合わせるのが定番のパターンになる
  let body = [];
  request.on('error', err => {
    console.error(err)
  }).on('data', chunk => {
    body.push(chunk)
  }).on('end', () => {
    body = Buffer.concat(body).toString()

    //レスポンスはここで行うの定番っぽい
    response.on('error', err => {
      console.error(err)
    })

    //リクエストのヘッダーを設定
    response.writeHead(200, {
      'Content-Type': 'application/json',
      'X-Powered-By': 'bacon'
    })
    // 以下のように分割しても設定できる
    response.status(200)
    response.setHeader('Content-Type', 'application/json');
    response.setHeader('X-Powered-By': 'bacon');

    //リクエスト本体を書き込む
    response.write('{Apple: 100, Orange: 200, Grope: 300}')
    response.end()
  })
})

//サーバーをHTTPリクエスト待ちにする
server.listen(port, host)
```

## 例) Echoサーバー

参考ページはEchoサーバーの例を複数出してエラー処理について説明していた。

`error`イベントにコールバックを設定することでエラー処理をカスタマイズできる。

以下、最終的なコード例

```js
const http = require('http');

http.createServer((request, response) => {
  //エラーが起きたときのイベント設定
  request.on('error', (err) => {
    console.error(err);
    response.statusCode = 400;
    response.end();
  });
  response.on('error', (err) => {
    console.error(err);
  });

  //処理本体
  if (request.method === 'POST' && request.url === '/echo') {
    //エラー処理とは無関係だが
    // requestとresponseはそれぞれnode.jsのReadableStreamとWritableStreamから派生しているのでこうゆうこともできる
    request.pipe(response);
  } else {
    response.statusCode = 404;
    response.end();
  }
}).listen(8080);
```
