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

このアプリは Swift 6 で実装しており、以下の 3つの構成でできています。

| モジュール | 役割 | 主な型 |
|---|---|---|
| `KoikoiCore` | 札の定義・役判定・ラウンドと対局の進行。Go 版からの移植で、Foundation 以外に依存しない | `Card` / `Game` / `YakuChecker` / `HeuristicOpponent` |
| `KoikoiAI` | 対戦相手の探索 | `RoundSimulator` / `Determinizer` / `ISMCTSEngine` |
| `KoikoiUI` | SwiftUI のビューとビューモデル。全プラットフォームで共有 | `GameViewModel` / `GameRecord` |

`KoikoiAI` は歴史的な経緯で AI と名乗っていますが、LLM や機械学習のモデルは使っておらず、Pure Swift でアルゴリズムを実装しています。

相手の強さとの対応は次のとおりです。

- かんたん・ふつう・つよい: `HeuristicOpponent` の評価ルール (Go 版の `cpu.go` の移植) で、札の価値を 光 20・タネ 10・短冊 5・カス 1 として、取れる札の合計がいちばん高い手を選ぶ
- かんたん: 3 回に 1 回はランダムに出し、こいこいはしない
- つよい: 光や、猪鹿蝶・赤短・青短の札を取れる手にボーナスを足し、手札に余裕があれば積極的にこいこいする
- たつじん: `ISMCTSEngine` の探索 (情報集合モンテカルロ木探索) で、相手の見えない札を仮定して、1 手ごとに 400 回シミュレーションする

### 札と ID

札は 48 枚の固定の配列で、`id` (0〜47) の並びを Go 版の `AllCards` と同じにしています。

```swift
enum Month: Int { case january, february, /* ... */ december }
enum CardType: Int { case kasu, tane, tanzaku, hikari }

struct Card {
    let id: Int        // 0〜47。Go 版と同じ並び
    let month: Month
    let type: CardType
}

// Card.all[0] = Card(id: 0, month: .january, type: .hikari)   // 松に鶴
// Card.all[1] = Card(id: 1, month: .january, type: .tanzaku)  // 松に赤短
```

Go 側のテストも、ID の列から札を作る形のまま移したので、ルール上の疑問が出たら、Go 版の実装とテストを正として直しています。

### たつじんの探索

たつじんは、自分からは見えない相手の手札と山札を、枚数の辻褄が合うようにシャッフルし直して 1 つの局面を仮定し、その局面でラウンドの最後までを、ふつうの評価ルールで打ち進めます。

```swift
// 仮定した局面 (相手の手札と山札を配り直したもの)
struct RoundSimulator {
    var game: Game
    var phase: RoundPhase
}

// 探索の木の 1 ノード
final class Node {
    let move: Move?
    var children: [Move: Node]
    var visits: Int           // 通った回数
    var availability: Int     // この手が打てた回数
    var totalReward: Double   // 勝ち負けと文数を 0〜1 にした報酬の合計
}
```

これを 400 回くり返し、通った回数がいちばん多い手を選びます。

### 対局の保存

対局は、盤面のスナップショットではなく、乱数のシードと、両者の全指し手の記録で保存しています。

```swift
struct GameRecord: Codable {
    var rounds: Int              // 3 / 6 / 12
    var difficulty: Difficulty   // easy / normal / hard / search
    var seed: UInt64             // 乱数のシード (配札もここから決まる)
    var moves: [Move]            // 打たれた手 (双方・順番どおり)
}

enum Move: Codable {
    case playHand(handID: Int, fieldChoiceID: Int?)  // 手札を出す
    case chooseDrawnField(fieldID: Int)              // 山札から引いた札の取り先
    case koikoi
    case shobu
}
```

開くときは、シードから同じ配札を作り直し、`moves` を頭から順に適用して、同じ局面まで戻しています。

## フィードバックのお願い

役の判定がおかしい、この操作で落ちた、相手のこの打ち方は変ではないか、といった報告は、とても助かります。

不具合の報告や機能のリクエスト、プルリクエスト、それから「遊んでみました」という一言も歓迎しますので、お気軽にお声がけください。

**https://github.com/ngs/koikoi-swift/issues**

息抜きに、ぜひ一局遊んでみてください 🎴
