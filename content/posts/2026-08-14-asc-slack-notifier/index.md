---
title: App Store Connect の審査状況を Slack に流す「asc-slack-notifier」
slug: "asc-slack-notifier"
description: App Store Connect の webhook を受けて審査状態やビルドの変化を Slack に通知する Go 製サーバー asc-slack-notifier を紹介します。Cloud Run でも AWS Lambda でも動く 1 バイナリで、App Store Connect API による通知の情報補完もできます。
date: "2026-08-14T09:00:00+09:00"
public: true
tags: ["asc-slack-notifier","app-store-connect","slack","go","ios","oss"]
archives: ["2026-08"]
draft: true
---

App Store Connect の [webhook](https://developer.apple.com/documentation/appstoreconnectapi/webhooks) を受けて、審査状態やビルドの変化を Slack に通知するサーバー **[asc-slack-notifier](https://github.com/ngs/asc-slack-notifier)** を作りました。

<!--more-->

## モチベーション

App Store Connect の審査通知はアカウント個人へのメールで届くため、これまではメールを見た自分がチームの Slack で「届いてましたね」とやりとりを始めていました。

それなら通知そのものを Slack に流し、通知への thread reply をコミュニケーションの起点にすればよいと思いつき、webhook を Slack に中継する受け口を書いたのがこのツールです。

作ってみると webhook の payload はかなり素っ気なく、審査状態の変化イベントに入っているのは **リソースの UUID と新旧の状態だけ** で、どのアプリのどのバージョンが動いたのかは payload からは分かりません。

複数アプリを 1 つのチャンネルに流すとそれでは役に立たないため、App Store Connect API キーを渡しておくと通知前にリソースを引いて、アプリ名・バージョン・ビルド番号と「App Store Connect で開く」ボタンを付ける補完機能を足しました。

## 使い方

webhook を受けると、状態ごとの絵文字付きで Block Kit メッセージを組み立てて Slack に投稿します (`READY_FOR_REVIEW` 📝、`PENDING_APPLE_RELEASE` ⏳、`READY_FOR_DISTRIBUTION` ✅、`REJECTED` ❌ など)。

Apple がまだドキュメント化していないイベントタイプも汎用の key/value 表示で必ず流すため、通知が黙って捨てられることはありません。

Slack への投稿先は Incoming Webhook URL か、bot トークン + チャンネル (`chat.postMessage`) のどちらかを選べます。

App Store Connect API キー (`Developer` か `App Manager` ロールの読み取りだけで十分) を設定すると、審査状態とビルド系の通知に **App / Version / Build** のフィールドが付き、ボタンからアプリの配信ページや TestFlight ページに直接飛べます。

補完はオプションなため、API が落ちていても通知自体は素の内容で届きます。

## 導入方法

いちばん手軽なのはリポジトリを fork して GitHub Actions に任せる方法で、リポジトリ変数 `DEPLOY_TARGET` を `cloudrun` か `lambda` にしてシークレットを設定すれば、master/main への push で自分のインスタンスがデプロイされます。

手順は [docs/DEPLOYMENT.md](https://github.com/ngs/asc-slack-notifier/blob/master/docs/DEPLOYMENT.md) にまとめました。

手動なら Cloud Run はこれだけです。

```sh
gcloud builds submit --tag "$IMAGE"
gcloud run deploy asc-slack-notifier \
  --image "$IMAGE" \
  --region asia-northeast1 \
  --allow-unauthenticated \
  --set-secrets "ASC_WEBHOOK_SECRET=asc-webhook-secret:latest,SLACK_WEBHOOK_URL=slack-webhook-url:latest"
```

デプロイできたら App Store Connect API で webhook を登録します。

```sh
curl -sS -X POST 'https://api.appstoreconnect.apple.com/v1/webhooks' \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "data": {
      "type": "webhooks",
      "attributes": {
        "name": "Slack notifier",
        "url": "https://your-service.example.com/webhook",
        "secret": "your-webhook-secret",
        "enabled": true,
        "eventTypes": [
          "APP_STORE_VERSION_APP_VERSION_STATE_UPDATED",
          "BUILD_UPLOAD_STATE_UPDATED",
          "BUILD_BETA_DETAIL_EXTERNAL_BUILD_STATE_UPDATED"
        ]
      },
      "relationships": {
        "app": { "data": { "type": "apps", "id": "'"$APP_ID"'" } }
      }
    }
  }'
```

webhook の `secret` は Apple が発行してくれるものではなく **自分で決める値** なため、`openssl rand -hex 32` などで作った同じ文字列を、webhook 登録時の `attributes.secret` とサーバー側の `ASC_WEBHOOK_SECRET` の両方に渡します。

ここが食い違うと配信はすべて `401` で弾かれます。

API キーの秘密鍵は `.p8` ファイルのパスを `ASC_API_PRIVATE_KEY_PATH` に渡すか、PEM の中身そのままか base64 エンコードした文字列を `ASC_API_PRIVATE_KEY` に渡すかを選べるため、fastlane の `key_content` と同じくシークレットに文字列しか入れられないプラットフォームでも困りません。

## Under the hood

Go の 1 バイナリで、環境変数ひとつで Cloud Run のような普通の HTTP サーバーとしても、AWS Lambda + API Gateway の背後でも動きます。

受信時は `x-apple-signature` ヘッダーの HMAC-SHA256 署名を生のリクエストボディに対して検証し、比較は constant time で行います。

Slack に届けられなかったときは `502` を返すため、App Store Connect 側の配信履歴に失敗として残り、そこから再送できます。

## フィードバックのお願い

自分のリリース番の道具として作りましたが、iOS アプリをチームで運用しているところならどこでも使えるはずです。

[MIT ライセンス](https://github.com/ngs/asc-slack-notifier/blob/master/LICENSE)で公開しています。

不具合や機能要望は [GitHub の Issue](https://github.com/ngs/asc-slack-notifier/issues) や Pull Request でご連絡ください。
