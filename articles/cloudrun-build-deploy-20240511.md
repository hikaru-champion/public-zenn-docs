---
title: "【Google Cloud】Cloud BuildとCloud Deployを利用したCloud Runをデプロイ"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GoogleCloud", "CloudRun", "CloudBuild", "CloudDeploy"]
published: true
---
# はじめに
こんにちは。とある事業会社でGoogle Cloudを扱っているHikaruです。
Google Cloudの認定資格の一つである「professional cloud developer」の受験を控えているので、試験範囲でもある以下のサービスについて実際に手を動かして理解していきたいと思います。
- Cloud Source Repositories
- Cloud Build
- Cloud Deploy
- Cloud Run

以下の認定資格についてはすでに取得済みです。
- Associate Cloud Engineer
- Professional Cloud Architect
- Professional Cloud Network Engineer

# Cloud Source Repositories
## Cloud Source Repositoriesとは
Cloud Source Repositoriesとは、Google Cloudが提供するプライベートGitリポジトリです。
Google Cloud上でGitを使ったソースコードのバージョン管理を行うことができます。一般的なGitコマンドに対応しているため、普段Githubをお使いの方は、Githubと同じ感覚で利用することができます。
Cloud BuildとのCI/CD連携やApp Engine/Cloud Functionsへの直接デプロイ等Google Cloudの他のサービスとの連携に強みがあります。

## 設定してみる

![](/images/cloudrun-build-deploy-20240511/CloudSourceRepositories01.png)

![](/images/cloudrun-build-deploy-20240511/CloudSourceRepositories02.png)

![](/images/cloudrun-build-deploy-20240511/CloudSourceRepositories03.png)

![](/images/cloudrun-build-deploy-20240511/CloudSourceRepositories04.png)

## 課題感
Cloud Source Repositoriesを使用してみての課題感ですが、Githubと比較して以下が不足しています。
- UIが見ずらい
- プルリクエストに対応していない
- Issue管理機能がない
このため、チームでの共同開発にはCloud Source Repositoriesの利用はおすすめできないと思います。
Githubが利用できるのであれば、Githubを利用した方が良いですね。