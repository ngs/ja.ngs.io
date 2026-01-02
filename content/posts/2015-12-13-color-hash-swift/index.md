---
title: Swift の文字列から NSColor / UIColor を生成する
slug: color-hash-swift
description: Swift の文字列から NSColor / UIColor を生成するライブラリ ColorHash を公開しました。
date: "2015-12-13T17:00:00+09:00"
public: true
tags: ["color","swift","coccoa","ios"]
alternate: true
archives: ["2015-12"]
image: og.png
---
![](screen.gif)

Swift の文字列から `NSColor` / `UIColor` を生成するライブラリ [ColorHash] を公開しました。

**[ngs/color-hash.swift]**

```swift
import ColorHash

let str = "こんにちは、世界"
let saturation = CGFloat(0.30)
let lightness = CGFloat(0.70)

ColorHash(str, [saturation], [lightness]).color
```

このライブラリは JavaScript ライブラリ [Color Hash](https://github.com/zenozeng/color-hash) を参考に開発しました。

<!--more-->

インストール
----------

### CocoaPods

```rb
pod 'ColorHash'
```

### Carthage

```rb
github 'ngs/color-hash.swift'
```

[ngs/color-hash.swift]: https://github.com/ngs/color-hash.swift
[ColorHash]: https://github.com/ngs/color-hash.swift

