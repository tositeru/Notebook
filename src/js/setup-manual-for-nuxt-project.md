# Nuxtプロジェクトのセットアップマニュアル

## インストール
プロジェクトディレクトリの親ディレクトリから始める。

```bash
yarn create nuxt-app <project-name>
  Uses a custom server framework > express
  Choose features to install > Progressive Web App(PWA) Support, Linter / Formatter, Prettier, Axios
  Use a custom UI framework > bulma
  Use a custom test framework > ava
  Choose rendering mode > Universal
  Choose a package manager > yarn

cd <project-name>
yarn add body-parser express-session path sqlite whatwg-getch sequelize
yarn add -D jsdom nodemon redirect-ssl sass-loader vue-eslint-parser
```

## 設定

### package.json

- `scripts.lint`に`--fix`オプションを追加
- `scripts`に`test:update-snapshots: ava --update-snapshots`を追加

### nuxt

`nuxt.config.js`の`build.extend(config, ctx)`を以下の内容に変更

```js
if (ctx.isDev && ctx.isClient) {
  config.mode = 'development' // 追加

  config.module.rules.push({
    enforce: 'pre',
    test: /\.(js|vue)$/,
    loader: 'eslint-loader',
    exclude: /(node_modules)/
  })
}
```

#### https

次にhttpsの設定を行う。
開発用に秘密鍵と公開鍵を作成するが、公開するときにはこれらは使わないこと

以下のコマンドでキーを作成する

```bash
openssl req -newkey rsa:2048 -nodes -keyout develop.key -x509 -days 365 -out develop.crt
```

`nuxt.config.js`の`server`に以下のものを追記

```js
// ...
  server: {
    https: {
      key: fs.readFileSync(path.resolve(__dirname, 'develop.key')),
      cert: fs.readFileSync(path.resolve(__dirname, 'develop.crt')),
    }
  },
// ...
```

<em>設定しただけだとHTTPS化しないので、`server/index.js`の内容を書き換える。</em>
Node.jsの`https`を利用する形になる。

```js
// server/index.js
// ... 
// Listen the server
//app.listen(port, host) // <- もともとあったコード
let server = https.createServer(nuxt.options.server.https, app)
server.listen(port, host)
// ... 
```

### ava

#### ava.config.js
`ava.config.js`に以下の項目を追加。
テスト用のファイルの拡張子には`.test.js`を付ける。

```js
export default {
  // ...
  files: [
    "test/**/*.test.js"
  ],
  snapshotDir: "ava-snapshots",
  // ...
}
```

#### jsdomを使うことによる不具合の修正

<em>将来的には不要になるかもしれない。</em>

`test/helpers/setup.js`の末尾に次のコードを追加

```js
// fix a overrided Date class by jsdom
// https://github.com/vuejs/vue-test-utils/issues/936
window.Date = Date
```

#### 確認

ここまで来たら、動作チェックをする。

``` bash
yarn test
```

以下のエラーが発生した時は、jsdomの設定を見直す。

```js
TypeError: Super expression must either be null or a function

  _inherits (node_modules/@vue/component-compiler-utils/node_modules/prettier/index.js:1957:11)
  node_modules/@vue/component-compiler-utils/node_modules/prettier/index.js:40358:5
  node_modules/@vue/component-compiler-utils/node_modules/prettier/index.js:40378:4
  createCommonjsModule (node_modules/@vue/component-compiler-utils/node_modules/prettier/index.js:193:35)
  Object.<anonymous> (node_modules/@vue/component-compiler-utils/node_modules/prettier/index.js:40350:18)
  compile (node_modules/require-extension-hooks/src/hook.js:75:10)
  hook (node_modules/require-extension-hooks/src/hook.js:32:3)
  actuallyCompile (node_modules/@vue/component-compiler-utils/dist/compileTemplate.js:100:24)
  compileTemplate (node_modules/@vue/component-compiler-utils/dist/compileTemplate.js:31:16)
  getCompiledTemplate (node_modules/require-extension-hooks-vue/src/index.js:123:20)
```

<em>成功したら、`test/specs`を削除する。</em>

### サーバーサイドの拡張をするとき

`yarn nuxt-app`でexpressを使用してプロジェクトを作成するとプロジェクトディレクトリに`server`というディレクトリができる。
ファイルの更新があった時自動で更新してくれるため、そのディレクトリにサーバーサイドのコードを追加していく形を取る。

ただし、`server/index.js`というファイルが`yarn nuxt-app`によって作られているので、混乱が生じないようにコードは`server`の子ディレクトリを新しく作ってそこに追加していく形を取る。

新しくサーバーサイドのファイルを作った時は`nuxt.config.js`の`serverMiddleware`にそのファイルを追加していくこと。

```js
// ...
  serverMiddleware: [
    "~/server/child1/index.js",
  ],
// ...
```

以下のサーバーサイドのファイルのテンプレートを使用する。

```js
//~/server/child1/index.js
import express from 'express'
import utils from '../utils' // server/utils.jsへのパス

app = express()

export default {
  path: utils.makeMiddlewarePath(__dirname), // ディレクトリ構造にあったパスを設定する
  handler: app,
}
````

#### 認証機能

サーバーサイド側で認証機能を作りたいには以下の設定も追加する。

```js
  serverMiddleware: {
    bodyParser.json(),
    session({
      secret: 'super-secret-key',
      resave: false,
      saveUninitialized: false,
      cookie: { maxAge: 60000 }
    }),
  },
```

## 確認

設定が終わったら、次のコマンドが上手く動作するか確認する。

```bash
yarn dev
yarn build && yarn start
```
