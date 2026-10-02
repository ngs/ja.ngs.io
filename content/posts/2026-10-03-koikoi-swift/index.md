---
title: iPhone / iPad / Mac / Apple Vision Pro で遊べる花札こいこい「Koikoi」
slug: "koikoi-swift"
description: 花札こいこいのアプリ Koikoi を、iPhone / iPad / Mac / Apple Vision Pro 向けにリリースしました。
date: "2026-10-03T06:00:00+09:00"
public: true
tags: ["koikoi","swift","swiftui","ios","macos","visionos","game","release"]
archives: ["2026-10"]
image: main.jpg
---

花札こいこいを遊べるアプリ **Koikoi** を、iPhone / iPad / Mac / Apple Vision Pro 向けにリリースしました。

広告もアプリ内課金もなく、オフラインで遊べます。

[App Store](https://apps.apple.com/app/koikoi-japanese-card-game/id6797163218) / [公式サイト](https://koikoiapp.ngs.io/ja/)

{{< youtube B4I2KRWN7bU >}}

<!--more-->

## モチベーション

[ターミナルで動く花札アプリ](/2026/06/15/go-koikoi/) を作ったり、子供達に幼少のときに英才教育するぐらい、花札を愛しています。

普段使っている iPhone や iPad、Mac でも、以前から余計な機能、有料・広告付きのアプリしか見つけられず、SwiftUI で作られた、クリーンな花札アプリが欲しいと思っていました。

それと、Vision Pro で XR 空間で遊べる花札を、実験的に作ってみたかったのもあります。

ルールの正解は Go 版の実装とテストにあるので、今回はそれを Swift に移して、札の見た目と、それぞれの端末での操作に手をかけることにしました。

## 使い方

起動すると対局設定の画面が出るので、ラウンド数 (3 / 6 / 12) と相手の強さ、配色を選んで始めます。

相手の強さは、かんたん・ふつう・つよい・たつじん の 4 段階です。

配色は、システムの設定に合わせるもののほかに、フェルト・畳・夜の 3 種類があります。

札は、タップでも、手札から場札へのドラッグ&ドロップでも、キーボードの矢印キーでも出せます。

画面には、成立している役に加えて「あと 1 枚」のリーチも出るので、役を覚えていなくても、次に何を狙えばよいかが分かります。

対局は自動で保存されるので、途中で閉じても、次に開いたところから続きを打てます。

Game Center のリーダーボードにも対応しています。

### Mac

Mac 版は、ウィンドウに合わせて、自分の取り札・場・相手の取り札を横に三列で並べています。

キーボードだけで最後まで遊べるようにしていて、矢印キーで札を選び、Return / Space で出し、Esc で取り消します。

ウィンドウは自由にリサイズでき、デスクトップが透けて見える半透明のウィンドウも選べます。

### Apple Vision Pro

visionOS 版は、平面のウィンドウを置くのではなく、volumetric window の中に実寸大の卓を置いています。

札は卓の上に並び、視線とピンチで選びます。

役のパネルとスコアボードは、下端のバーをつかんで好きな位置へ動かせて、動かした位置は保存されます (対局設定の「Reset Panel Positions」で元に戻せます)。

## 導入方法

App Store から入れられます。

iPhone / iPad / Mac / Apple Vision Pro で同じアプリです。

**[App Store](https://apps.apple.com/app/koikoi-japanese-card-game/id6797163218)**

必要なのは iOS 26 / macOS 26 / visionOS 26 以降です。

ソースコードは GitHub で公開していて、ルールエンジンのテストは手元で回せます。

```bash
git clone https://github.com/ngs/koikoi-swift.git
cd koikoi-swift
swift test                 # ルールエンジン・対戦相手・ビューモデルのテスト
tuist generate --no-open   # Xcode のワークスペースを生成
```

ただ、札の絵とアプリアイコンは別の private リポジトリに分けていて、この MIT ライセンスの対象外にしているため、公開リポジトリだけではアプリ本体はビルドできません。

## Under the hood

遊ぶうえでは知らなくても問題ない、技術的な裏側の話です。

Swift 6 で書いていて、モジュールは 3 つに分けています。

| モジュール | 中身 |
|---|---|
| `KoikoiCore` | 札の定義・役判定・ラウンドと対局の状態の管理。UI フレームワークに依存しない |
| `KoikoiAI` | 対戦相手 |
| `KoikoiUI` | SwiftUI のビューとビューモデル。全プラットフォームで共有 |

`KoikoiCore` は Go 版からの移植で、Go 側のテストも一緒に移しました。

札の ID (0〜47) の並びも Go 版と同じにしてあるので、ルール上の疑問が出たら、Go 版の実装とテストを正として直しています。

たつじんの打ち筋は determinized ISMCTS で、相手の手札と山は見えないため、辻褄の合う配り方をその都度仮に決めて、その状態で対局を最後まで回し、勝率のよかった手を選んでいます。

対局の保存は、盤面のスナップショットではなく、乱数のシードと全指し手の記録で、復元はリプレイで行っています。

## フィードバックのお願い

役の判定がおかしい、この操作で落ちた、相手のこの打ち方は変ではないか、といった報告は、とても助かります。

不具合の報告や機能のリクエスト、プルリクエスト、それから「遊んでみました」という一言も歓迎しますので、お気軽にお声がけください。

**https://github.com/ngs/koikoi-swift/issues**

息抜きに、ぜひ一局遊んでみてください 🎴
