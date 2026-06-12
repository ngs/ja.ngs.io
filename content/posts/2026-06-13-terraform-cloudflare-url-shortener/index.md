---
title: Cloudflare Workers の無料プランで動く URL 短縮サービスを Terraform で管理する
slug: "terraform-cloudflare-url-shortener"
description: Cloudflare Workers の無料プランだけで完結する小さな URL 短縮サービスを構築する Terraform モジュール terraform-cloudflare-url-shortener を公開しました。
date: "2026-06-13T00:00:00+09:00"
public: true
tags: ["terraform","cloudflare","workers","url-shortener","oss"]
archives: ["2026-06"]
image: main.jpg
---

Cloudflare Workers の無料プランだけで動作する小さな URL 短縮サービスをデプロイする
Terraform モジュール [terraform-cloudflare-url-shortener] を公開しました。
短縮リンクは Terraform の map として宣言的に定義され、Worker スクリプトに直接埋め込まれます。
KV namespace もデータベースも、有料機能も一切不要です。

モジュールは [Terraform Registry] に `ngs/url-shortener/cloudflare` として公開しています。

<!--more-->

## モチベーション

廃止された [Google Short Links](https://support.google.com/a/answer/9140741) のような仕組み
—— 自分のドメイン上で、短く覚えやすいスラッグから長い URL へリダイレクトする
`go/` リンク —— が欲しかったのがきっかけです。

この用途に対して、セルフホスト型の URL 短縮サービスの多くはオーバースペックです。
月に数回しか変更されない小さなキーバリューの表を管理するためだけに、
管理画面、データベース、認証機能が付いてきます。「サーバーレス」を謳うものでも、
たいてい Workers KV や D1 データベースを要求し、プロビジョニングすべきステートと
バックアップ対象が増えてしまいます。

しかし、リンクがどのみち Terraform の設定に書かれるのであれば、データストアは不要になります。
リンクの一覧は Worker スクリプトの中に丸ごと収まってしまうので、次のような利点が生まれます。

- **リンクがコードになる。** リンクの追加はプルリクエストです。他のコードと同じように
  レビューされ、バージョン管理され、`git log` から差し戻せます。
- **ランタイム依存ゼロ。** KV の読み取りもデータベースへの問い合わせも発生しません。
  すべてのリクエストはエッジ上のメモリにある定数オブジェクトから応答されます。
- **無料。** Workers スクリプト、ルート、プロキシされた DNS レコードはすべて
  Cloudflare の無料プランで利用できます。

## 仕組み

モジュールが管理するリソースはたった 3 つです。

1. **Workers スクリプト** —— テンプレートからレンダリングされ、`links` map が
   JSON として埋め込まれます。
2. ホスト名に対するプロキシされた**プレースホルダー DNS レコード**(`AAAA 100::`)——
   これによりトラフィックが Cloudflare のエッジに到達します。`100::` は IPv6 の
   discard prefix で、すべてのリクエストを Worker が処理するため、実在する
   オリジンを指す必要がありません。
3. **Workers ルート**(`<hostname>/*`)—— ホスト名へのすべてのリクエストを
   Worker に送ります。

リクエストの処理は意図的にシンプルにしています。

| リクエスト | レスポンス |
|---|---|
| `/<既知のスラッグ>` | 対応する URL へ `redirect_status`(デフォルト `301`)でリダイレクト |
| `/` | `root_url` へ `302`、`root_url` が `null` の場合は `404` |
| それ以外 | `404` |

細かいですが重要な点として、スラッグの検索には `Object.hasOwn` を使っているため、
`constructor` や `__proto__` といったプロトタイプのプロパティ名がリンクとして
解決されることはありません。

リンクの更新は `terraform apply` を実行するだけです。Terraform がスクリプトの
テンプレートを再レンダリングして新しい Worker をアップロードし、数秒で世界中に
反映されます。

## 使い方

```hcl
module "url_shortener" {
  source  = "ngs/url-shortener/cloudflare"
  version = "~> 0.1"

  account_id = var.cloudflare_account_id
  zone_id    = var.cloudflare_zone_id
  hostname   = "s.example.com"

  root_url = "https://example.com/"

  links = {
    gh      = "https://github.com/ngs"
    resume  = "https://example.com/resume.pdf"
    "a.txt" = "https://example.com/files/a.txt"
  }
}
```

設定はこれだけです。これで `https://s.example.com/gh` が GitHub プロフィールへ
リダイレクトされ、ルートパスへのアクセスは `root_url` にフォールバックします。

プロバイダーが使用する Cloudflare API トークンには、対象ゾーンの Workers スクリプト、
Workers ルート、DNS レコードを管理する権限が必要です。モジュールは Terraform >= 1.0 と
[cloudflare プロバイダー](https://registry.terraform.io/providers/cloudflare/cloudflare/latest)
~> 5 を要求します。

いくつかのオプションも用意しています。

- `redirect_status` —— デフォルトの `301` から `302`、`307`、`308` に変更できます
  (バリデーション付きなので、タイプミスは plan の時点で失敗します)。
- `script_name` —— Workers スクリプト名を上書きします。デフォルトはホスト名の
  ドットをダッシュに置き換えたものです。
- `root_url` —— `null` のままにすると、リダイレクトせずルートで `404` を返します。

## 認証情報なしでテストできる

モジュールには 2 つのテストスイートが付属しており、どちらも Cloudflare の
アカウントを必要としません。

```sh
# Terraform ネイティブテスト(モックプロバイダー、plan のみ)
terraform init -backend=false
terraform test

# Worker のユニットテスト(依存なし、Node.js >= 18)
node --test tests/worker.test.mjs
```

Terraform のテストは Terraform 1.7 で追加されたモックプロバイダーのサポートを使い、
plan の時点でリソースの構成を検証します。Worker のロジックは、テストフレームワークの
インストールが不要な素の `node --test` でカバーしています。

ソースコードは MIT ライセンスで [GitHub][terraform-cloudflare-url-shortener] に
公開しています。Issue やプルリクエストをお待ちしています。

[terraform-cloudflare-url-shortener]: https://github.com/ngs/terraform-cloudflare-url-shortener
[Terraform Registry]: https://registry.terraform.io/modules/ngs/url-shortener/cloudflare/latest
