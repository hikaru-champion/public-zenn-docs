---
title: "【合格記】Google Cloud Professional Cloud DevOps Engineer"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GoogleCloud"]
published: true
---

# 初めに
こんにちは。hikaruです。
普段、Google CloudでSRE的な業務内容を行なっています。
今回、Google Cloud上でのSREの原理原則や基本用語、立ち回りの理解を確認するために、Google Cloud Professional Cloud DevOps Engineerの認定試験を受験しまして、無事合格することができました。

また、今までは以下のGoogle Cloud認定資格を取得済みです。
- Associate Cloud Engineer
- Professional Cloud Architect
- Professional Cloud Network Engineer
- Professional Cloud Developer

# Professional Cloud DevOps Engineerとは
Professional Cloud DevOps Engineerの[試験ページ](https://cloud.google.com/learn/certification/cloud-devops-engineer?hl=ja)に行くと、以下のように定義されています。


> Professional Cloud DevOps Engineer は、Google 推奨の手法とツールを使用して、システム開発ライフサイクル全体にプロセスを実装します。また、ソフトウェアとインフラストラクチャのデリバリー パイプラインを構築してデプロイし、本番環境のシステムとサービスを最適化して保守し、サービスの信頼性と配信速度の調整を行います。

SREとしての基本的な理解と、Professional Cloud Developerより発展的なアプリケーション開発についてや、コンテナ/CICD/テスト・デプロイ手法/モニタリング/監視/アラート等を理解しているかを試す試験になります。

# 試験範囲
詳細な試験範囲については公式の資料を確認してください。
https://cloud.google.com/learn/certification/guides/cloud-devops-engineer?hl=ja


# どのように対策したのか
## 試験ガイドでの把握
安定の、Google公式の試験ガイドを読み込んで試験範囲を把握しました。
まずは、敵の全体像を知ることが合格の第一歩です。

https://cloud.google.com/learn/certification/guides/cloud-devops-engineer?hl=ja

## テックブログによる情報収集
いつも解説ブログや試験対策マニュアル等でお世話になっているG-genさんの試験対策マニュアルを熟読しました。
試験範囲の中でも重要なポイントを公式ドキュメント付きで解説してくれています。また、G-genさんのサービス別基本機能の解説ブログも基礎の立ち返りができ、試験対策だけでなく、実務で役に立つブログ記事になっているので非常にありがたいです。

https://blog.g-gen.co.jp/entry/professional-cloud-devops-engineer

クラウドエースさんの解説記事も試験範囲の重要なポイントに絞って解説されており、傾向の把握に助かりました。
https://zenn.dev/cloud_ace/articles/cert-devops-engineer-gcp

## SRE本
SREの原則が学べる本と言えば、こちらのSRE本と呼ばれる[SRE サイトリライアビリティエンジニアリング ―Googleの信頼性を支えるエンジニアリングチーム](https://www.oreilly.co.jp/books/9784873117911/)です。
SREを行うにあたっての基本概念
- SLI/SLO
- ポストモーテム
- インシデント対応
- トイル

等を学ぶことができます。前職でSRE本の輪読会を行っていたので、内容は大体把握していましたので軽く見直し程度に読み直しました。このSRE本は、実際のシステム運用を行う上で非常に参考になる概念が記載されている名著なので、これからも定期的に読み直したいと思います。


## 公式模擬試験
Google公式の模擬試験を受けました。
どのような観点の問題が出題されるのかを一通り理解することができます。

## Udemy
Professional Cloud Developerの試験対策と同様にUdemyの問題集を利用し、間違えた問題は公式ドキュメントを確認しました。
また、解説文だけで理解できない点や知識を整理したい機能に関しては、実際に手を動かして検証しアウトプットすることで理解を深めました。
以下の3つの記事がアウトプットした記事になります。

https://zenn.dev/hi_ka_ru/articles/ops-agent-20240610

https://zenn.dev/hi_ka_ru/articles/cloudmonitoring-kind-and-value-20240615

https://zenn.dev/hi_ka_ru/articles/cloudlogging-20240620


# まとめ
無事Professional Cloud DevOps Engineerに合格することができました。
SREの概念や、Cloud RunやGKEのデプロイ方法など普段利用しているサービスの理解度を確認するのに非常に良い試験でした。
次は、Professional Security Engineerを目指して頑張っていきたいと思います。