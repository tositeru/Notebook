# Sequelize

node.jsのためのPromiseベースのORMライブラリ。

ORMはObject-relational mappingの略語でデータベースとオブジェクト指向の言語の間の橋渡しを行うもののことである。

## 初期化

ローカルに建てるかURL越しからアクセスするか選択できる。

`dialect`オプションで使用するデータベースを指定する。指定できるデータベースは以下のもの。
指定したデータベースによって使用できるオプションやSequelizeの機能に差が出てくる。

- mysql
- postgres
- sqlite
- mssql

[Sequelize](http://docs.sequelizejs.com/class/lib/sequelize.js~Sequelize.html#instance-constructor-constructor)

```js
const Sequelize = require('sequelize');

const sequelize = new Sequelize('databaseName', {
  dialect: 'mysql',
  host: 'my.server.tld',
  post: 9876
})

const sequelize = new Sequelize('www.example.com:9821/db_name', {
  dialect: 'mysql', // 使用するデータベース
})

```

**レプリケーションにも対応している。**

```js
const sequelize = new Sequelize('database', null, null, {
  dialect: 'mysql',
  port: 3306
  replication: {
    read: [
      { host: '8.8.8.8', username: 'read-username', password: 'some-password' },
      { host: '9.9.9.9', username: 'another-username', password: null }
    ],
    write: { host: '1.1.1.1', username: 'write-username', password: 'any-password' }
  },
  pool: { // If you want to override the options used for the read/write pool you can do so here
    max: 20,
    idle: 30000
  },
})
```

## 生クエリ

`Sequelize.query()`で生のSQL文を実行できる。

SQL文内のパラメータの置き換えを行うには`replacements`オプションを使う。

- ?, 配列を渡す時に使う
- :\<name\>, オブジェクトを渡す時に使う。

[Sequelize.query](http://docs.sequelizejs.com/class/lib/sequelize.js~Sequelize.html#instance-method-query)

```js
sequelize.query('SELECT * FROM Data', options)
  .then(rows => console.log(rows))

// パラメータ付き
sequelize
  .query(
    'SELECT * FROM projects WHERE status = :status ',
    { raw: true, replacements: { status: 'active' } }
  )
  .then(projects => {
    console.log(projects)
  })

sequelize
  .query('SELECT 1', {
    // A function (or false) for logging your queries
    // Will get called for every SQL query that gets send
    // to the server.
    logging: console.log,

    // If plain is true, then sequelize will only return the first
    // record of the result set. In case of false it will all records.
    plain: false,

    // Set this to true if you don't have a model definition for your query.
    raw: false,

    // The type of query you are executing. The query type affects how results are formatted before they are passed back.
    type: Sequelize.QueryTypes.SELECT
  })
```

## モデル

ORMなので、テーブルの内容をモデルとしてマッピングできる。

マッピングする内容はキーをカラム名、値をデータベースの型として持つオブジェクトを渡すことで指定できる。

[定義オプション](http://docs.sequelizejs.com/class/lib/model.js~Model.html#static-method-init)
[指定できるデータベースの型はこちら](http://docs.sequelizejs.com/variable/index.html#static-variable-DataTypes)

モデルを使ったデータの検索、作成などはこちらを参照

[使用方法](http://docs.sequelizejs.com/manual/tutorial/models-usage.html)
[Query](http://docs.sequelizejs.com/manual/tutorial/querying.html)

```js
const Users = sequelize.define('users', {
  firstName: Sequelize.STRING,
  lastName: Sequelize.STRING,
  description: Sequelize.TEXT
})

Users.sync() //テーブルの同期。なければ作成する
Users.drop() //テーブル削除

// 作成
//テーブルに追加せずインスタンス作成
let instanceNonPersistent = Users.build({firstName: 'hoge', lastName: 'foo', description: 'bar rab'})
//こちらはテーブルに追加する
let instance = Users.create({firstName: 'hoge', lastName: 'foo', description: 'bar rab'})

//検索
let instance2 = Users.findById(10)

//データ更新
instance.update({description: 'foooooof'}).then(() => {})

instance2.decription = 'hogehogehoge'
instance2.save().then(() => {})

// 現在のデータベースの内容と同期
instance.reload()

instance2.destroy()

//複数個に対する処理
Users.bulkCreate()
Users.update([{}, ...])
Users.destroy([user, user2,...])

//イベントハンドリング
Users.[sync|drop]().then(() => {
  // OK
}),catch(error => {console.error(error)})

//クラスのようにメソッドを追加できる
Users.classLevelMethod = function() {
  return 'foo'
}
// インスタンス単位
Users.prototype.instanceLevelMethod = function() { return 'bar'}

```

他のファイルに定義されたモデルを読み込むこともできる。

例えば以下のような内容の`project.js`がある場合

```js
module.exports = (sequelize, DataTypes) => {
  return sequelize.define("project", {
    name: DataTypes.STRING,
    description: DataTypes.TEXT
  })
}
```

`sequelize.import()`を使うことでそのモデルを使うことができる。

```js
const Project = sequelize.import('project.js')
```

### アクセサの定義

アクセサも定義できる。

カラムにアクセスする時は以下の関数を使うこと

- this.getDataValue('name')
- this.setDataValue('name', value)

#### 定義の方法

- カラム定義のオプションとして定義
    ```js
    const Users = sequelize.define('users', {
      firstName: {
        type: Sequelize.STRING,
        allowNull: false,
        get() {
          const firstName = this.getDataValue('firstName')
          return 'firstName is ' + firstName
        },
        set(val) {
          this.setDataValue('firstName', val.toUpperCase())
        }
      },
      lastName: Sequelize.STRING,
      description: Sequelize.TEXT
    })
    ```
- `sequelize.define()`のオプションとして定義
    ```js
    const Users = sequelize.define('users', {
      firstName: Sequelize.STRING,
      lastName: Sequelize.STRING,
      description: Sequelize.TEXT
    }, {
      getterMethods: {
        fullName() { return this.firstName + ' ' + this.lastName }
      },
      setterMethods: {
        fullName(value ) {
          const names = value.split(' ')

          this.setDataValue('firstName', names.slice(0, -1).join(' '))
          this.setDataValue('lastName', names.slice(-1).join(' '))
        }
      }
    })
    ```

### バリデーション

各カラムにはバリデーションを指定することができる。

アクセサと同じくカラム単位または`sequelize.define()`のオプションとして定義できる。

```js
const ValidateMe = sequelize.define('foo', {
  email: {
    type: Sequelize.STRING,
    validate: { isEmail: true }
  }
}, {
  //モデル全体のバリデーション
  validate: {
    checkEntity() {
      if (! (email typeof string)) {
        throw new Error('!!Occur error!!')
      }
    }
  }
});
```

#### 使用できるバリデータ一覧

[validator.js](https://github.com/chriso/validator.js)を内部で利用しているので、使用できるバリデータを確認する時はそちらも参照すること。

```js
const ValidateMe = sequelize.define('foo', {
  foo: {
    type: Sequelize.STRING,
    validate: {
      is: ["^[a-z]+$",'i'],     // will only allow letters
      is: /^[a-z]+$/i,          // same as the previous example using real RegExp
      not: ["[a-z]",'i'],       // will not allow letters
      isEmail: true,            // checks for email format (foo@bar.com)
      isUrl: true,              // checks for url format (http://foo.com)
      isIP: true,               // checks for IPv4 (129.89.23.1) or IPv6 format
      isIPv4: true,             // checks for IPv4 (129.89.23.1)
      isIPv6: true,             // checks for IPv6 format
      isAlpha: true,            // will only allow letters
      isAlphanumeric: true,     // will only allow alphanumeric characters, so "_abc" will fail
      isNumeric: true,          // will only allow numbers
      isInt: true,              // checks for valid integers
      isFloat: true,            // checks for valid floating point numbers
      isDecimal: true,          // checks for any numbers
      isLowercase: true,        // checks for lowercase
      isUppercase: true,        // checks for uppercase
      notNull: true,            // won't allow null
      isNull: true,             // only allows null
      notEmpty: true,           // don't allow empty strings
      equals: 'specific value', // only allow a specific value
      contains: 'foo',          // force specific substrings
      notIn: [['foo', 'bar']],  // check the value is not one of these
      isIn: [['foo', 'bar']],   // check the value is one of these
      notContains: 'bar',       // don't allow specific substrings
      len: [2,10],              // only allow values with length between 2 and 10
      isUUID: 4,                // only allow uuids
      isDate: true,             // only allow date strings
      isAfter: "2011-11-05",    // only allow date strings after a specific date
      isBefore: "2011-11-05",   // only allow date strings before a specific date
      max: 23,                  // only allow values <= 23
      min: 23,                  // only allow values >= 23
      isCreditCard: true,       // check for valid credit card numbers

      // custom validations are also possible:
      isEven(value) {
        if (parseInt(value) % 2 != 0) {
          throw new Error('Only even values are allowed!')
          // we also are in the model's context here, so this.otherField
          // would get the value of otherField if it existed
        }
      }
    }
  }
});
```

### 自動追加されるカラム

`sequelize.define()`を使いテーブルを定義するとデフォルトでID(`id`,主キー)と作成日時(`createdAt`)、更新日時(`updatedAt`)が追加される。

#### タイムスタンプ

作成日時と更新日時は`timestamps`オプションに`false`を設定すると自動的には追加はされない。
また`createdAt`や`updateAt`オプションに`false`を指定することでも追加されない。

ただし作成日時と更新日時はマイグレーションを使用する時使われていることに注意。

`paranoid`オプションを`ture`にすることで削除日時(`deletedAt`)を追加することもできる。
これがあるとデータを削除した時、その日時が記録されデータベース上からは削除されなくなる。

ここで挙げたタイムスタンプで使われるカラムはオプションから作成される名前を変更することができる。
また`timestamps`オプションが`false`なら、これらの機能は無効化される。

- `createdAt`オプション, 文字列を与えると作成日時のカラム名を変更できる
- `updateAt`オプション, 文字列を与えると更新日時のカラム名を変更できる
- `paranoid`オプション, `true`なら削除日時(`deletedAt`)カラムを追加する。
- `deletedAt`オプション, 文字列を与えると削除日時のカラム名を変更できる

```js
const Foo = sequelize.define('foo',  { /* bla */ }, {
  // don't forget to enable timestamps!
  timestamps: true,

  // I don't want createdAt
  createdAt: false,

  // I want updatedAt to actually be called updateTimestamp
  updatedAt: 'updateTimestamp',

  // And deletedAt to be called destroyTime (remember to enable paranoid for this to work)
  deletedAt: 'destroyTime',
  paranoid: true
})
```

## モデル間関係


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
`--attributes`オプションで指定できる型はSequelizeが用意している型と同じである。

[DataType](http://docs.sequelizejs.com/variable/index.html#static-variable-DataTypes)

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
