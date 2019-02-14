# Browser Storage

## Local Storage

手軽にデータを記録することができる。

[Local Storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)は開発者ツールから簡単に書き換えできるので、重要なデータを記録することには使わないこと。

代わりに、IDBやSimpleDBなどのJavaScriptモジュールを使うこと。


## 使い方

[Storage](https://developer.mozilla.org/en-US/docs/Web/API/Storage)インタフェースを使う。

使い方は簡単で、記録したいデータのキーとその値を指定するだけでいい。
ただし、自動的に文字列に変換される。

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


