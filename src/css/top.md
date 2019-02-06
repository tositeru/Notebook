# CSS

## Fontawesomeアイコンのサイズを変更してもアイコンが中央に来るようにするためのパラメータ。

Fontawesomeアイコン専用というわけではなく、なんにでも使える方法

[参考サイト](https://saruwakakun.com/html-css/basic/centering)

```css
親要素 {
  position: relative;
}

Fontawesomeを指定したi {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translateX(-50%) translateY(-50%);
}

```
