---
title: "noteから、Hugo + Github Pagesに移行した"
date: 2020-12-31T11:52:09+09:00
draft: false
description: ""
tags: []
---

今まで自分のアウトプットをはてなブログ、note、Qiita、dev.toなどに書いてきましたが、

1. コンテンツが分散する
2. コンテンツの流行り廃りで引越しが必要になる
3. なんとなくGithub Pagesでブログ作ってみたい

の3点から移行しました。

以前の記事は以下。noteは転職ブログしか書いてない・・w

* [note](https://note.com/attsun)
* [はてなブログ](http://attsun-1031.hatenablog.com)
* [Qiita](https://qiita.com/__Attsun__)

## ページの構築

このあたりを参考に作りました。ありがとうございます。

* https://yonehub.y10e.com/2019/10/22/20191022_hugo_githubio/
* https://gohugo.io/getting-started/quick-start/

### ホスティング

以下条件で考え、Github Pages一択。まぁ最初から決まってたんですが。他の選択肢は考えてません。

* 無料
* サービス安定性
* 管理しやすさ

### サイトジェネレータ

`jekyll` もちょっとだけ気になりましたが、一定知見があるやつならなんでもいいやということで `Hugo` に。
