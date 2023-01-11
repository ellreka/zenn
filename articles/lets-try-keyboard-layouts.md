---
title: "色々なキーボード配列を試したり、カスタマイズ出来るサイトを作った"
emoji: "⌨️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["keyboard", "remix"]
published: true
---

## 概要

![](https://gyazo.com/9db742dc7682848ef22a998a7a9af5b5.png)

色々なキーボード配列(Dvorak, Colemak, Eucalyn など)を QWERTY 配列のまま試したり、配列をカスタマイズしてオリジナルの配列を作って共有することが出来ます。

※似たようなサイトはいくつかあります。

https://lets-try-keyboard-layouts.ellreka.net

## 動機

QWERTY 配列に疑問を抱いている人が気軽に他の配列を試せるようにしたかったのが動機です。

またキーボード配列は有名だから良い、計測結果が優れているから良いというわけではなく、人それぞれの好みや使い方に合わせてカスタマイズした方が良いと思っています。

そのため自分好みの配列を作成して試せるようにしました。

以下技術的な話。

## 技術的な構成

フレームワークは Remix で、デプロイ先は Vercel です。

### 配列データをクエリ文字列に変換する

ユーザーが配列を作成すると URL が発行され、その URL を開くと作成された配列が表示されるようになっています。

こんな感じ。

```
https://lets-try-keyboard-layouts.vercel.app/?q=1!%202%40%203%23%204%24%205%25%206%5E%207%26%208*%209(%200)%20%5B%7B%20%5D%7D%20'%22%20%2C%3C%20.%3E%20p%20y%20f%20g%20c%20r%20l%20%2F%3F%20%3D%2B%20a%20o%20e%20u%20i%20d%20h%20t%20n%20s%20-_%20%3B%20q%20j%20k%20x%20b%20m%20w%20v%20z
```

`JSON.stringify`で文字列化すれば楽なのですが、なるべく短い URL にしたかったので別の方法で実装しました。

### エンコード

```ts
export const encodeLayoutData = (layoutData: Array<string[] | string>[]) => {
  const flattenLayout = layoutData.flat().reduce<string[]>((acc, cur) => {
    if (Array.isArray(cur)) {
      return [...acc, cur.join("")];
    } else {
      return [...acc, cur];
    }
  }, []);
  const encoded = encodeURIComponent(flattenLayout.join(" "));
  return encoded;
};
```

```js:配列データ
[
  [["1", "!"], ["2", "@"], ["3", "#"], ["4", "$"], ["5", "%"], ["6", "^"], ["7", "&"], ["8", "*"], ["9", "("], ["0", ")"], ["-", "_"], ["=", "+"]],
  ["q", "w", "e", "r", "t", "y", "u", "i", "o", "p", ["[", "{"], ["]", "}"]],
  ["a", "s", "d", "f", "g", "h", "j", "k", "l", [";", ":"], ["'", '"']],
  ["z", "x", "c", "v", "b", "n", "m", [",", "<"], [".", ">"], ["/", "?"]],
]
```

まず配列をフラット化して 1 次元配列にします。

`["1", "!"]`のように２つのキーがある場合は join して`1!`という文字列にします。

<!-- prettier-ignore-start -->
```js:フラット化した配列データ
[ '1!', '2@', '3#', '4$', '5%', '6^', '7&', '8*', '9(', '0)', '-_', '=+',
  'q', 'w', 'e', 'r', 't', 'y', 'u', 'i', 'o', 'p', '[{', ']}',
  'a', 's', 'd', 'f', 'g', 'h', 'j', 'k', 'l', ';:', '\'"',
  'z', 'x', 'c', 'v', 'b', 'n', 'm', ',<', '.>', '/?']
```
<!-- prettier-ignore-end -->

次にこの配列をスペースで join して文字列にし、

```txt:スペースで区切られた文字列
1! 2@ 3# 4$ 5% 6^ 7& 8* 9( 0) -_ =+ q w e r t y u i o p [{ ]} a s d f g h j k l ;: '" z x c v b n m ,< .> /?
```

`encodeURIComponent`でエンコードします。

```txt:エンコード後
1!%202%40%203%23%204%24%205%25%206%5E%207%26%208*%209(%200)%20-_%20%3D%2B%20q%20w%20e%20r%20t%20y%20u%20i%20o%20p%20%5B%7B%20%5D%7D%20a%20s%20d%20f%20g%20h%20j%20k%20l%20%3B%3A%20'%22%20z%20x%20c%20v%20b%20n%20m%20%2C%3C%20.%3E%20%2F%3F
```

スペースで区切った理由はスペースがキーとして使われることがないからです。

`,`などで区切ってしまうとエンコード後に区切り文字の`,`なのか、キーの`,`なのか区別がつかなくなってしまいます。

### デコード

先程エンコードした文字列を受け取ったら、デコードして配列データに戻します。

逆のことをしてるだけなので説明は省きます。

今回は各段のキー数を固定しているので`slice`で決め打ちして取り出しています。

```ts
export const generateLayoutData = (encodedData: string) => {
  let layout = null;
  try {
    const decoded = decodeURIComponent(encodedData);
    const decodedArr = decoded.split(" ");
    const nestedArr = decodedArr.map((i) => {
      if (i.length === 1) {
        return i;
      } else {
        return i.split("");
      }
    });
    const row1 = nestedArr.slice(0, 12);
    const row2 = nestedArr.slice(12, 24);
    const row3 = nestedArr.slice(24, 35);
    const row4 = nestedArr.slice(35, 46);
    layout = [row1, row2, row3, row4];
  } catch (e) {
    console.error(e);
    layout = null;
  }
  return layout;
};
```

### OGP 画像の生成

作成した配列を共有するために動的 OGP 画像を実装しました。

https://twitter.com/ellreka/status/1609235541354250243

実装方法はいくつかありますが、今回はヘッドレスブラウザでスクリーンショットを撮ってくる方法を採用しました。

Remix には [Resource Routes](https://remix.run/docs/en/v1/guides/resource-routes) という機能があり画像や PDF などのファイルを返すことができます。

そこで playwright を動かし、撮影用の[テンプレートページ](https://github.com/ellreka/lets-try-keyboard-layouts/blob/main/app/routes/ogimages/template.tsx)からスクリーンショットを撮ってきて返しています。

Vercel で Chrome を動かすために [chrome-aws-lambda](https://github.com/alixaxel/chrome-aws-lambda) というライブラリを使っていますが、 Node.js v16 には対応してないので注意が必要です。

以下記事をとても参考にさせていただきました。

https://dev.classmethod.jp/articles/remix-with-microcms-ogimage/

## 最後に

もし需要があれば Karabiner-Elements の設定ファイルを生成出来るようにしたら面白そうだと思いました。
