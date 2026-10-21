---
title: Flickr API の全メソッドを tool にした MCP サーバー
slug: "flickr-mcp-server"
description: Flickr API の 224 メソッドをすべて MCP の tool として出す flickr-mcp-server を公開しました。撮影日や場所で自分の写真を探し、EXIF と縮小版を LLM に渡せます。
date: "2026-10-22T09:00:00+09:00"
draft: true
public: true
tags: ["go","oss","mcp","flickr","claude-code"]
archives: ["2026-10"]
alternate: true
# image: main.jpg  # TODO: メイン画像 (992×525) は本人が置く
---

Flickr API のメソッドをすべて MCP の tool として出すサーバー [flickr-mcp-server](https://github.com/ngs/flickr-mcp-server) を公開しました。

Claude Code などの MCP クライアントから、自分の Flickr の写真を撮影日や場所で探し、EXIF を読み、縮小版をダウンロードできます。

<!--more-->

## モチベーション

釣りに行くと、釣った魚や仕掛けの写真をけっこうな枚数撮ります。

その写真を LLM に見せて、撮影時刻から何尾目かを並べたり、写っているルアーの全長を基準に魚の長さを見積もらせたりしたかったのですが、受け渡しの方法で詰まりました。

最初は iCloud の共有リンクを渡していましたが、リンクに有効期限があるため、あとから同じ写真を読み直させることができません。

Flickr Pro を契約しているので、写真は Flickr に上げておき、LLM のほうから撮影日や場所で探して必要な分だけ取りにいく形にしました。

検索とダウンロードに限らず Flickr API の機能をそのまま使えるように、API のメソッドは全部 tool にしています。

## 使い方

### 写真を探す

Claude Code に「10 月 16 日の夜に撮った写真を探して」と頼むと、`flickr_photos_search` が次のような引数で呼ばれます。

```json
{
  "user_id": "me",
  "min_taken_date": "2026-10-16 18:00:00",
  "max_taken_date": "2026-10-16 23:59:59",
  "extras": "date_taken,geo",
  "per_page": "50"
}
```

`user_id` に `me` を渡せるのは OAuth で認可したアカウントで呼んでいるためで、非公開の写真も検索の対象になります。

返ってくるのは Flickr の JSON そのままで、`page` / `pages` / `total` を見て続きのページを取りにいきます。

### EXIF と縮小版

写真が決まったら、`flickr_photos_getExif` で撮影日時やカメラの情報を、`flickr_photos_geo_getLocation` で位置を読みます。

画像は `flickr_download` で、`flickr.photos.getSizes` が返すサイズの中から選んでダウンロードします (既定は `Large`、LLM に見せるだけなら `Medium 800` で足ります)。

ダウンロードした画像は `--download-dir` の下に `<photo id>_<secret>_<size>.<ext>` という名前で置かれ、同じ写真を 2 回目に頼んだときは Flickr に取りにいかずにそのファイルを返します。

古い画像は自動では消さず、`flickr_cache_status` で使用量を見て、`flickr_cache_purge` で消します。

### 権限の上限

起動時の `--max-perms` (`read` / `write` / `delete`) が、そのサーバーが使える権限の上限です。

上限を超える権限が必要なメソッドは tool として登録しないため、`read` で起動すれば、LLM が写真を消したり書き換えたりする tool はそもそも一覧に出てきません。

認可済みのトークンがあるときの tool の数は次のとおりです。

| `--max-perms` | tool の数 |
|---|---|
| `read` | 152 |
| `write` | 215 |
| `delete` | 218 |

トークンが無いときは、ログインの要らない 92 メソッドだけが出ます。

数が多くてクライアントが扱いにくい場合は、`--namespaces photos,photosets,tags` のように絞れます。

アップロードとダウンロードはローカルのファイルを触るため、`--upload-dir` と `--download-dir` でディレクトリを指定したときだけ有効になり、そのディレクトリの外のファイルは読み書きしません。

## 導入方法

Homebrew で入ります。

```sh
brew install ngs/tap/flickr-mcp-server
```

Go 1.26 以降なら `go install` でも入ります。

```sh
go install go.ngs.io/flickr-mcp-server@latest
```

Flickr の [App Garden](https://www.flickr.com/services/apps/create/) で非商用の API キーを作り、キーとシークレットを保存してから、アカウントを認可します。

```sh
flickr-mcp-server setup                 # キーとシークレットを ~/.config/flickr-mcp-server/credentials (0600) に保存
flickr-mcp-server auth --perms read     # 表示された URL で許可し、Flickr に出たコードを貼る
flickr-mcp-server whoami                # 認可できたか確かめる
```

`auth` で頼んだ権限が、そのトークンで使える権限の上限になります。

Claude Code への登録は次の 1 行です。

```sh
claude mcp add flickr -- flickr-mcp-server --download-dir ~/Pictures/flickr
```

なお、無料アカウントの写真は 2025 年 5 月 15 日から原寸と 1024px を超えるサイズのダウンロードが制限されているため ([Flickr Blog](https://blog.flickr.net/en/2025/04/15/service-update-original-large-size-download-limitations-on-free-accounts/))、大きいサイズを取るには Flickr Pro が要ります。

## Under the hood

Go で、公式の [MCP Go SDK](https://github.com/modelcontextprotocol/go-sdk) を使って実装しています。

tool は手で書かず、Flickr の `flickr.reflection.getMethods` と `flickr.reflection.getMethodInfo` から取ったメソッドの一覧 (`api/methods.json`) を `go generate` でコードにしています。

同じ一覧を API ドキュメントのページからも作れるようにしてあり、2026 年 10 月 21 日に取った 2 つは、224 メソッドの名前・必要な権限・引数・説明まで一致しました。

一覧と生成したコードがずれていると CI が落ちるため、Flickr にメソッドが増えたら `snapshot` を取り直して `go generate` するだけで追従できます。

アップロード・差し替え・ダウンロードは REST のメソッドではないため、ここだけ手で書いています。

認証は OAuth 1.0a で、ローカルに HTTP サーバーを立てずに、ブラウザに出たコードを貼る方式にしました。

実際の API で試していて、写真の配信元の `live.staticflickr.com` が Go の既定の User-Agent に 502 を返すことに気づいたため、すべてのリクエストにこのプログラムの名前を入れた User-Agent を付けています。

`go test ./...` は Flickr を HTTP のスタブに差し替えてネットワークに出ずに走り、カバレッジが 98% を下回ると CI が落ちます。

## フィードバックのお願い

不具合報告・機能要望・プルリクエストを歓迎しますので、お気軽にお声がけください。

https://github.com/ngs/flickr-mcp-server
