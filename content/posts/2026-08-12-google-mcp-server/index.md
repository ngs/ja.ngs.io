---
title: Google Workspace を LLM から読み書きする自作 MCP サーバー「google-mcp-server」
slug: "google-mcp-server"
description: Calendar / Drive / Gmail / Sheets / Docs / Slides を Claude などの MCP クライアントから読み書きする Go 製 MCP サーバー google-mcp-server を紹介します。claude.ai の Google コネクタや Google 公式の Sheets MCP API との違いも整理しました。
date: "2026-08-12T09:00:00+09:00"
draft: true
public: true
tags: ["google-mcp-server","mcp","go","google-workspace","claude","oss"]
archives: ["2026-08"]
image: main.jpg
---

Google Calendar / Drive / Gmail / Sheets / Docs / Slides を、Claude などの MCP クライアントから読み書きできる自作の MCP サーバー **[google-mcp-server](https://github.com/ngs/google-mcp-server)** を紹介します。

Go の単一バイナリで、Homebrew からインストールできます。

```bash
brew tap ngs/tap
brew install google-mcp-server
```

<!--more-->

## モチベーション

昨年から自分用に作って使っていたものですが、最近、受託案件で本格的に使う場面がありました。

クライアントと共有する Google スプレッドシートを TODO 管理表にして、人間と LLM の両方が読み書きする運用にしたかったのです。

まず claude.ai の Google Drive コネクタを試しましたが、スプレッドシートの読み取りはできても、**セル単位の書き込みができませんでした**。

Google 公式の [Sheets MCP API](https://developers.google.com/workspace/sheets/api/guides/configure-mcp-server) も検討しましたが、Developer Preview 段階で対象は Sheets のみ、自分の環境ではすぐに使える状態ではありませんでした。

そこで自作の google-mcp-server を実戦投入したところ、診断ログが標準出力に混ざって JSON-RPC ストリームを壊す不具合などをいくつか踏んだので、その場で修正して v0.5.0 までリリースしました。

実案件で使えることを確認できたので、あらためて記事にしています。

## 既存の選択肢との違い

Google Workspace を LLM から触る方法は、大きく 3 つあります。

| | claude.ai の Google コネクタ | Google 公式 Sheets MCP API | google-mcp-server |
|---|---|---|---|
| 対象サービス | Drive / Gmail / Calendar など | Sheets のみ | Calendar / Drive / Gmail / Sheets / Docs / Slides |
| セル単位の書き込み | 不可 (2026-08 時点・自分の環境) | 可 | 可 |
| 提供形態 | claude.ai 組み込み | Google ホストの HTTP + OAuth | ローカル実行 (stdio) の単一バイナリ |
| 提供段階 | 正式提供 | Developer Preview | OSS (MIT) |
| セットアップ | 画面から接続するだけ | GCP で API 有効化 + クライアントごとのリダイレクト URI 設定 | GCP で OAuth クライアント作成 + バイナリ配置 |
| 複数アカウント | アカウントごとに接続 | — | 1 サーバーで複数アカウントを自動選択 |

**claude.ai の Google コネクタ** は設定が一番簡単なため、Drive の検索・ファイルの読み取り・メールやカレンダーの参照だけならこれで足ります。

一方で、スプレッドシートのセル値更新や Slides の操作は提供されていません。

**Google 公式の Sheets MCP API** は `sheetsmcp.googleapis.com` を有効化して HTTP + OAuth 2.0 で接続するマネージドな MCP サーバーで、`get_values` / `update_values` / `update_formulas` / `insert_dimension` などのツールを提供します。

Google がホストするので運用は楽ですが、Developer Preview 段階で、対象が Sheets 単体な点と、MCP クライアントごとにリダイレクト URI の登録が必要な点は注意が必要です。

**google-mcp-server** はローカルで動く stdio の MCP サーバーで、上記 6 サービスをひとつのバイナリで提供します。

自分の GCP プロジェクトで作った OAuth クライアントを使うため、トークンもデータも自分のマシンと Google の API の間だけで完結します。

## 使い方

MCP クライアントに登録すると、次のような依頼が自然言語でできるようになります。

- 「この TODO シートに今日やった作業を追記して、ステータス列を更新して」 (`sheets_values_get` / `sheets_values_update`)
- 「この Markdown の議事録を Google Docs にして Drive の共有フォルダに置いて」 (`drive_markdown_upload`)
- 「来週の空き時間を 3 つ出して」 (`calendar_freebusy_query`)
- 「この Markdown からスライドを作って PDF で書き出して」 (`slides_markdown_create` / `slides_export_pdf`)

複数の Google アカウントを登録でき、依頼文の中のメールアドレスやドメインから **使うアカウントを自動選択** します。

たとえば「仕事用アカウントのカレンダーに登録して」と書けば、該当アカウントのトークンで API を呼びます。

`calendar_events_list_all_accounts` のように、全アカウント横断で一覧するツールも用意しています。

## 導入方法

前提として、自分の Google Cloud プロジェクトと OAuth 2.0 クライアント (デスクトップアプリ) が必要です。

(1) Google Cloud Console でプロジェクトを作り、使うサービスの API (Google Sheets API、Google Drive API など) を有効化します。

(2) 「API とサービス > 認証情報」で OAuth クライアント ID を「デスクトップアプリ」として作成します。

(3) クライアント ID とシークレットを環境変数か `config.json` で渡します。

```bash
export GOOGLE_CLIENT_ID="YOUR_CLIENT_ID.apps.googleusercontent.com"
export GOOGLE_CLIENT_SECRET="YOUR_CLIENT_SECRET"
```

(4) MCP クライアントに登録します。

Claude Code の場合:

```bash
claude mcp add google -- /opt/homebrew/bin/google-mcp-server
```

Claude Desktop の場合は `~/Library/Application Support/Claude/claude_desktop_config.json` に書きます。

```json
{
  "mcpServers": {
    "google": {
      "command": "/opt/homebrew/bin/google-mcp-server"
    }
  }
}
```

初回はブラウザが開いて OAuth の同意画面が出るので、許可すればトークンが保存されて以降は自動更新されます。

アカウントの追加は、クライアントから「Google アカウントを追加して」と頼むと `accounts_add` が認可 URL を返してくれます。

## フィードバックのお願い

実案件で使いながら直している段階なので、不具合報告・機能要望・プルリクエストを歓迎します。

[GitHub リポジトリ](https://github.com/ngs/google-mcp-server) の Issue でお気軽にお声がけください。
