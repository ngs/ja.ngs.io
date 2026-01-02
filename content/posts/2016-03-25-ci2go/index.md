---
title: CI2Go v1.3.0 をリリースしました
slug: ci2go
description: "@circleci iOS クライアント CI2Go の v1.3.0 をリリースしました"
date: "2016-03-25T19:15:00+09:00"
public: true
tags: ["circleci","ios","app","ci","pusher","websocket","realm","ci2go"]
alternate: true
app_id: 940028427
archives: ["2016-03"]
---
![](ci2go.gif)

[CI2Go] の v1.3.0 をリリースしました。

[![](/images/appstore.svg.svg)][AppStore]

**[CI2Go on the App Store][AppStore]**

<!--more-->

この更新で、以下の新機能が追加されました:

- キャッシュをクリアして再ビルド
- ビルド成果物を閲覧する
- `circle.yml` を読む

また、_リアルタイムログ表示_ が CircleCI の Web Socket API の更新により使えなくなっていたのを修正しました。

_ビルド一覧_ は、まだリアルタイム更新が使えなくなっていますが、[修正] を行い、次回アップデートで修正される予定です。

[修正]: https://github.com/ngs/ci2go/pull/55
[CI2Go]: https://github.com/ngs/ci2go
[AppStore]: https://itunes.apple.com/app/id940028427?mt=8

