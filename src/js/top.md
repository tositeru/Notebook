# JavaScript

- [Use Module List](./use-module-list.md)
- [Setup Manual for Nuxt's Project](./setup-manual-for-nuxt-project.md)
- [Vue/Nuxt](./vue-nuxt.md)
- [Only-Vue](./only-vue.md)
- [Vue Router](./vue-router.md)
- [Vuetify](./vuetify.md)
- [Browser Storage](./browser-storage.md)
- [axios](./axios.md)
- [node.js](./nodejs.md)
- [Sequelize](./sequelize.md)

## ファイル書き込み時の注意点

直ちにファイルに書き込んでほしいときは`fs.fsync`を呼び出すこと。

[参考記事](https://qiita.com/kazupaka/items/b6479f3f8d13347bf867?utm_source=Qiita%E3%83%8B%E3%83%A5%E3%83%BC%E3%82%B9&utm_campaign=d4b326c10b-Qiita_newsletter_353_03_13_2019&utm_medium=email&utm_term=0_e44feaa081-d4b326c10b-34477805)

ファイルの書き込み関数を使った後すぐに実際のファイルに保存されるわけではないので、すぐに書き込んでほしいときは[fs.fsyncSync](https://node.readthedocs.io/en/latest/api/fs/#fsfsyncfd-callback)を使う。

すぐに書き込まれないのはパフォーマンス上の都合で、これはOSレベルの話でほかの言語でも同じことである。

`fs.fsync`がファイル情報も書き出し、`fs.fdatasync`はデータのみをファイルに書き込む。
そのため、純粋にデータのみを書き込んでほしいときは`fs.fdatasync`を使うといい。

以下、参考記事から抜粋
`fs.fsync`は非同期的に書き込むので、すぐに書き込んでほしいときは`fs.fsyncSync()`を使うこと。

```js
const fs = require('fs')

fs.open('./fsync_test.txt', 'w', (err, fd) => {
    fs.write(fd, 'fsync() test\n', () => {

        // ここがポイント!!
        fs.fsyncSync(fd);

        response.writeHead(200, {'Content-Type': 'text/plain'})
        response.end('Write success\n');
        fs.close(fd, ()=>{});
    })
})
```
