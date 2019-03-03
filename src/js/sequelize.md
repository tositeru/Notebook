# Sequelize

node.jsのためのPromiseベースのORMライブラリ。

ORMはObject-relational mappingの略語でデータベースとオブジェクト指向の言語の間の橋渡しを行うもののことである。

## Migrations

データべースのマイグレーションを行うには`sequelize-cli`を使用する。

マイグレーションを利用することでデータベースの変更を記録することができるので、データベースの修正はマイグレーションを通じて行うことを推奨する。

```sh
yarn add -D sequelize-cli
# 空プロジェクトを作成する
node_modules/.bin/sequelize init
```

空プロジェクトを作成したら、以下のフォルダーといくつかのファイルが作成される。

- config, 設定関係
- models, モデル関係
- migrations, マイグレーション関係
- seeders, シードファイル関係

プロジェクトのルートディレクトリに`.sequelizerc`ファイルを作ることで上のファイルの位置を変更できる。

例えば以下のようなファイル配置にしたい時は

- config     -> config/database.json
- models     -> db/models
- migrations -> db/seeders
- seeders    -> db/migrations

`.sequelizerc`にこのように書く

```js
const path = require('path')

module.exports = {
  'config': path.resolve('config', 'database.json'),
  'models-path': path.resolve('db', 'models'),
  'seeders-path': path.resolve('db', 'seeders'),
  'migrations-path': path.resolve('db', 'migrations')
}
```

### 設定

データベースにアクセスするために設定ファイルが必要になる。

設定ファイルは`json`か`js`ファイルで書くことができる。
jsファイルだと動的に設定を変えられるので、基本こちらを使うと思う。

設定ファイルの内容は`Sequelize`インスタンスに渡すパラメータと同じである。

また、設定ファイル内のパスのルートはプロジェクトルートディレクトリになる。

```js
module.exports = {
  development: {
    username: "root",
    password: null,
    database: "database_development",
    host: "127.0.0.1",
    dialect: "sqlite"
  },
  test: {
    username: "root",
    password: null,
    database: "database_test",
    host: "127.0.0.1",
    dialect: "sqlite"
  },
  production: {
    username: "root",
    password: null,
    database: "database_test",
    host: "127.0.0.1",
    dialect: "sqlite"
  }
}
```

### モデル作成と修正

`model:generate`でモデルを作成できる。

テーブルの名前とその要素は`--name`と`--attributes`オプションで指定できる。

```sh
node_modules/.bin/sequelize model:generate \
  --name User --attributes firstName:string,lastName:string,email:string
```
#### モデルの修正

一度作ったモデルの更新を行うには以下のコマンドでマイグレートファイルを作成し、その中に更新内容を書く必要がある。

`<filename>`には行う変更がわかるものにしたほうがいいと思う。

```sh
node_modules/.bin/sequelize migration:generate --name <filename>
```

作成したファイルにモデルの変更内容を書くには`QueryInterface`を利用する。

ただ、各々の関数の戻り値は`Promise`になるようになればいい。

```js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.addColumn('users', "id", Sequelize.INTEGER);
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface. removeColumn('users', 'id')
  }
};
```

### マイグレーションの実行

**モデルを作っただけではデータベースに反映されないので、`db:migrate`を実行する。**

もし、以前のマイグレーションの状態に戻したくなったら、`db:migrate:undo`を使用する。

```sh
# 最新の状態にする
node_modules/.bin/sequelize db:migrate

# 一つ前の状態に戻る
node_modules/.bin/sequelize db:migrate:undo

# 初期状態に戻る
node_modules/.bin/sequelize db:migrate:undo:all
```

### シード

シードを使うとデータベースにシードに記述したデータを挿入することができる。

利用するにはまずシードファイルを作成し、内容を記述していく。

その後、`db:seed`オプションでシード内容を元にデータベースを更新する。
**ただし2回以上実行すると、その回数分だけデータが追加されるので注意すること**

もし取り消したかったら、`db:seed:undo`オプションを使うといい。

```sh
# シードファイルを作成
node_modules/.bin/sequelize seed:generate --name demo-user
# シード適応
node_modules/.bin/sequelize db:seed --seed <seed-path>...
node_modules/.bin/sequelize db:seed:all
# シード適応を無効にする
node_modules/.bin/sequelize db:seed:undo --seed <seed-path>...
node_modules/.bin/sequelize db:seed:undo:all
```

シードファイルの内容は以下のようなものになる。

こちらも関数の戻り地は`Promise`になるようにし、`QueryInterface`を利用できる。

```js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.bulkInsert('Users',[
      {firstName: "Tom", lastName: "Sum", email: "a@bs.com", createdAt: new Date(), updatedAt: new Date()},
      {firstName: "Sum", lastName: "Tom", email: "a@bs.com", createdAt: new Date(), updatedAt: new Date()}
    ])
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.bulkDelete('Users', null, {});
  }
};
```
