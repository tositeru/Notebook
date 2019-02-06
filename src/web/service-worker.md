# Service Worker

参考サイト

[Service Workerの紹介](https://developers.google.com/web/fundamentals/primers/service-workers/?hl=ja)

## Service Workerとは？

上のリンクから

> Service Worker はブラウザが Web ページとは別にバックグラウンドで実行するスクリプトで、Web ページやユーザーのインタラクションを必要としない機能を Web にもたらします。 既に現在、[プッシュ通知](https://developers.google.com/web/updates/2015/03/push-notifications-on-the-open-web?hl=ja)や[バックグラウンド同期](https://developers.google.com/web/updates/2015/12/background-sync?hl=ja)が提供されています。 さらに将来は定期的な同期、ジオフェンシングなども導入されるでしょう。 このチュートリアルで説明する機能は、ネットワーク リクエストへの介入や処理機能と、レスポンスのキャッシュをプログラムから操作できる機能です。


## 前提条件

- Service Workerを対応しているブラウザを使う
- HTTPSが必要

Chromeが現時点で全ての機能を対応しているので、検証にはそれを使うのが良さそう。
GitHub Pagesがデモを公開するにはいい環境だそうだ。

## ライフサイクル

Service Workerには以下のようなライフサイクルがある。(画像も上のサイトから引用)

！[](https://developers.google.com/web/fundamentals/primers/service-workers/images/sw-lifecycle.png?hl=ja　”引用した画像”)


### インストール

はじめのインストールにはWebページ上のJavascriptを利用する必要がある。

```js
if ('serviceWorker' in navigator) {
  window.addEventListener('load', function() {
    navigator.serviceWorker.register('/sw.js').then(function(registration) {
      // Registration was successful
      console.log('ServiceWorker registration successful with scope: ', registration.scope);
    }, function(err) {
      // registration failed :(
      console.log('ServiceWorker registration failed: ', err);
    });
  });
}
```

基本的な流れは以下のものになる

ただし **Service　Workerを登録するタイミングによっては処理落ちする可能性があるので、全ての初期化処理が終わってからインストールした方がいい。**

1. ブラウザが対応しているかチェック
1. Service　Workerとして実行するJaveScriptファイルを指定

一度インストールされるとキャッシュされるため、以降再度ページにアクセスしてもキャッシュされたものが使われる。

キャッシュされたものがある状態で`navigator.serviceWorker.register()`を呼び出しても何もしないので安心。

参考サイト
[Service Worker 登録](https://developers.google.com/web/fundamentals/primers/service-workers/registration?hl=ja)

### ファイルの更新

何らかの作業でインストールしているService　Workerファイルの内容が変わった時は自動的に更新してくれる。(ただし、ファイルサイズが変わったときだけ)

更新の際は以下の流れを取る。**注意点として再度ページを開かないと新しいものが実行されない。**

> 1. Service Worker の JavaScript ファイルを更新します。 ユーザーがサイトに移動してきたとき、ブラウザは Service Worker を定義するスクリプト ファイルをバックグラウンドで再度ダウンロードしようとします。 現在ブラウザが保持しているファイルとダウンロードしようとするファイルにバイトの差異がある場合、それは「新しい」と認識されます。
> 2. 新しい Service Worker がスタートし、install イベントが起こります。
> 3. この時点では、まだ古い Service Worker が現在のページを制御しているため、新しい Service Worker は waiting 状態になります。
> 4. 開かれているページが閉じると、古い Service Worker は終了し、新しい Service Worker がページを制御するようになります。
> 5. 新しい Service Worker がページを制御するようになると、activate イベントが起こります。

installイベントのときにキャッシュ管理を行うと古い方の処理に影響がでてしまうため、activateイベントの際にキャッシュ管理を行うのを推奨している。

### Service Workerファイルの場所とfetchイベントの関係

`navigator.serviceWorker.register()`でService Workerをインストールするが、その際のファイルのパスによってService　Workerが受け取る`fetch`イベントが変わってくることに注意。

ファイルパスはドメインと関連しており、例えば`/example/sw.js`にあるService　WorkerはURLに`/example`が含まれているページからの`fetch`イベントを受け取る。

`/sw.js`だと、そのドメイン全てのページからの`fetch`イベントを受け取るようになる。

## Service Workerのコールバック

### installイベント

### fetchイベント

### fetchイベント


## 有効になっているService　Workerの確認

### Chrome

`chrome://inspect/#service-workers`から確認できる。

### Firefox
