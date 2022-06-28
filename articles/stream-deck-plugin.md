---
title: "Stream Deck Pluginの作り方 (TypeScript)"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["StreamDeck"]
published: false
---

# Stream Deck とは

# はじめに

プラグインは色々な言語で開発することができますがこの記事では TypeScript を使います。

また

# 開発

## ディレクトリ構成

```txt:ディレクトリ構成
├── com.example.plugin-name.sdPlugin
│   ├── images
│   │   └── actionIcon.png
│   │   └── actionIcon@2x.png
│   │   └── categoryIcon.png
│   │   └── categoryIcon@2x.png
│   │   └── icon.png
│   │   └── icon@2x.png
│   ├── manifest.json
│   ├── code.html
│   └── code.js
│   └── pi.html
│   └── pi.js
│   └── sdpi.css
├── src
│   └── code.ts
│   └── pi.ts
├── package.json
├── tsconfig.json
```

esbuild を使って TypeScript をコンパイルしています。

```json
"scripts": {
    "build:main": "esbuild ./src/main.ts --bundle --outfile=./net.ellreka.slack-status.sdPlugin/main.js --target=esnext",
    "build:pi": "esbuild ./src/pi.ts --bundle --outfile=./net.ellreka.slack-status.sdPlugin/pi.js --target=esnext",
    "dev": "yarn concurrently \"yarn build:main --watch\" \"yarn build:pi --watch\"",
    "build": "yarn build:main && yarn build:pi",
}
```

### manifest.json

プラグインのメタ情報を記述するファイルです。

詳しい項目はドキュメントを参照ください。

<https://developer.elgato.com/documentation/stream-deck/sdk/manifest/>

### code.html、code.ts

プラグインのメイン処理を記述するファイルです。

```html:code.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>com.example.plugin-nam</title>
    <script src="./code.js"></script>
  </head>
  <body></body>
</html>
```

```ts:code.ts
const connectElgatoStreamDeckSocket = (
  inPort: number,
  inPluginUUID: string,
  inRegisterEvent: any,
  inInfo: any
) => {
  const ws = new WebSocket(`ws://127.0.0.1:${inPort}`);
  ws.addEventListener("open", () => {
    ws.send(
      JSON.stringify({
        event: inRegisterEvent,
        uuid: inPluginUUID,
      })
    );
  });

  ws.addEventListener("message", async (e) => {
    const data = JSON.parse(e.data);
    const { event, payload, action } = data;
    switch (event) {
      case "keyDown":
        switch (action) {
          case ACTIONS.UPDATE_STATUS:
            await changeStatus(payload.settings);
            break;
          case ACTIONS.CLEAR_STATUS:
            await clearStatus(payload.settings);
            break;
        }
        break;
    }
  });
};

(window as any)["connectElgatoStreamDeckSocket"] =
  connectElgatoStreamDeckSocket;
```

### pi.html、pi.ts

Property Inspector の略で、プラグインの設定画面を表示するためのファイルです。
