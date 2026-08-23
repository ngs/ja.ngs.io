---
title: Slack から常駐 Claude Code セッションと対話する MCP サーバー
slug: "slack-bridge-mcp-server"
description: Claude Code の常駐セッションを Slack のチャンネルに橋渡しする MCP サーバーです。指示の過程がチャンネルに残るため、1 人での開発でもチームメイトが途中からフィードバックできます。
date: "2026-08-24T02:00:00+09:00"
public: true
tags: ["go","oss","mcp","slack","claude-code"]
archives: ["2026-08"]
alternate: true
---

手元で動いている Claude Code の常駐セッションを、Slack のチャンネルに橋渡しする MCP サーバー [slack-bridge-mcp-server](https://github.com/ngs/slack-bridge-mcp-server) を公開しました。

Slack からメッセージを送ると、ローカルの Mac で動いているセッションがそれを受け取り、作業して、返信を返してきます。

<!--more-->

## モチベーション

1 人で LLM に指示を出しながら開発していると、その指示の過程が自分の端末の中に閉じて、他の人からは何も見えません。

成果物のプルリクエストは見えても、「なぜその設計にしたか」「途中でどう方針を変えたか」というやりとりは残らないため、チームメイトがフィードバックできるのは全部終わったあとになります。

実装や調査の指示を Slack のチャンネル上の対話で進めれば、過程がそのままチームに公開され、途中の判断にも口を挟んでもらえます。

もうひとつの動機は課金の境界で、従量課金の API でボットを立てるのではなく、サブスクリプションの範囲で動いているセッションそのものに外から触りたいと考えました。

ソファや外出先から「あの続きをやっておいて」が言えるようになったのは、副産物として気に入っています。

## 使い方

### 常駐ループの指示

セッション側には、次のようなループを指示しておきます (README に全文があります)。

```
You are bridged to my Slack via the slack-bridge MCP server. Run this loop and
do not stop:

1. Call slack_wait.
2. If it returns timed_out, go back to step 1.
3. For each message: do what it asks, then reply with slack_post. ...
```

### メッセージ送信後の挙動

あとは Slack にメッセージを送るだけで、次のように動きます。

- セッションがメッセージを取り込んだ時点で 👀 リアクションがサーバーから自動で付く (待機中なら送った直後、別の作業中なら手が空いた時点)
- 返信に 10 秒以上かかると「⏳ Working… (1m 05s)」がやりとりと同じ場所 (チャンネル直下かスレッド内) に現れ、経過時間つきで更新され、返信と同時に消える
- セッション側が CI 待ちのような長い処理を始めると、`slack_progress` で「⏳ Working… (2m 10s) — waiting for CI」のように何を待っているかも表示される
- 判断が必要なときは `slack_ask` でボタン付きの設問が届き、タップした選択肢がセッションに返る
- 「さっきの議論をまとめて」と頼むと、`slack_history` でチャンネルの全員の発言を読んで要約してくれる

### ホーム以外のチャンネルでの会話

ホームのチャンネル以外でも、アプリを招待したチャンネルで自分がメンションすると会話が開き、場になるのはチャンネル直下ならそのメッセージの下のスレッド、スレッド内ならそのスレッド自身です。

以後そのスレッド内はメンション不要でやりとりでき、受け取ったメッセージにはチャンネルが付いてくるため、セッションはそれを添えて元の場所に返信します。

どのチャンネルでも中継されるのは自分の発言だけなため、他の人がアプリをメンションしてもセッションには届きません。

ホームのチャンネル以外の追い取りはベストエフォートで、1 回の再接続で読み直すスレッドは 20 個まで、新しいメンションの探索は所属チャンネル 20 個の直近 100 件までに限られます。

### 信頼境界の扱い

中継されるのが自分の発言だけとはいえ、Slack のアカウントとログイン済みの端末を使える人はローカルのツールを持つエージェントに指示できるため、このチャンネルへのアクセスはその Mac のターミナルへのアクセスと同じものとして扱い、セッションのツール権限もそれに合わせて絞ります。

### スリープからの復帰

Mac がスリープしても、メッセージは Slack 側に残っているため失われません。

復帰したセッションが最後に読んだ位置から履歴を追い取り、スレッド内の返信も直近 200 件のメッセージから最大 20 スレッドぶんまで回収します。

### リリース初日の実例

実例をひとつ挙げると、この記事で紹介した機能の大半 (進捗表示・ボタン設問・履歴読み取り・スレッド返信の回収) は、初版リリースの当日にモバイルの Slack から指示して実装したものです。

昼過ぎに v0.1.0 を出し、Slack で会話しながら不具合報告と機能要望を投げ続け、同じ日のうちに v0.2.1 まで進みました。

その指示の過程は全部チャンネルに残っています。

## 導入方法

Homebrew で入ります。

```bash
brew tap ngs/tap
brew install slack-bridge-mcp-server
```

Slack アプリの作成からの手順は [docs/setup.md](https://github.com/ngs/slack-bridge-mcp-server/blob/main/docs/setup.md) にまとめてあり、必要なスコープが設定済みの app manifest も同梱しています。

セッションに与えるループの完全版は、Claude Code のスキルとしてそのまま使える形で [examples/attend/SKILL.md](https://github.com/ngs/slack-bridge-mcp-server/blob/main/examples/attend/SKILL.md) に同梱しています。

環境変数を 4 つ設定し、Claude Code の `.mcp.json` に登録すれば準備完了です。

```json
{
  "mcpServers": {
    "slack-bridge": {
      "command": "slack-bridge-mcp-server",
      "args": []
    }
  }
}
```

## Under the hood

Go で、公式の [MCP Go SDK](https://github.com/modelcontextprotocol/go-sdk) を使って実装しています。

サーバーは Claude Code の子プロセスとして stdio で動くため、デーモンもポートも launchd も持たず、セッションが終われば一緒に終わります。

Slack への WebSocket (Socket Mode) は Slack を必要とする最初のツール呼び出しまで張らないため、`.mcp.json` に入れてあっても番をしないセッションは何も接続しません。

取りこぼし対策は再送ではなく追い取りで、セッションへ引き渡す直前のタイムスタンプをカーソルとして永続化し、再接続時に `conversations.history` と `conversations.replies` でカーソル以降を回収します。

`slack_wait` の待ち時間は既定 300 秒・上限 1,500 秒にしてあり、これは Claude Code が MCP ツールを 30 分の無応答で打ち切る実測から逆算した値です。

似た OSS には [claude-slack-bridge](https://github.com/tomeraitz/claude-slack-bridge) があり、こちらは Docker で常駐するデーモンが、質問を投げる `ask_on_slack` と、Slack のメンションごとに `claude -p` を起動する実行の両方を担います。

slack-bridge-mcp-server は起動済みの常駐セッションにそのまま話しかける設計なため、都度プロセスを起こさずデーモンも置かない点と、スリープ耐性をカーソルの追い取りで出す点が違います。

## フィードバックのお願い

不具合報告・機能要望・プルリクエストを歓迎します。

https://github.com/ngs/slack-bridge-mcp-server

Slack から自分のセッションに「あれどうなった？」と聞ける生活は思いのほか快適なので、常駐セッションを飼っている方はぜひ試してみてください 📱
