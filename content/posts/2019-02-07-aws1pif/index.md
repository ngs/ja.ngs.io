---
title: AWS の credentials.csv を 1Password に取り込む CLI ツール
slug: aws1pif
description: AWS の credentials.csv を 1Password の .1pif ファイルフォーマットとして出力するコマンドラインツールを公開しました。
date: "2019-02-07T21:00:00+09:00"
public: true
tags: ["aws","cli","tool","golang"]
alternate: true
archives: ["2019-02"]
image: import.jpg
---
![](desktop.jpg)

AWS の credentials.csv を 1Password の .1pif ファイルフォーマットとして出力するコマンドラインツールを公開しました。

[ngs/aws1pif](https://github.com/ngs/aws1pif)

<!--more-->

## インストール方法

Homebrew か `go install` コマンドを使ってインストールします。

```sh
brew install ngs/formulae/aws1pif
```

もしくは

```sh
go install github.com/ngs/aws1pif
```

## 利用方法

`aws1pif` は標準出力として取り込んだ CSV を .1pif の JSON フォーマットで標準出力します。

```sh
cat ~/Downloads/credentials.csv | aws1pif > aws.1pif
```

保存された .1pif ファイルをダブルクリックなどで開くと、1Password が起動し、以下のように取り込みの確認ダイアログが出現します。

![](import.jpg)

今の所、複数の Credential を取り込む機能はありません。

