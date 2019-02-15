# Vue.js/Nuxt.js


## プロジェクト構築

vue-cliを使う

開発環境の準備

```bash
# install yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt-get update && sudo apt-get install yarn

# install vue
sudo yarn -g install vue-cli
```

プロジェクトの作成

TypeScriptを使うため[typescript-template](https://github.com/nuxt-community/typescript-template)を使っている

```bash
# install vue and create project using typescript
sudo npm install -g vue-cli
vue init nuxt-community/typescript-template <project-name>
cd <project-name>
yarn
```

開発時の作成しているページの確認

```bash
yarn run dev
```

今回はGithub pageを作るために使っているので静的ページの作成する。以下のコマンドで作れる。

```bash
yarn run generate
```

注意点として、`vue init nuxt-community/typescript-template`を使って作ったプロジェクトはサーバーサイドレンダリングを使うように設定されている。

静的ページを作成する時はサーバーは使わないので、Nuxt.jsのモードを`spa`に指定する必要がある。

その方法は`nuxt.config.js`の`export`に`mode: 'spa'`を追記することでできる。
```js
//nuxt.config.js
export default {
  mode: 'spa',
  // ... 以下、他の設定
}
```

## SASSを使う

SASSを使う時はまず`node-sass`と`sass-loader`をインストールする

```bash
yarn add --dev node-sass sass-loader
```

SASSの定義にはファイルを使うかVueのプリプロセッサを使うかの二通りの方法がある

**ファイルを使う方法**

`nuxt.config.js`ファイル内でSASSファイルを指定すればいい。
```js
export default {
  css: [
    '@/assets/css/main.sass'
  ]
}
```

**プリプロセッサを使う方法**

こちらはVueファイルに埋め込む形になる。

```html
//直接指定できる
<style lang="sass">
.red {
  color: red;
  .inner {
    background-color: black;
  }
}
<\/style>
// ファイルを読み込むこともできる
<style lang="sass" src="#/styles/hoge.sass"></style>
```


## ESLintとPrettier

インストールには以下のコマンドを使う

```bash
yarn add --save-dev babel-eslint eslint eslint-config-prettier eslint-loader eslint-plugin-vue eslint-plugin-prettier prettier
```

その後に`.eslintrc.js`をルートディレクトリに置くことでESLintの設定ができる。

`nuxt.config.js`の`build`を修正することで、ホットリロードモードの時にESLintを実行することができるので、必ず有効化すること。

```js
build: {
  //ここで webpack config を拡張できます
  extend(config, ctx) {
    // Run ESLint on save
    if (ctx.isDev && ctx.isClient) {
      config.module.rules.push({
        enforce: "pre",
        test: /\.(js|vue)$/,
        loader: "eslint-loader",
        exclude: /(node_modules)/
      })
    }
  }
}
```

[eslint-と-prettier](https://ja.nuxtjs.org/guide/development-tools/#eslint-%E3%81%A8-prettier)

## ファイルパスにおいての~と@の扱い

`~`と`@`はWebpackが認識するパス記法のエイリアスになっている。

エイリアスには以下のものがある。

- `~`,`@`: [srcDir](https://ja.nuxtjs.org/api/configuration-srcdir)を表す。
- `~~`,`@@`: [rootDir](https://ja.nuxtjs.org/api/configuration-rootdir)を表す。

[srcDir](https://ja.nuxtjs.org/api/configuration-srcdir)と[rootDir](https://ja.nuxtjs.org/api/configuration-rootdir)は`nuxt.config.js`で設定できる。

## imgタグに画像のURLを指定する時の注意点(Nuxt.js)

v-bind:srcで設定するときと、直接srcを書き込む時でパスの指定方法が異なる。

webpackが関係している問題。<sub>(が、別のうまい方法がある気がする。)</sub>

**v-bind:srcのとき**

staticディレクトリに画像を保存した状態で下のコードみたいに書く

```js
<img v-bind:src="`images/${path}`">
```

[How to link img src to a rendered list #448](https://github.com/nuxt/nuxt.js/issues/448)

**直接srcを書き込む時**

この場合はassetsディレクトリに画像を保存した状態で以下のように書く。

```js
<img src="~assets/images/QR_URL_itch.png">
```

[css プロパティ](https://ja.nuxtjs.org/api/configuration-css/)
[プリプロセッサを使うには？](https://ja.nuxtjs.org/faq/pre-processors/)

## .VueファイルでSASS／SCSSを使う

[プリプロセッサの使用](https://vue-loader-v14.vuejs.org/ja/configurations/pre-processors.html)

.Vueファイル内でCSSを定義する時にlang属性を付けるとSASS/SCSSなど使えるようになる。

その際にはyarnなどで別個でsass-loaderとnode-sassパッケージをインストールする必要がある。

```html
<style lang="sass">
  /* ここにSassを書きます */
</style>
```

## プロジェクト環境の構築

```bash
yarn create nuxt-app my-project
#　いろいろな設定を行う

# vue
yarn add -D vue-propetry-decorator vue-eslint-parser

# typescript
yarn add -D nuxt-ts ts-loader typescript
```

### ESLint対応
ESLintをVueに対応させるには`vue-eslint-parser`をインストールする必要がある。

[vue-eslint-parser](https://github.com/mysticatea/vue-eslint-parser)

```
yarn add -D vue-eslint-parser
```

あとは`.exlintrc.js`も修正すればOK。
`eslint`を実行する時に`.vue`も対象に含めるために、`--ext`オプションを付けるようにすること。
(Nuxt.jsではデフォルトでそうなっている)

```js
module.exports = {
  // ...
  parser: "vue-eslint-parser",
  // ...
  extends: [
    // ...
    'eslint:recommended',
    // ...
  ],
  // ...
} 
```
