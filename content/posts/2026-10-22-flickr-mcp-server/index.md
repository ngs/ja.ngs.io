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

きっかけは、釣りの写真に写っているルアーのサイズから、魚の長さを LLM に計算させたいことです。

iCloud の共有リンクは有効期限があるので、写真の受け渡しには現実的ではありません。

Flickr は Pro を契約しています。

既存の Flickr の MCP 実装には大したものがなさそうなので、Flickr API が提供するフル機能を tool として使えるものを自前で実装しました。

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

画像の配信元の `live.staticflickr.com` は Go の既定の User-Agent に 502 を返すので、リクエストにはこのプログラムの User-Agent を付けています。

## Claude Code での実装

実装は、Slack とつないだ Claude Code のセッションに、モバイルの Slack から頼んでいます。

頼んだのは次のことです。

- Flickr API のフル機能を tool として提供する
- 完成までは private にする
- プルリクエストは最初の 1 本だけにして、あとは main に直接入れる (レビューもしない)
- テストのカバレッジを 98% まで上げる
- README に Homebrew などでのインストール方法を載せる
- 画像の容量は自動でパージせず、使っているローカル容量を応答に入れてパージを促す

Claude Code は最初に設計の指示書を書き、構成を [slack-bridge-mcp-server](https://github.com/ngs/slack-bridge-mcp-server) (Go・公式の MCP Go SDK・GoReleaser・MIT) に揃えたうえで、実装をサブエージェントに任せています。

OAuth の認可は外出先で済ませていて、Claude Code が Slack に貼った認可 URL をモバイルで開いて許可し、表示されたコードを Slack で返すと、Mac 側でトークンに交換されます。

実 API での確認は読み取りだけで行い、カバレッジは 67.8%、89.9%、98.5% と上げ、公開の前に gitleaks で全履歴を検査してから、public 化・v0.1.0 のリリース・Homebrew の cask までを Claude Code が行っています。

## フィードバックのお願い

不具合報告や機能要望、プルリクエストも歓迎しますので、お気軽にお声がけください。

https://github.com/ngs/flickr-mcp-server
