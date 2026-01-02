---
title: hubot-browserstack でクロスブラウザのスクリーンショットを撮影する
slug: hubot-browserstack
description: Hubot にお願いして、BrowserStack でクロスブラウザのスクリーンショットを撮ってもらうスクリプトを公開しました。
date: "2014-06-08T09:30:00+09:00"
public: true
tags: ["hubot","browserstack"]
alternate: true
archives: ["2014-06"]
---
[Hubot] にお願いして、[BrowserStack] でクロスブラウザのスクリーンショットを撮ってもらうスクリプトを公開しました。

```
me > hubot screenshot me http://www.google.com/
hubot > Started generating screenshots in http://www.browserstack.com/screenshots/d804f186e460dc4f2a30849a9686c3a8c4276c21
```

このスクリプトを追加するには `npm install` コマンドを [Hubot] ディレクトリで実行して、

```bash
npm install --save hubot-browserstack
```

`hubot-browserstack` を `external-scripts.json` に追加してください。

```json
["hubot-browserstack"]
```

詳しくは GitHub リポジトリを参照してください: **[ngs/hubot-browserstack]**.

[Hubot]: https://hubot.github.com/
[BrowserStack]: http://www.browserstack.com/
[ngs/hubot-browserstack]: https://github.com/ngs/hubot-browserstack

