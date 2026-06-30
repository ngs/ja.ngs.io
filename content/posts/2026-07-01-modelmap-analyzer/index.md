---
title: Modelmap Analyzer アップデート
slug: "modelmap-analyzer"
description: Modelmap Analyzer のアップデートを行いました。SVG 書き出し対応、描画の安定性向上、"選択を中央に表示"、"全体を画面に合わせる" 機能の追加、Fluent 2 準拠。
date: "2026-07-01T05:00:00+09:00"
public: true
tags: ["modelmap","modelmap-analyzer","excel","google-sheets"]
archives: ["2026-07"]
image: main.png
---

2019年の公開に公開した、**Modelmap Analyzer** に、久々のアップデートを行いました。

<!--more-->

## Modelmap Analyzer とは

Excel と Google Sheets で、セルの数式がどのセルを参照しているのかを解析して、依存関係をツリーで見られる Google Sheets, Excel on Office 365 向けのアドインです。

[製品サイト](https://analyzer.modelmap.co/)

このソフトウェアは、2019 年に参画したスタートアップ Modelmap で開発したもので、廃業に伴い、2024年にインスタンスゼロ株式会社に移管し、以降公開・販売を続けています。

参照:
- [参画時ブログ記事](/2019/02/12/modelmap/)
- [移管時リリース記事](https://ins0.jp/news/modelmap-analyzer/)

今回、この Modelmap Analyzer に 2 つの機能アップーデートと、価格改定を行いました。

## SVG 書き出しサポート

これまで依存ツリーの書き出しは **PDF のダウンロードだけ**でしたが、**SVG でも書き出せる**ようにしました。

あわせて描画にも手を入れて、書き出しの安定性を上げています。

大きなツリーでも崩れにくくなりました。

SVG は拡大しても潰れないので、仕様書やレビュー資料に貼ったり、印刷して眺めたりするのに使えます。

## UI の安定性向上

- 依存ツリーの表示を整える **選択を中央に表示** と **全体を画面に合わせる** を追加
- UI を **[Fluent 2](https://fluent2.microsoft.design/)** 準拠に更新
- ダーク・ライトモード切替に対応

## 価格改定

価格を見直して、**月額換算 400円**（年払い）に値下げしました。

## つかいかた・導入

- **Excel / Microsoft 365**: [Microsoft AppSource](https://appsource.microsoft.com/product/office/WA200006896)
- **Google Sheets**: [Google Workspace Marketplace](https://workspace.google.com/marketplace/app/modelmap_analyzer/239836082546)

## フィードバック

使ってみて、不具合や機能要望がありましたら、[サポート](https://analyzer.modelmap.co/support)から ご連絡ください。
