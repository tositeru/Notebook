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

``` bash
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

## テスト

`@vue/test-utils`と`ava`を使って行う。
どちらも`nuxt-app`で自動的にインストールできるが、後からインストールする時は以下のコマンドを使う。

それと`jsdom`が必要になるので忘れずにインストールすること。

``` bash
yarn add -D @vue/test-utils ava jsdom
```

※2019/2/15現在
jsdomが原因によるエラーが起きてVueコンポーネントをインポートするとエラーが発生する状態にある

その際はどこかで以下のコードを追加する必要がある。
ava.config.jsのrequireプロパティに指定したテスト前に実行されるスクリプトに追記するのがお手軽。

[jsdom-global in mocha example may break modules bundled by rollup](https://github.com/vuejs/vue-test-utils/issues/936)
```js
window.Date = Date
```

`ava`をテスト実行用に使用し、`@vue/test-utils`でVueコンポーネントの検証に使う形になる。

### AVA


[AVA Configuration](https://github.com/avajs/ava/blob/master/docs/06-configuration.md)

`ava`を使ったテストは以下の形になる。

```js
import test from 'ava';

// 現在テストしているファイル名を表示する
console.log('Test currently being run:', test.meta.file)

//基本
test('test name', t => {
  //組み込みのアサーション関数
  t.pass('successed') //テストを成功させる
  t.fail('failed') //テストに失敗させる
  t.truthy(value, 'message')
  t.falsy(value, 'message')
  t.is(value, expected, 'message')
  t.not(value, expected, 'message')
  t.deepEqual(value, expected, 'message')
  t.notDeepEqual(value, expected, 'message')
  t.regex('abcde', /[a-e]+/)
  t.notRegex('abcde', /[A-E]+/)

  //前に記録したsnapshotとの比較を行うテスト
  // snapshotについてはあとで
  t.shapshot(expected, 'message')
  t.shapshot(expected, {id: 'specify snapshot name'}, 'message')

  // 例外のテスト
  const error = t.throws(()=> {
    throw new ExpectedExceptionType('')
  }, ExpectedExceptionType, 'message')
  //例外を投げない
  t.notThrows(() => {}, 'message')
  // asyncを使った例外のテスト。個々にあるもの以外にもパラメータがいくつかある
  await t.throwsAsync(async () => {
    throw new ExpectedExceptionType('check message')
  }, {instanceOf: ExpectedExceptionType, message: 'check message'})
  //例外を投げない
  await t.notThrowsAsync(promise);

})

//promiseを使ったもの
test('used promise test', t => {
  return somePromise().then(result => {
    t.is(result, 'apple')
  })
})

//asyncを使ったもの
test('used async test', t => {
  const result = await somePromise();
  t.is(result, 'apple')
})

//AVAにはビルドインのobservablesがある。
test('used observable test', t => {
  t.plan(3);
  return Observable.of(1, 2, 3, 4, 5, 6)
    .filter(n => {
      return n % 2 == 0;
    })
    .map(() => t.pass())
})

// onlyがついたものがあるとonlyがあるものだけがテストされる
test.only('only run this test', t => {
  t.pass()
})

// スキップするテスト
test.skip('skip this test', t => {
  
})

// 同時に実行させないテストの定義
// 並列に実行されるものの前にテストされる
test.serial('pass', t => {
  t.pass()
})

//Node.jsスタイルのエラーファーストコールバック
test.cb('Node.js-style error-first callback API', t => {
  fs.readFile('data.txt', t.end)
})
```

各テストにはHookを付けることもできる。

```js
// テスト予定のものを定義
test.todo('todo')
test.only.todo('todo')

//全てのテストの前/後処理
test.before(t => {})
test.after(t => {})

//各テストの前/後処理
test.beforeEach(t => {})
test.afterEach(t => {})

//afterXXXの後ろにalwaysを付けると失敗しても必ず実行される
test.after.always(t => {})
test.afterEach.always(t => {})

// serialを付けるとserialなテストのみに対しての処理になる
test.serial.before.always(t => {})
test.serial.after.always(t => {})

