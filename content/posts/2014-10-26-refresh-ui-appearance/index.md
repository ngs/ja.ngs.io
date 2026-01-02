---
title: UIAppearance 変更時、リアルタイムに画面反映する
slug: refresh-ui-appearance
description: UIAppearance の proxy メソッドから見た目を変更して、即時全画面に適用する方法
date: "2014-10-26T21:30:00+09:00"
public: true
tags: ["ios","uiappearance","objective-c","swift","uikit"]
alternate: true
archives: ["2014-10"]
image: screen.gif
---
UIAppearance の proxy メソッドから見た目を変更して、即時全画面に適用する方法を調べていて、[UISS] という iOS で JSON 形式の Stylesheet を扱うライブラリにその答えがあったので、メモです。

<!--more-->



`window` の子ビューを全て剥がして貼り直しています。

`setNeedsDisplay`, `setNeedsLayout` も試してみましたが、動かなかったので、こちらを採用します。

Swift で書くとこんな感じです。



[UISS]: https://github.com/robertwijas/UISS

