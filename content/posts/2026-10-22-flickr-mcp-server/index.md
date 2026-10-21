---
title: Flickr API の全メソッドを tool にした MCP サーバー
slug: "flickr-mcp-server"
description: Flickr API の全メソッドを MCP の tool として使える flickr-mcp-server をリリースしました。
date: "2026-10-22T03:00:00+09:00"
public: true
tags: ["go","oss","mcp","flickr","claude-code"]
archives: ["2026-10"]
alternate: true
# image: main.jpg  # TODO: メイン画像 (992×525) は本人が置く
---

Flickr API の全メソッドを MCP の tool として使えるようにした [flickr-mcp-server](https://github.com/ngs/flickr-mcp-server) をリリースしました。

<!--more-->

## モチベーション

釣りに行くと、釣った魚やルアーの写真をそれなりの枚数撮ります。

これを LLM に見せて、撮影時刻から何尾目かを並べたり、写っているルアーの全長から魚の長さを見積もってもらったりしたかったのですが、写真の渡し方で困っていました。

iCloud の共有リンクを渡してみたものの、リンクに有効期限があるので、あとから同じ写真を読み直してもらうことができません。

Flickr Pro は契約しているので、写真は Flickr に上げておき、LLM の側から撮影日や場所で探してもらうことにしました。

それで、どうせなら Flickr API が提供する機能を全部使えるようにしようと思い、全メソッドを tool にした MCP サーバーを書きました。

## 使い方

`flickr.photos.search` なら `flickr_photos_search` のように、Flickr API のメソッド名のドットをアンダースコアに置き換えたものが tool の名前です。

引数も API のものをそのまま渡し、Flickr の JSON がそのまま返ってきます。

Claude Code には次のように登録します。

```sh
claude mcp add flickr -- flickr-mcp-server --download-dir ~/Pictures/flickr
```

`--max-perms` を指定しなければ `read` で起動するので、写真の削除や編集をする tool は登録されません。

`--download-dir` を指定すると `flickr_download` が使えるようになり、`Medium 800` など `flickr.photos.getSizes` が返すサイズを選んで、そのディレクトリの下に保存します。

## 導入方法

Homebrew で入ります。

```sh
brew install ngs/tap/flickr-mcp-server
```

Go 1.26 以降なら `go install` でも入ります。

```sh
go install go.ngs.io/flickr-mcp-server@latest
```

Flickr の [App Garden](https://www.flickr.com/services/apps/create/) で非商用の API キーを作ってから、次を実行します。

```sh
flickr-mcp-server setup             # API キーとシークレットを保存
flickr-mcp-server auth --perms read # ブラウザで許可して、表示されたコードを貼る
flickr-mcp-server whoami            # 確認
```

## Under the hood

tool は手で書かず、Flickr の reflection API (`flickr.reflection.getMethods` と `flickr.reflection.getMethodInfo`) で取ったメソッドの一覧から `go generate` で作っています。

2026 年 10 月 21 日時点で 224 メソッドあり、読み取りの権限で認可したトークンだと 152 個の tool が出ます。

ひとつ引っかかったのは、画像の配信元の `live.staticflickr.com` が Go の既定の User-Agent だと 502 を返すことで、User-Agent を付けるようにして回避しています。

## フィードバックのお願い

不具合報告や機能要望、プルリクエストも歓迎しますので、お気軽にお声がけください。

https://github.com/ngs/flickr-mcp-server
