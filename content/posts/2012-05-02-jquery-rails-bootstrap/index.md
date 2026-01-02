---
title: jquery-rails で確認プロンプトを Bootstrap のモーダルとして表示する
slug: jquery-rails-bootstrap
description: jquery-rails を使ったプロジェクトでの、window.confirm を Bootstrap のモーダルで表示します。
date: "2012-05-02T00:00:00+09:00"
public: true
tags: ["jquery","rails","coffeescript","bootstrap"]
archives: ["2012-05"]
---
jquery-rails を使ったプロジェクトで、



みたいにすると、デフォルトでは JavaScript の `confirm` で確認され、格好悪いので、`$.rails.fire` メソッドを上書きして Bootstrap のモーダルを出します。

<!--more-->



