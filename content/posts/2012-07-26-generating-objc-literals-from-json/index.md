---
title: JSON から Objective-C のリテラルに変換する
slug: generating-objc-literals-from-json
description: JSON から Objective-C のリテラルに変換する
date: "2012-07-26T00:00:00+09:00"
public: true
tags: ["objective-c","json","javascript"]
alternate: true
archives: ["2012-07"]
---
Xcode 4.4 からNSDictionary, NSArray, NSNumber が簡単にかけるようになったので、ユニットテストのデータなど、コードに直接書こうと思います。
[http://clang.llvm.org/docs/ObjectiveCLiterals.html](http://clang.llvm.org/docs/ObjectiveCLiterals.html)

手で作るのも面倒なので、JSON2.js の stringify メソッドをカスタマイズして、JS オブジェクトから Objective-C リテラルの文字列を返却するようなスクリプトを作りました。

<!--more-->

Node.js からは以下の様に標準入力で JSON を渡すと、標準出力します。



ブラウザからも、`if(process)` の中を削れば Dictionary.stringify メソッドで実行できます。





