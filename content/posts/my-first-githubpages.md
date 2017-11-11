---
title: "初めての GitHubPages でブログを公開"
date: 2017-11-07T11:27:55+09:00
lastmod: 2017-11-07T11:27:55+09:00
draft: true
keywords: []
description: "株式会社ホットソウル従業員募集のプロモーション第1弾として、FC2で細々やってきたブログをHugoでリニューアル。Hugoで作成し、GitHubPages x 独自ドメインで公開するまでの記録。今回はGitHubPages編。"
tags: [ "Hugo", "GitHubPages", "独自ドメイン" ]
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

---

## GitHubPagesでは独自ドメインを利用した場合HTTPSが使えない
よく見ると、GitHubPagesの設定画面に以下のように書いてある。  
```
Enforce HTTPS — Unavailable for your site because you have a custom domain configured (blog.hotsoul.jp)
HTTPS provides a layer of encryption that prevents others from snooping on or tampering with traffic to your site.
When HTTPS is enforced, your site will only be served over HTTPS.
```
これは、`Enforce HTTPS`のチェックボックスに書いてある説明だが、ざっくり言うと、
```
custom domain使っているから、Enforce HTTPSはチェックできないようにするよ。
だって、HTTPSではあなたのサイトは見られなくなるからね。
```
と言うことらしい。  
実際、httpsでアクセスすると、ブラウザ側で不正なサイトと判断されアクセスできない。  
`blog.hotsoul.jp`にアクセスした時に、SSLの証明書に書かれているドメインは`github.io`のままだから、のようだ。

---

## CDNサービスを利用して独自ドメインでのHTTPS利用を可能にする
GitHubPages + 独自ドメインで、HTTPSを実現する方法は、ネット上ではすでにたくさん情報があった。  
中でももっとも情報が多かった、**[CLOUDFLARE](https://www.cloudflare.com)** というサービスを利用する方法を採用した。
もちろん、無料のサービスである。すごい世の中だ！

検討の中で、契約しているレンタルサーバで、ブログ公開用のGitリモートリポジトリを作り、Hugoで生成したページをそこにpushする。
というアイデアもあったのだが、契約しているプランではSSHが利用できないことがわかり、別の方法を選ぶことにした。

---

## やったこと（主にドメイン管理の作業）

1. まずCLOUDFLAREにアカウント登録した
1. 自社のドメイン情報をCLOUDFLAREに検索させる
1. CLOUDFLAREからネームサーバが払い出されるので、契約しているドメイン管理サービスでネームサーバの変更をする
1. HTTPS関連の
