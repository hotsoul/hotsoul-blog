---
title: "初めての GitHubPages でブログを公開"
date: 2017-11-07T11:27:55+09:00
lastmod: 2017-11-07T11:27:55+09:00
draft: true
keywords: []
description: "株式会社ホットソウル従業員募集のプロモーション第1弾として、FC2で細々やってきたブログをHugoでリニューアル。Hugoで作成し、GitHubPages x 独自ドメインで公開するまでの記録。今回はGitHubPages編。"
tags: [ "hugo", "GitHubPages", "独自ドメイン" ]
categories: [ "essay", "ICT" ]
author: "hiroshi"
slug: "My-First-GitHubPages"
---

---

## GitHubPagesでブログを公開しよう
これまでのエンジニア人生では、GitHubやそれを取り巻く文化には無縁だったのだが、今年に入ってから携わったお仕事で初めてこれらの文化に触れた。  

今の世の中には無料で使えるクラウドサービスなどがたくさんあって、それらを積極的に使っていくと面白いことが比較的手軽に実現できる、ということが身をもって分かったのだった。  

久しぶりにパソコンをいじってワクワクする体験をしたので、リニューアルするブログにもこれで行こう！と思った。

---

## やったこと（主にGitHubでの作業）

1. まずGitHubに自分のアカウントを作った。
1. 自分のアカウントで「hotsoul」というOrganizationを作った。
1. hotsoulのリポジトリとして、Hugoでブログを開発・執筆するためのリポジトリを作った。
1. hotsoulのリポジトリとして、Hugoで生成したHTMLを公開するためのリポジトリ（hotsoul.github.io）を作った。
1. 公開用リポジトリ（hotsoul.github.io）の設定画面で、CNAMEの設定を追加した。

一応時系列に沿って補足すると、事前に契約しているドメイン管理サービスの方で、サブドメイン（blog.hotsoul.jp）の定義を追加している。

---
