---
title: "初めての CircleCI でブログを build & deploy"
date: 2017-11-21T11:40:19+09:00
lastmod: 2017-11-21T11:40:19+09:00
draft: false
keywords: []
description: "株式会社ホットソウル従業員募集のプロモーション第1弾として、FC2で細々やってきたブログをHugoでリニューアル。Hugoで作成し、GitHubPages x 独自ドメインで公開するまでの記録。今回はCircleCI編。"
tags: [ "Hugo", "GitHubPages", "独自ドメイン", "CircleCI" ]
categories: [ "essay", "ICT" ]
author: "hiroshi"
slug: "My-First-Circleci"
---

---

## ブログアップデートの手順を自動化しよう
今年に入ってから携わったお仕事で初めて触れた文化？の中に、CI(Continuous Integration：継続的インテグレーション)というものもあった。  

> 継続的インテグレーションとは、ソフトウェア開発において、ビルドやテストを頻繁に繰り返し行なうことにより問題を早期に発見し、開発の効率化・省力化や納期の短縮を図る手法。特に、専用のツールを用いてこのプロセスを自動化あるいは半自動化し、効率的に実施する方式。  
【出典】：[IT用語辞典e-Words](http://e-words.jp/w/%E7%B6%99%E7%B6%9A%E7%9A%84%E3%82%A4%E3%83%B3%E3%83%86%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3.html)

Hugo を使ってブログを作り、作成したコンテンツは GitHubPages で公開するという作業を分解すると、最低３段階は必要になる。  

1. Hugo の機能を利用して、新しい記事情報やブログ自体の機能追加を行う。(変更内容は確実な場所に保管する)
1. Hugo の機能で、ブログのコンテンツ(HTMLファイルなど)を生成する。
1. 生成したコンテンツを、公開用の GitHubPages にアップ(push)する。

この文面だけでは伝わりにくいが、行間に隠れた手間が多々あり、毎回これを行うのはナカナカしんどいものがある。  
そこで、CIの考え方を用いてこれらの作業を半自動化しよう、というのが今回の趣旨。  

そして、このCIの実現に必要な**専用のツール**も、無料のサービスとして提供されている、というのがイマドキの凄いところである。

---

## やったこと（主にCircleCIでの作業）
CIを実現するためのツール(サービス？)には色々あるが、無料で手軽に始められそうということで、CircleCIを選択。  

1. まずCircleCIにアカウントを登録。Googleのアカウントでそのまま登録できた。
1. Hugoでのブログ作成用リポジトリ(hotsoul-blog)を対象としたProjectを作成
1. CircleCI2.0用の設定ファイル`.cricleci/config.yml`を作成しhotsoul-blog直下に追加
1. ブログ公開用リポジトリ(hotsoul.github.io)にpushするためのSSHキーを作成＆登録（GitHubに公開鍵、CircleCIに秘密鍵）

---

## 実際に作ったconfig.ymlがこれ

```
version: 2
jobs:
  build:
    working_directory: ~/hotsoul-blog
    docker:
      - image: circleci/golang:latest
    steps:
      - checkout
      - run:
          name: install hugo
          command: go get -v github.com/gohugoio/hugo
      - run:
          name: build hugo
          command: hugo -t hotsoul-blog
      - save_cache:
          key: hotsoul-blog-public-{{ epoch }}
          paths:
            - ~/hotsoul-blog/public

  deploy:
    working_directory: ~/public-repo
    machine: true
    steps:
      - run:
          name: clone
          command: git clone git@github.com:hotsoul/hotsoul.github.io.git
      - run:
          name: clean
          command: |
            cd hotsoul.github.io
            mv CNAME ../
            mv .git ../
            rm -rf ./*
            mv ../CNAME ./
            mv ../.git ./
            ls -la
      - restore_cache:
          keys:
            - hotsoul-blog-public
      - add-ssh-keys:
          fingerprints:
            - "ここに作成したSSH公開鍵のfingerprintを書く"
      - run:
          name: push contents
          command: |
            cd hotsoul.github.io
            mv ~/hotsoul-blog/public/* ./
            git config --global user.name ${CIRCLE_USERNAME}
            git config --global user.email ${GIT_USER_EMAIL}
            git add --all
            git commit -m "push contents [ci skip]"
            git push

workflows:
  version: 2
  build_and_deploy:
    jobs:
      - build
      - deploy:
          requires:
            - build
          filters:
            branches:
              only: master
```

---

## ハマったところ
細かくあげるとキリがない（忘れてしまっている）ので、大きなトピックのみ。  

### 1. build - hugoのインストール
参考にしたサイトの記述をそのまま利用したところなんだかエラーになった。
どうも、hugoのインストールに`go get`コマンドを使用しているところが上手くいかなかったようだった。   
CircleCI2.0では、Dockerの利用が推奨されているっぽいので、CircleCIで用意されているgo言語用DockerImageを指定したら動いた。
```
docker:
  - image: circleci/golang:latest
```

### 2. build - working_directoryの指定
working_directoryは、hugoでnewしたsite名と合わせておかないと、hugoコマンド実行時にエラーになるようだった。
```
build:
  working_directory: ~/hotsoul-blog
```

### 3. deploy - add-ssh-keysの指定
作成したSSHの鍵をGitHubとCircleCIそれぞれに登録したにも関わらず、pushで認証エラー的なのが出続けていた。  
頑張ってCircleCIのドキュメントを見ていると、それらの設定に加えて、  
`add-ssh-keys`の指定が必要とのことだった。  
https://circleci.com/docs/2.0/gh-bb-integration/#adding-readwrite-deployment-keys-to-github-or-bitbucket
```
- add-ssh-keys:
    fingerprints:
      - "ここに作成したSSH公開鍵のfingerprintを書く"
```

---

## ここまでで参考にした情報、活用したサイトについて（謝辞）  
以下の２サイトで、最低限必要な情報はほとんど網羅されていると思う。

- [Hugo で作ったサイトを CircleCI で GitHub Pages に自動デプロイ](https://blog.yuyat.jp/post/auto-deploy-hugo-to-github-pages-with-circleci/)  
- [CircleCI2.0事始め -新しいcircle.ymlとworkflows編- #circleci](https://blog.stormcat.io/post/entry/circleci2.0-overview01/)  

導入としては、こちらの２サイトもとても良かった。  

- [HugoとCircleCIでGitHub PagesにBlogを公開してみた](https://hori-ryota.com/blog/create-blog-with-hugo-and-circleci/)
- [CircleCIで出来るコト](https://qiita.com/noboru_i/items/7d300eb63ae667bf8dc2)

やはり最後は、本家ドキュメント。最終的に分からなければ頑張って読むしかない。  

- [2.0 Documentation](https://circleci.com/docs/2.0/)
