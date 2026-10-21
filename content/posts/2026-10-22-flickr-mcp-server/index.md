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

釣りの写真を LLM に見せて、写っているルアーのサイズから魚の長さを計算してもらおうとしました。

iCloud の共有リンクを渡してみましたが、リンクに有効期限があるので、この方法は現実的ではありませんでした。

Flickr は Pro を契約していて、API や MCP から写真にアクセスできないかと考えました。

既存の MCP の実装には大したものがなさそうだったので、Flickr API が提供するフル機能を tool として使えるものを自前で実装しました。

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
