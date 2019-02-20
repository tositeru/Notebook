# Vue Router

[Vue.js公式ルータ](https://github.com/vuejs/vue-router)。
Vue.jsを使うならこれも使うべき。

Nuxt.jsの内部でも使われている。
Nuxt.jsで設定したい時は`nuxt.config.js`のrouterプロパティを使うといい。

[ドキュメント](https://router.vuejs.org/ja/)

## インストール

``` bash
yarn add -D vue-router
```

モジュールシステムを使う時は`Vue.use()`を使って明示的にルータをインストールする必要がある。

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```

## 使い方

Js側で設定を行い、`a`タグを<router-link>に置き換える形になる。

### <router-link>

[router-link](https://router.vuejs.org/ja/api/#router-link)

`a`タグの上位互換。
HTML5で追加された[History API](https://developer.mozilla.org/ja/docs/Web/API/History)を使ってくれるので、Vue.jsを使うならこちらを使ったほうがいい。

```html
<!-- router.push()を使う -->
<router-link to="/foo">Go to Foo</router-link>

<!-- replaceを付けると履歴に残さず、URLを変更する(中でrouter.replace()を使う) -->
<router-link to="/foo" replace>Replace to Foo</router-link>

<!-- appendを付けると現在のURLに対する相対パスになる -->
<router-link to="/foo" append>Replace to XXX/Foo</router-link>

<!-- exactを付けるとパスが完全に一致したときにだけアクティブクラス化になる -->
<router-link to="/foo" exact>Go to Foo(with exact)</router-link>

<!-- 好きなタグでaタグをラップできる -->
<router-link tag="li" to="/foo">
  <a>/foo</a>
</router-link>
```

ただしjs側でコードを書く必要があるので、そのコードは下の方で示す。

```js
// 公式ドキュメントから
// 0. モジュールシステムを使っている場合 (例: vue-cli 経由で)、Vue と VueRouter をインポートし、`Vue.use(VueRouter)` を呼び出します。

// 1. ルートコンポーネントを定義します
// 他のファイルからインポートすることもできます
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. ルートをいくつか定義します
// 各ルートは 1 つのコンポーネントとマッピングされる必要があります。
// このコンポーネントは実際の `Vue.extend()`、
// またはコンポーネントオプションのオブジェクトでも構いません。
// ネストされたルートに関しては後で説明します
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. ルーターインスタンスを作成して、ルートオプションを渡します
// 追加のオプションをここで指定できますが、
// この例ではシンプルにしましょう
const router = new VueRouter({
  routes // `routes: routes` の短縮表記
})

// 4. root となるインスタンスを作成してマウントします
// アプリケーション全体がルーターを認知できるように、
// ルーターをインジェクトすることを忘れないでください。
const app = new Vue({
  router
}).$mount('#app')

// これで開始です!

```

### アクティブクラス