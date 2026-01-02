---
title: Hubot でホットペッパーグルメ検索
slug: hubot-rws
description: Hubot にホットペッパーグルメ検索機能を仕込んでみました。
date: "2014-05-24T09:15:00+09:00"
public: true
tags: ["hotpepper","recruit","api","hubot","coffeescript"]
archives: ["2014-05"]
---
<img src="" srcset=" 2x">

会社の [HipChat] に [Hubot] が常駐して、レビュアーの指名、デプロイ、会議のお知らせ (アジェンダ、Hangout URL) などで活躍しています。

自分は、まだ、いじったことがなかったので、手始めに、使い慣れた [リクルート ウェブサービス] と連携させて、遊んでみました。

[リクルート ウェブサービス]: http://webservice.recruit.co.jp/
[Hubot]: https://hubot.github.com/
[HipChat]: https://hipchat.com/

<!--more-->

```bash
heroku config:set HUBOT_RWS_API_KEY=$(取得した API キー)
```

<!-- similar:ignore:begin -->

<!-- similar:ignore:end -->

