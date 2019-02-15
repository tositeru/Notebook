# Nuxt.jsを使わずにそれっぽい環境を構築する

Nuxt.jsとTypeScriptを組み合わせていく内によくわからないことになったので、Vue.jsのみでNuxt.jsと同じ環境っぽいものを構築する方法をメモする

## 目標

以下の機能が使える環境を目指す

- yarn
- Hot Reload機能(webpack)
- TypeScript
- テスト(mocha-webpack)
- コードフォーマッター(ESLint&Prettier)

参考サイト

TypeScriptを利用する時の参考資料
[TypeScript Vue Starter](https://github.com/Microsoft/TypeScript-Vue-Starter)

DevServerを立てるための参考資料
[DevServer](https://webpack.js.org/configuration/dev-server/#devserver-host)
[webpack-dev-server](https://webpack.js.org/guides/development/)

## yarn

パッケージ依存関係管理ツール。
npmにセキュリティ的な問題があったので、その代替ツールとして誕生したもの。
とても速くて安全だそうだ。

ともかく、Javascriptでプロジェクトを作るにはまずパッケージ依存関係管理ツールを使用する。

yarnを使ってvueやwebpackやら色々なパッケージをインストールしていく。

### Install

インストールは以下のコマンドを使う。
[Installation](https://yarnpkg.com/en/docs/install#debian-stable)

```bash
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt-get update && sudo apt-get install yarn
```
### 使い方

```bash
#プロジェクト作成
yarn init

yarn add <option> <package-name>
yarn remove <package-name>
```

`yarn`コマンドでプロジェクトに登録されているパッケージを全てインストールできる。
gitとかからプロジェクトを持ってきた時に使う。

```bash
yarn
# or
yarn install
```

`yarn add`でパッケージを追加する時、追加で以下のオプションが指定できる。

- 何も付けない: 普通の依存パッケージ。プロジェクトのコードが実行しているときも使われるパッケージとして登録する。
- -D: 開発環境時のときだけ使用するパッケージとして使用する。
    デプロイや公開するものには含まれない。
- -P: 自分でパッケージを公開する際にも必要となるパッケージとして登録する
- -O: 必ずしも使わないパッケージとして登録する

## webpack

サイトで使用するファイルを設定した形でまとめてくれるツール。

複数のJavaScriptファイルを一つにまとめたり、リソースを埋め込んだりできる。

それ以外にも開発環境として便利な機能があって、Javascriptでサイトを使うなら是非使いたいツールである。

### 設定

`webpack.config.js`ファイルをプロジェクトのルートディレクトリに置いておくと認識してくれる。

無い場合はデフォルトのものが使用される。
[デフォルトの設定](https://github.com/webpack/webpack/blob/master/lib/WebpackOptionsDefaulter.js)

また、`--config`オプションで直接指定できる。

```bash
webpack --config myconfig.js
```

#### 使用するファイルの設定

起点となるファイルを指定して、その出力先を決める形になる。
そのままだとJavaScriptのみをまとめるだけだが、`html-webpack-plugin`というプラグインを使うことでWebページのトップ画面となるHTMLファイルも出力してくれる。

また、`xxx-loader`などを利用することで、CSSや画像もまとめることもできる。

上で挙げた機能を使うには`yarn`を使ってインストールする必要がある。

[html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)
[css-loader](https://github.com/webpack-contrib/css-loader)
[file-laoder](https://github.com/webpack-contrib/file-loader)
[webpack-manifest-plugin](https://github.com/danethurber/webpack-manifest-plugin)

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CleanWebpackPlugin = require('clean-webpack-plugin');
const ManifestPlugin = require('webpack-manifest-plugin');

module.exports = {
  // 一つのファイルにまとめる際にルートとなるファイルの指定
  entry: {
    app: './src/index.js',
  },
  // 出力先の指定
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: ['file-loader'],
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: ['file-loader'],
      }
    ]
  },
  plugins: [
    new ManifestPlugin(),
    new CleanWebpackPlugin(['dist']),
    new HtmlWebpackPlugin({
      title: 'Development'
    }),
  ],
};
```

## webpack watch

プロジェクト内のファイルの更新があったら自動的にビルドしてくれる機能。

`webpack --watch`で利用できるが、`webpack-dev-server`パッケージという便利なものがあるので基本的にそちらを使う。

`webpack-dev-server`の設定には[devServer](https://webpack.js.org/configuration/dev-server/)を主に使う。

[Guide](https://webpack.js.org/guides/development/)
[Watch Configuration](https://webpack.js.org/configuration/watch/)

最低限以下の設定をしないと動作しないので注意

```js
module.exports = {
  //最低限の機能
  mode: 'development',
  watch: true,
  watchOptions: {
    aggregateTimeout: 300,
    poll: 100,
  },
  //webpack-dev-serverの設定
  devServer: {
    contentBase: './dist',
  },
  //一つのファイルにまとめた時に元のファイルの場所がわかるようにするための設定
  devtool: 'inline-source-map',
};
```

`development`モードでないと動作しないが、デプロイするたびに必要な設定を手作業で変更していたら大変なので、以下のコードで動的に切り変えるようにする。

```js
//webpack.config.js
var config = {
  // 変数に設定を格納する
}

module.exports = (env, argv) => {
  switch(argv.mode) {
    case 'development':
      config.devtool = 'source-map';
      break;
    case 'production':
    break;
  }
  return config;
}
```

それか、[こちらのページ](https://webpack.js.org/guides/production/#setup)から、以下のような複数のwebpackの設定ファイルを用意する方法もある。

個人的にはこちらの方が好み。

- 共通の設定を書いたファイル(webpack.common.js)
- 開発時の設定ファイル(webpack.dev.js)
- 公開時の設定ファイル(webpack.prod.js)

`webpack.common.js`を他のファイルからimportして使っている形になる。
また、内部で`webpack-merge`パッケージを使っているのでインストールする必要がある。

```js
// webpack.common.js
// 共通の設定
const path = require('path');
const CleanWebpackPlugin = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
  entry: {
    app: './src/index.js'
  },
  plugins: [
    new CleanWebpackPlugin(['dist']),
    new HtmlWebpackPlugin({
      title: 'Production'
    })
  ],
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

```js
//webpack.dev.js
//開発時の設定
const merge = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'development',
  devtool: 'inline-source-map',
  devServer: {
    contentBase: './dist'
  }
});
```

```js
//webpack.prod.js
//公開時の設定
const merge = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'production'
});
```

## webpack HMR

watch機能を使うと変更があった時、ブラウザで毎回ページを更新し直さないとその確認ができない。

**Hot Module Replacement** と呼ばれる機能を使うことで、自動的に更新してくれるようになり、より楽に作業が勧められるようになる。

ただし、Javascriptコードを追加で書かないといけない。

[webpack HMR](https://webpack.js.org/concepts/hot-module-replacement/)
[Guide](https://webpack.js.org/guides/hot-module-replacement/)
[API](https://webpack.js.org/api/hot-module-replacement/)

webpackの設定は下のようなものになる。
```js
// webpack.config.js
const webpack = require('webpack');

module.exports = {
  // ...
  //webpack-dev-serverの設定
  devServer: {
    contentBase: './dist',
    hot: true
  },
  plugins: [
    // ...
    new webpack.HotModuleReplacementPlugin(),
  ],
  // ...
};
```

設定ができたら、Javascriptのコードを追加していく。

```js
// ...

// Hotリロードが有効かチェック
if (module.hot) {
  // ./print.jsが更新された時の処理を書いていく。
  module.hot.accept('./print.js', function() {
    console.log('Accepting the updated printMe module!');
    //
    // 望んだ形になるように、直接DOMを書き換える処理を書いていく形になる。
    //
  })
}

```
## CSS/SASSのMinimumize

`optimize-css-assets-webpack-plugin`がなぜか使えなかったので、代わりに`cssnano`と`postcss`を使う。

なので、webpackとは別工程でCSSの生成を管理することになる。

また、SASSも使うので、変換の流れは以下のようなものになる。

1. SASS -> CSS変換
1. nanocssでファイルサイズを削減
1. webpackによるプロジェクトへの組み込み

webpack以外の工程は以下のコマンドで行える。

```js
sass sass/:src/css && postcss src/css/**/*.css --replace
```

sassとpostcssは共にファイル更新機能があるが、Vue.jsでインラインCSSをメインに使うかもしれないので、今のところは手動でコンパイルしていく。

## ダイナミックインポート

EMACScript2015から`import`を式の中にかけたり、Promiseみたいに使えるようになった。

これによって実行時に必要なものだけをブラウザに読み込めたりと便利になったそうだ。

この新しく追加された`import`の機能のことをダイナミックインポートと呼ばれている。

[webpack code-splitting](https://webpack.js.org/guides/code-splitting/)
[Webpack and dynamic import](https://medium.com/front-end-weekly/webpack-and-dynamic-imports-doing-it-right-72549ff49234)

importのところにある`/* webpackChunkName: "lodash" */`はwebpackのマジックコメントという機能である。

[module-methods magic-comments](https://webpack.js.org/api/module-methods/#magic-comments)

```js
import printMe from './print.js'

function getComponent() {
  // コメントはwebpackのマジックコメント
  return import(/* webpackChunkName: "lodash" */ 'lodash').then(({default: _}) => {
    let element = document.createElement('div');
    element.innerHTML = _.join(['Hello', 'webpack']);
    printMe();// 今までのimportも使える
    return element;
  }).catch(error => 'An error occurred while loading the component');
}

getComponent().then(component => {
  document.body.appendChild(component)
})
```

async化もできる。

```js
async function getComponent() {
  let element = document.createElement('div');

  const {default: _ } = await import(/* webpackChunkName: "lodash" */ 'lodash');
  element.innerHTML = _.join(['Hello', 'webpack']);

  return element;
}

getComponent().then(component => {
  document.body.appendChild(component)
}).catch(error => 'An error occurred while loading the component');
```
### webpackのモジュール解析ツール

生成したモジュールの構成を解析してくれるツールをwebpack側で提供してくれている。

それらのツールを使う際は次のコマンドで生成するファイルが必要になる。
```js
webpack --profile --json > stats.json
```

- [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)
- [webpack-chart](https://alexkuz.github.io/webpack-chart/)
- [Webpack Visualizer](https://chrisbateman.github.io/webpack-visualizer/)
- [Bundle optimize helper](https://webpack.jakoblind.no/optimize)

## TypeScript
TypeScriptを使う時は以下のパッケージをインストールする

```bash
yan add -D typescript ts-loader
```

インストールが終わったらTypeScriptの設定ファイルをプロジェクトに追加する。

```json
{
  "compilerOptions": {
    "outDir": "./dist/",
    "sourceMap": true, // ソースマップを有効化するにはwebpack側にで設定をすること
    "noImplicitAny": true,
    "module": "es6",
    "target": "es5",
    "jsx": "react",
    "allowJs": true
  }
}
```

Webpackの設定ファイルも修正する。

```js
const path = require('path');

module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      }
    ]
  },
  resolve: {
    extensions: [ '.tsx', '.ts', '.js' ]
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```
