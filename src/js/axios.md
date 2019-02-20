# axios

ブラウザとNode.jsでHttpリクエストをラップしてくれているモジュール。
将来的には[Fetch API](https://developer.mozilla.org/ja/docs/Web/API/Fetch_API)に置き換わるかもしれないが、これを書いている時点でまだ完全にサポートされていないのでAxiosが好んで使われている。

[axios](https://github.com/axios/axios)

## 使い方

すごくわかりやすい。

Promise/Asyncに対応している。

```js
const axios = require('axios')

// Promise
axios.get('example.com')
  .then(response => {
    // 成功した時
  })
  .catch(error => {
    // 失敗した時
  })
  .then(() => {
    // 成功/失敗にかかわらず最後に実行するコード
  })

//Async/Await
async function getData() {
  try {
    const reponse = await axios.get('example.com');
    console.log(reponse)
  } catch (error) {
    console.error(error)
  }
}

// パラメータ付き
axios.get('example.com/user?id=123')
axios.get('example.com/user', {param: {id: 123}})

// POSTリクエスト
axios.post('example.com/user', {userName: 'XXX', password: 'YYY'})

```

## リクエストの設定

```js

//axiosでは以下の関数で全てのHttpリクエストを実行できる。
//　axios.get()などはこれの省略形
// configについては下を参照
axios(config)

//　axios.defaultに値を設定すると、グローバル設定となり全てのリクエストに反映される。
let config = {
  //  以下の設定でURLは https://example.com/user?id=123　になる。
  method: 'get',
  url: 'user',
  baseURL: 'https://example.com',
  params: {
    param: {id: 123}
  },
  //以下、色々な設定

  // ヘッダーの追加設定
  headers: {'X-Requested-With': 'XMLHttpRequest'},
  
  // リクエストの本体部分
  data: { hoge: 'foo'},

  // タイムアウト(ミリ秒)
  timeout: 1000,

  //レスポンスの設定
  responseType: 'json',
  responseEncoding: 'utf8',

  // CSRF対策で使われるもの
  xsrfCookieName: 'XSRF-TOKEN',
  xsrfHeaderName: 'X-XSRF-TOKEN',

  // リクエストを送る前にparamsをURLのクエリ文字列に変換する処理をカスタマイズする時に使う。
  paramsSerializer: function(params) {},

  //認証関係
  // オリジン間リソース共有 (CORS)で使われるパラメータ
  withCredentials: false, // <- default

  //HTTP Basic auth
  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

  //アップロード/ダウンロード実行中のイベント
  onUploadProgress: function(progressEvent) {}
  onDownLoadProgress: function(progressEvent) {}

  //Httpリクエストの内容のデータサイズの上限(byte単位)
  maxContentLength: 2000,

  // リクエストのキャンセルする時に使うキャンセルトークンの設定
  cancelToken: new CancelToken(function (cancel) {
  })

  //リクエストのレスポンスステートの検証
  validateStatus: function(status) {
    return status >= 200 && status < 300;
  },

  // その他
  maxRedirects: 5, // default
  socketPath: null, // default
  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),
  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },
}

```

## レスポンスの内容

Httpリクエストに成功した時のresponseの値は以下の要素を持つ

- data       : レスポンスデータ。ここにリクエストの結果が入っている。
- status     : HTTPステータス
- statusText : ステータスのテキスト
- headers    : レスポンスのヘッダー情報
- config     : リクエストした時axiosに渡した設定
- request    : このレスポンスに対応しているリクエストの内容

##　エラーハンドリング

エラーが起きた時に引数として渡される変数を使うとリクエストかレスポンスどちらでエラーが起きたか判別できる。

下で登場している`error`のメンバ

- response
- request
- message
- config

```js
axios.get('/user/12345')
  .catch(function (error) {
    if (error.response) {
      //レスポンスの結果がエラーの時
      //設定の　validateStatus　でエラーになるHttpステータスの範囲を変更できる。
      console.log(error.response.data);
      console.log(error.response.status);
      console.log(error.response.headers);
    } else if (error.request) {
      //レスポンスが受け取れなかった時
      //error.requestは環境に応じて以下のインスタンスを取る
      //  ブラウザ： XMLHttpRequest
      //  node.js: http.ClientRequest
      console.log(error.request);
    } else {
      //リクエストを準備している時にエラーが起きた時
      console.log('Error', error.message);
    }
    console.log(error.config);
  });
```

##　キャンセル

リクエストをキャンセルすることができる。

キャンセルする時はリクエストを送る際に設定の`cancelToken`に`axios.CancelToke.source.token`を渡す必要がある。

渡した後に`source.cancel()`を呼び出すことでキャンセルできる。

キャンセルした時はエラーが発生したと扱われ、キャンセルであることを見分けるには`axios.isCancel(error)`を使うといい。

```js
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token
}).catch(function (thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // handle error
  }
});

axios.post('/user/12345', {
  name: 'new name'
}, {
  cancelToken: source.token
})

source.cancel('Operation canceled by the user.');
```

また以下のような方法もある。

```js
const CancelToken = axios.CancelToken;
let cancel;

axios.get('/user/12345', {
  cancelToken: new CancelToken(function executor(c) {
    // An executor function receives a cancel function as a parameter
    cancel = c;
  })
});

// cancel the request
cancel();
```

## 複数のリクエストをまとめて送る

```js
//複数のリクエストをまとめて送ることもできる。
// そのとき、送るリクエストは関数の戻り値にすること
function getUser() { return axios.get('url1'); }
function getPost() { return axios.get('url2'); }
axios.all([getUser, getPost])
  .then(axios.spread((user, post) => {
    // レスポンスの処理
  }))
  .catch()
```

## 割り込み処理

`axios.interceptors`を使うとリクエスト/レスポンスに割り込み処理を挟むことができる。

```js
// リクエストへの割り込み
axios.interceptors.request.use(function (config) {
    // 送る前
    return config;
  }, function (error) {
    // エラーが起きた時
    return Promise.reject(error);
  });

// レスポンスへの割り込み
axios.interceptors.response.use(function (response) {
    // レスポンスを処理する前
    return response;
  }, function (error) {
    // エラーが起きた時
    return Promise.reject(error);
  });
```

`axios.interceptors.request.use()`と`axios.interceptors.response.use()`に空の関数を渡すことで、後から登録した関数を取り除くことができる

## リクエストのインスタンス化

`axios.create()`を使うことでリクエストのインスタンスを作ることもできる
引数には設定を渡すことができる。

あとから設定を変える時は`instance.defaults`を使うとできるが、リクエストを投げる時にも設定を指定でき、そちらが優先される。

また、インスタンスにも割り込み処理を登録できる。(`interceptors`)

```js
let instance = axios.create()
instance.defaults.timeout = 1000
instance.get({timeout: 5000}) //GETリクエスト

instance.interceptors.request.use(config => {
  //割り込み処理
})
```