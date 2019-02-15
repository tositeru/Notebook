# Browser Storage

[Web Storage API を使用する](https://developer.mozilla.org/ja/docs/Web/API/Web_Storage_API/Using_the_Web_Storage_API)

## Local Storage

手軽にデータを記録することができる。

[Local Storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)は開発者ツールから簡単に書き換えできるので、重要なデータを記録することには使わないこと。

代わりに、IDBやSimpleDBなどのJavaScriptモジュールを使うこと。


## 使い方

[Storage](https://developer.mozilla.org/en-US/docs/Web/API/Storage)インタフェースを使う。

使い方は簡単で、記録したいデータのキーとその値を指定するだけでいい。
ただし、自動的に文字列に変換される。

一応現在使用できるか確認をしておいたほうがいい。

```js

// set
window.localStorage.setItem('key1', 100)；
window.localStorage.setItem('color', "#FF0000")；

// get
let color = window.localStorage.getItem('color')； //<- "#FF0000"

for(let index=0; window.localStorage.length; ++index) {
    let value = window.localStorage.key(index); // get with index
    console.log(value);
}

window.localStorage.removeItem('color');

window.localStorage.clear();
```

ページで定められたプロトコルに違反するとSecurityErrorという例外を投げる。

関係するページ
[Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#Definition_of_an_origin)


## Session Storage

こちらも[Storage](https://developer.mozilla.org/en-US/docs/Web/API/Storage)インタフェースを使う。

`window.sessionStorage`からアクセスできる。

`window.sessionStorage`に保存されたデータはページのセッション終了時に消される。