```

#### Snapshot

[Snapshot testing](https://github.com/avajs/ava/blob/master/docs/04-snapshot-testing.md)

AVAではファイルにある内容とテスト中の値を比較することができる。
この機能のことを**Snapshot**と呼ぶ。

[Jest](https://jestjs.io/docs/en/snapshot-testing)が発祥だそうだ。

Snapshotを使うと、1回目はテスト中に渡した値を元にテストファイルと同じ名前を持った2種類のファイルを作られ、渡した値がそこに記録される。

そして2回目以降は1回目に記録した値と比較され、異なっているとテストに失敗する。

例えば、`main.js`というファイルで使うなら、以下のファイルが新しく作られる。

- snapshots/main.js.snap : 比較対象となるファイル。手で書くのではなくテスト実行中のデータを保存する
- snapshots/main.js.md   : 上のファイルの内容

比較内容を更新したい時は以下のコマンドで行える。

``` bash
ava --update-snapshots
```

デフォルトだと生成されるディレクトリは`main.js`と同じディレクトリに作られるが、設定で生成先を変更することができる。

``` js
//ava.config.js
export default {
  // ...
  snapshotDir: "ava-snapshots",
  // ...
}
```

### Vue Test Utils

[Wrapper(Vue Test Utils)](https://vue-test-utils.vuejs.org/api/wrapper/#properties)

この中のselectorについては下で説明する。

```js
import {mount, shallowMount} from '@vue/test-utils'
import test from 'ava'
import TargetVueComponent from './' //todo ルートパスの書き方。

test('test title', t => {
  const wrapper = mount(TargetVueComponent) // Vueコンポーネントを作成。
  // 大規模なコンポーネントで子要素にアクセスしない時はshallowMountを使うと効率的
  const shallowWrapper = shallowMount(TargetVueComponent) // 子コンポーネントを作成せずに作成。

  let instance = wrapper.vm // 実体はvmの中にある
  wrapper.html() // 生のHTMLソース
  wrapper.text() // テキスト
  wrapper.props() // VueコンポーネントのProps
  wrapper.attibutes().id // Domノードの属性にアクセスする
  wrapper.attibutes('id')
  wrapper.classes() // DomノードのCSSクラスのリスト
  wrapper.classes('bar') // barを取得

  wrapper.trigger('click', { /*引数*/}) // イベント実行
  wrapper.$emit('custom-event', 123) // イベント発行

  //状態確認
  wrapper.exists() //作成に成功したか？
  wrapper.is('div') // 指定したselectorがあるか？
  wrapper.isEmpty() // 子要素がないとtrue
  wrapper.isVueInstance() // Vueインスタンスか？
  wrapper.isVisible() // v-showで非表示になっているか？

  // 検索
  let exist = wrapper.contains('div') // コンポーネントの中にdivタグがあるか？
  let div = wrapper.find('div') // コンポーネントの内のdivタグを探す
  let divList = wrapper.findAll('div') // コンポーネントの内のdivタグを探す
  t.is(wrapper.isVueInstance(), true) // Vueインスタンスかチェック

  // 値設定
  // Vue関係
  wrapper.setMethods({clickMethod: hoge()})
  wrapper.setData({hoge: 'foo'})
  wrapper.setProps({foo: 'bar'}) //VueのPropsを設定
  // form関係
  wrapper.find('input[type="radio"]').setChecked() // ラジオボタンをチェックする
  wrapper.findAll('option').at(1).setSelected() // option要素を選択する
  wrapper.find('input').setValue('set to input element') // input要素に値を設定。v-modelにバインドされている
})

```

### Selector
'@vue/test-utils'には[Selector](https://vue-test-utils.vuejs.org/api/selectors.html)があり、関数のいくつかはそれを利用できる。

`Selector`を使うと以下のものが検索できる。

- タグ
- CSSクラス
- タグの属性
- タグid
- Vueコンポーネント
    - [$ref](https://jp.vuejs.org/v2/api/index.html#ref)による検索もできる
