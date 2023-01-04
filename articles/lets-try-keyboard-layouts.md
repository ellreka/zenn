---
title: "色々なキーボード配列を試したり作成したり出来るサイトを作った"
emoji: "⌨"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["keyboard"]
published: false
---

## 概要

![](https://gyazo.com/9db742dc7682848ef22a998a7a9af5b5.png)

様々なキーボード配列(Dvorak, Colemak, Eucalyn など)を QWERTY 配列のまま試すことが出来たり、オリジナルの配列を作成して共有することが出来ます。

また自分の使っているキーボード配列を選択することが出来るため、Dvorak 配列 を使いつつ他の配列を試すことも出来ます。

※似たようなサイトはいくつかあります。

https://lets-try-keyboard-layouts-ellreka.vercel.app/

## 動機

QWERTY 配列に疑問を抱いている人が気軽に他の配列を試せるようにしたかったのが動機です。

またキーボード配列は有名だから良い、計測結果が優れているから良いというわけではなく、人それぞれの好みや使い方に合わせてカスタマイズすべきだと思っています。

そのため自分好みの配列を作成して共有出来るようにしました。

## 技術的な構成

フレームワークは Remix でデプロイ先は Vercel です。

もともと Cloudflare Workers で動かそうとしたのですが OGP 画像の生成周りが面倒(Vercel のが楽)だったので断念しました。

### OGP 画像の生成

作成した配列を共有するために動的 OGP 画像を実装しました。

https://twitter.com/ellreka/status/1609235541354250243

実装方法はいくつかありますが、今回はヘッドレスブラウザでスクリーンショットを撮ってくる方法を採用しました。

Remix には [Resource Routes](https://remix.run/docs/en/v1/guides/resource-routes) という機能があり画像や PDF などのファイルを返すことができます。

そこで playwright を動かし、撮影用のテンプレートページからスクリーンショットを撮ってきて返しています。

Vercel で Chrome を動かすために [chrome-aws-lambda](https://github.com/alixaxel/chrome-aws-lambda) というライブラリを使っていますが、 Node.js v16 には対応してないので注意が必要です。

以下記事をとても参考にさせていただきました。

https://dev.classmethod.jp/articles/remix-with-microcms-ogimage/

## 最後に

もし需要があれば Karabiner-Elements の設定ファイルを生成出来るようにしたら面白いなと思いました。(既にありそうですが)
