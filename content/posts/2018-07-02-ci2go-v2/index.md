---
title: CI2Go バージョン 2
slug: ci2go-v2
description: iPhone と iPad 向け CircleCI クライアント CI2Go のメジャーバージョンアップを行いました。
date: "2018-07-02T09:20:00+09:00"
public: true
tags: ["ci2go","circleci","ios"]
alternate: true
archives: ["2018-07"]
image: main.png
---
iPhone と iPad 向け [CircleCI] クライアント  [CI2Go] のメジャーバージョンアップを行いました。

[![](/images/appstore.svg.svg)][AppStore]

バージョン 2 は以下のアップデートを含んでいます。

<!--more-->

## iPhone X に最適化

![](iphonex.png)

iPhone X の 5.8 インチのスクリーンに最適化しました。

## シングルサインオン

![](sso.gif)

API トークンを CircleCI の画面からコピーしてくる代わりに、シングルサインオンを利用するよう、認証フローを刷新しました。

内部で自動的に [個人 API トークン] を作成します。

## ビルド成果物 ブラウザー

![](artifacts.gif)

ビルド成果物 ブラウザー を再構築しました。

全てのファイルのダウンロードを待つことなく、素早く成果物を確認できます。

必要な時にだけ、ファイルのダウンロードを開始し、画面を離れてもバックグラウンドでダウンロードを続けます。

## Apple Watch コンプリケーション

![](watch.png)

Apple Watch [コンプリケーション] の提供を開始しました。

時計の文字盤でビルドのステータスを確認できます。

watchOS API の制約により、リアルタイムな情報の更新は行いません。

## パフォーマンス改善

このバージョンで、全ての実装を一からみなおしました。

オフラインデータの扱いを行わなくなった代わりに、より素早い挙動を行いようになりました。

不具合などありましたら、 [イシュー] を起票いただけると幸いです。

[CI2Go]: https://itunes.apple.com/app/id940028427?mt=8
[AppStore]: https://itunes.apple.com/app/id940028427?mt=8
[CircleCI]: https://circleci.com
[イシュー]: https://github.com/ngs/ci2go/issues/new
[個人 API トークン]: https://circleci.com/docs/2.0/managing-api-tokens/
[コンプリケーション]: https://support.apple.com/ja-jp/ht205536

