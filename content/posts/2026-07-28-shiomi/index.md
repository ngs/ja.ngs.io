---
title: 潮見表アプリ Shiomi
slug: "shiomi"
description: 軽量・シンプル・無料・広告なしの潮見表アプリ Shiomi を iOS / iPadOS / macOS / visionOS / watchOS 向けに公開しました。オフラインで潮汐予測が動きます。
date: "2026-07-28T08:00:00+09:00"
public: true
tags: ["shiomi","tides","swift","swiftui","ios","macos","visionos","watchos","fishing"]
archives: ["2026-07"]
image: main.jpg
---

iPhone / iPad / Mac / Apple Vision Pro / Apple Watch で動く、潮見表アプリ **Shiomi（潮見表）** を公開しました。

無料で、広告もアプリ内課金もありません。

[App Store](https://apps.apple.com/app/shiomi-simple-tide-chart/id6754582175) / [公式サイト](https://shiomi-app.ngs.io/ja/)

## 特徴

すでにある他の潮見表アプリと違い、緯度経度をもとに、その地点の調和定数（潮汐を構成する分潮のパラメータ）をダウンロード・アプリ内に保存し、潮位の予測は通信をおこなわず、端末内で計算するので、オフラインや電波状況が悪いフィールドでもサクサク動きます。

<!--more-->

## なぜ作ったか

日々釣りのことばかり考えて過ごしているので、天気アプリと潮位表を頻繁に確認します。

普段使いする上で、天気アプリは [Windy] 有料版で満足しているのですが、潮位については使っていてストレスのないアプリがありませんでした。

- 課金が必要・広告が出る
- 時合予測など、不要な機能が付いている
- OS になじまない、オリジナルなユーザーインターフェース
- Apple Watch 専用で、iPhone や Mac では見られない

これらを克服するため、自前でパラメータを返す API を構築し、クライアントは SwiftUI で実装し、ストレスのない、軽量でシンプルなソフトウェアを開発しました。

## 使い方

![](points.jpg)

潮位表を見たい地点を登録し、選択するだけのシンプルなユーザーインターフェースです。

![](map.jpg)

追加 (+) ボタンからマップを起動し、パラメータ取得を実行すると、地点追加完了です。

## オープンソース

アプリケーションはすべて GitHub で公開しています。

- アプリ本体: [ngs/tides-swift](https://github.com/ngs/tides-swift)
- 潮汐計算 API: [ngs/tides-api](https://github.com/ngs/tides-api)

不具合を発見し、修正方法が分かったら、プルリクエストを送っていただけると嬉しいです。

## フィードバックのお願い

使ってみて気づいたこと、欲しい機能、表示がおかしい地点などがあれば、[GitHub Issues](https://github.com/ngs/tides-swift/issues) で教えてください。

特に、普段釣りやサーフィンで潮見表を使っている方の「ここが足りない」が一番ありがたいです。

[Windy]: https://www.windy.com/