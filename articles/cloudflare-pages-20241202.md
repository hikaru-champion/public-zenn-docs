---
title: "Cloudflare Pagesでポートフォリオサイトを公開する"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Cloudflare", "astro"]
published: true
---
# はじめに
こんにちは。championです。
普段は、Google CloudやAWSを中心としたクラウドインフラの設計～保守運用を行なっています。

インフラ中心のため、今までJavaScript等のフロントエンド言語はほぼ触ったことがなく、
エンジニアとして危機感を持っていましたので、最近、JavaScriptやTypeScript等のインプットをしています。

今回は、アウトプットの一環として、自分のポートフォリオサイトをJavaScriptのフレームワークである「Astro」を利用して、
Cloudflare Pagesに公開したいと思います。

# Cloudflare Pagesとは
Cloudflare Pagesとは、**サーバーレスのフルスタック開発プラットフォーム**でCloudflareのグローバルネットワーク環境に
デプロイすることができます。Cloudflare Pagesでは、以下の機能がサポートされています。
無料プランでも利用できる機能が多いのは非常にありがたいです。

- Git統合（GitHub or GitLabリポジトリと接続）
- プレビュー環境へのデプロイをサポート
- カスタムドメインのサポート
- デプロイ時のロールバックをサポート

https://www.cloudflare.com/ja-jp/developer-platform/products/pages/

# 早速公開してみる
## サインアップ
自身のCloudflareアカウントにサインアップしてください。

## Cloudflare Pagesの利用
Cloudflare上のダッシュボードから左メニューの「Worker & Pages」をクリックし、「Pages」タブをクリックします。
自身のGitHubリポジトリと統合するために、「Gitに接続」をクリックします。
![](/images/cloudflare-pages-20241202/cloudflare-pages-01.png)

## Git接続
自身のポートフォリオのコードがあるGitHubリポジトリと統合するために、「GitHub」タブをクリックし、
「GitHubに接続」をクリックします。（GitLabとも接続することができます。）
![](/images/cloudflare-pages-20241202/cloudflare-pages-02.png)

Git接続したいリポジトリを選択し、「Install & Authorize」をクリックします。

![](/images/cloudflare-pages-20241202/cloudflare-pages-03.png)

するとCloudflare側でGitHubのデプロイ対象のリポジトリが認識されますので、「セットアップの開始」をクリックします。
![](/images/cloudflare-pages-20241202/cloudflare-pages-04.png)

## ビルド/デプロイの実施
Git接続まで完了したので、ビルドとデプロイの実施をしていきます。

- プロジェクト名
  - デフォルトでは、Git接続したGitHubリポジトリ名が設定されています。
  - デプロイ先のURLで使用されます。

- プロダクションブランチ
  - 本番環境で使用されるブランチ名を指定します。

- フレームワークプリセット
  - 使用しているWebフレームワークを選択します。今回は「**Astro**」を選択します。
- ビルドコマンド
  - 「**npm run build**」と入力します
  - ※デフォルト値
- ビルド出力ディレクトリ
  - 「**/dist**」と入力します
  - ※デフォルト値

ルートディレクトリや、環境変数は今回入力無しで行きます。
「保存してデプロイする」をクリックします。

![](/images/cloudflare-pages-20241202/cloudflare-pages-05.png)

デプロイが実施されます。
ビルド時間としては、大体26sでしたが、実際にサイトに接続できるまでは数分かかりました。
（DNS関連だと思います。）

![](/images/cloudflare-pages-20241202/cloudflare-pages-06.png)

デプロイが正常に完了すると、「Workers & Pages」のホームに「**portfoli-astro**」と管理対象のプロダクトが表示されるようになります。

![](/images/cloudflare-pages-20241202/cloudflare-pages-07.png)


## 動作確認
Cloudflare Pages へのデプロイが完了したため、
ポートフォリオサイトへアクセスして確認してみましょう。
無事インターネットへポートフォリオサイトが公開できています。

https://portfolio-astro-bfz.pages.dev/ 


# 最後に
Cloudflare Pagesを使用して、ポートフォリオサイトをデプロイしてみました。
プロダクトさえあれば、ものの数分でインターネットに公開することができ、無料プランでも利用できるため
個人開発をされる方にはうってつけのサービスだと思いました。

次は、今回デプロイしたポートフォリオサイトへカスタムドメインの設定を行っていきたいと思います。