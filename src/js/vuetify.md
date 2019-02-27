# Vuetify

Vue.js用のMaterial Component Framework

[公式サイト](https://vuetifyjs.com/ja/)

## Grid System

12分割グリッドと[Flex](https://developer.mozilla.org/ja/docs/Web/CSS/flex)を利用したレイアウトが提供されている。

グリッドはv-containerとv-layout、v-flexを組み合わせて表現され、v-containerを親として左から右にかけて親子関係がある。

一応必ず組み合わせなくても動作はするし、入れ子にすることもできるため親子関係が逆転しても問題なく動作する。
が、それぞれ役割があるのでうっかりそれを忘れると痛い目にあう。

- v-container: ページ中央にセンタリングする。 `fluid`プロパティを渡すと横幅をフルに展開する。
- v-layout:　子要素の並び方の制御を行う。セクションの分割に使用する。
- v-flex:　主にセクションの横幅やオフセットを指定するために使用する。`flex: 1 1 auto`を自動的に設定してくれる。

個人的にはv-layoutが一番使用されると思う。

### ディスプレイサイズ

Vuetifyのグリッドシステムにはディスプレイのサイズに合わせて複数の単位が存在し、Breakpointと呼ばれている。

詳しくは以下を参照。

[Breakpoints](https://vuetifyjs.com/ja/framework/breakpoints)
[Display](https://vuetifyjs.com/ja/framework/display)

### Breakpoints

以下のものがある。

\*がついている数値はデスクトップ上で-16pxされる。

- xs - < 600px
- sm - 600px > < 960px
- md - 960px > < 1264*
- lg - 1264 > < 1904px*
- xl - > 1904px*

vuetifyでは`$vuetify.breakpoint`オブジェクトを提供しており、現在のディスプレイを簡単に取得できるようになっている。

```js
{
  xs: boolean
  xsOnly: boolean
  sm: boolean
  smOnly: boolean
  smAndDown: boolean
  smAndUp: boolean
  md: boolean
  mdOnly: boolean
  mdAndDown: boolean
  mdAndUp: boolean
  lg: boolean
  lgOnly: boolean
  lgAndDown: boolean
  lgAndUp: boolean
  xl: boolean
  xlOnly: boolean
  name: ('xs' | 'sm' | 'md' | 'lg' | 'xl')
  width: number
  height: number
  thresholds: { xs: number, sm: number, md: number, lg: number }
  scrollbarWidth: number
}
```

### 非表示設定

`hidden-{breakpoint}-{condition}`を使うことでディスプレイのサイズに合わせて要素を非表示にすることができる。

condition一覧
- only : 指定したbreakpointのみ非表示にする
- and-down : 指定したbreakpoint以下のものを非表示にする
- and-up : 指定したbreakpoint以上のものを非表示にする

`overflow-hidden`または`overflow-{axis}-hidden`ではみ出したものを非表示にできる。

axis一覧
- x : X軸
- y : Y軸

## テキストアライメント

`text-{breakpoint}-{position}`の形式で指定できる。

`{breakpoint}`で指定したサイズ以上で適応される。

そうでなかったら、左詰めになる。

```html
<p class="text-lg-right">Right align on large viewport sizes</p>
<p class="text-md-center">Center align on medium viewport sizes</p>
<p class="text-sm-left">Left align on small viewport sizes</p>
<p class="text-xs-center">Center align on all viewport sizes</p>
<p class="text-xs-right">Right align on all viewport sizes</p>
```

## タイポグラフィ

VuetifyではMaterial Design用のフォントサイズを用意している。

> - .display-4 - Good for `<h1>`
> - .display-3 - Good for `<h2>`
> - .display-2 - Good for `<h3>`
> - .display-1 - Good for `<h4>`
> - .headline - Good for `<h5>`
> - .title - Good for `<h6>`
> - .subheading - Good for supporting text
> - .body-2 - Regular body text with additional weight
> - .body-1 - Regular body text
> - .caption - Smaller size text

線の細さもある

> - .font-weight-thin - Sets font-weight to 100
> - .font-weight-light - Sets font-weight to 300
> - .font-weight-regular - Sets font-weight to 400
> - .font-weight-medium - Sets font-weight to 500
> - .font-weight-bold - Sets font-weight to 700
> - .font-weight-black - Sets font-weight to 900

斜体には以下のものを使う

> - .font-italic

cssの`text-transform`にも対応している。

> - .text-capitalize - Sets text-transform to capitalize
> - .text-lowercase - Sets text-transform to lowercase
> - .text-none - Sets text-transform to none
> - .text-uppercase - Sets text-transform to uppercase

折り返し指定もある。

> - .text-no-wrap - Sets whitespace to no-wra
> - .text-truncate - Truncates overflowed text

## 色

Vuetifyではクラスに設定するだけで簡単に色を変更できるようになっている。

以下の形式を取る。
`--`で区切られているが、順序は変わってもいい。(ただし{lighten|darken|accent}-{1-5}はセット)

color以外は省略可能である。

- {text|background}--{color}--{lighten|darken|accent}-{1-5}

※darkenとaccentの数値は{1-4}の範囲になる。

定義は[こちら](https://github.com/vuetifyjs/vuetify/blob/master/packages/vuetify/src/stylus/settings/_colors.styl)にある。

```html
<div class="red"></div>
<span class="red--text"></span>
<div class="red--lighten-5"></div>
```

Javascriptからも使用できる。

```js
// src/index.js

// Libraries
import Vue from 'vue'
import Vuetify from 'vuetify'

// Helpers
import colors from 'vuetify/es5/util/colors'

Vue.use(Vuetify, {
  theme: {
    primary: colors.red.darken1, // #E53935
    secondary: colors.red.lighten4, // #FFCDD2
    accent: colors.indigo.base // #3F51B5
  }
})
```
### 色一覧

- red
- pink
- purple
- deep-purple
- indigo
- blue
- light-blue
- cyan
- teal
- green
- light-green
- lime
- yellow
- amber
- orange
- deep-orange

accentがないもの
- brown
- blue-gray
- grey

固定
- black
- white
- transparent

