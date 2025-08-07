---
title: "【個人利用】Google Workspace Business Standardの登録"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GoogleWorkspace"]
published: false
---

# はじめに
こんにちは。championです。
今回は個人利用でGoogle Workspaceを利用したく、Google Workspace Business Standardを契約したので、契約までに行ったことを記事にしたいと思います。
普段は会社の都合上MSに支配されおり、Google Workspaceは利用していないのですが、前職ではGoogle Workspaceを利用していたことや、
Gemini Advancedや他の生成AI機能が利用できることから個人でもGoogle Workspaceのアカウントが欲しくなり契約することにしました。

# Google Workspaceとは
Google Workspaceとは、文字通りGoogleが提供しているビジネス向けのクラウド型生産性向上ツールです。
企業の生産性向上、共同作業の促進、そしてセキュリティ強化を目的として設計された統合ソリューションであり、
組織の規模や業種を問わず、多様なワークスタイルに対応することができます。

## 契約プランの選定
Google Workspaceにはいくつか契約プランがあります。個人で利用する場合は【Business Starter】か【Business Standard】になるかと思います。
![](/images/google-workspace-business-standard/gws30.png)

私はGoogleドキュメントやスプレッドシート等のアプリケーションでもGeminiを利用してみたかったのとGoogleドライブのデータ保存量の大きさからStandardプランを選びました。
各契約プランごとの詳細な比較については公式の資料を確認してください。
https://workspace.google.co.jp/pricing?hl=ja

## 契約手順
Google Workspaceの公式ページへアクセスし、Google Workspace Business Standardの【無料試用を開始】をクリック
![](/images/google-workspace-business-standard/gws01.png)

【会社名】と【従業員数】、【地域】を聞かれるのでそれぞれ入力して【次へ】をクリック
![](/images/google-workspace-business-standard/gws02.png)

【姓】と【名】、【メールアドレス】を聞かれるのでそれぞれ入力して【次へ】をクリック
![](/images/google-workspace-business-standard/gws03.png)

Google Workspace Business Standardを契約するにはドメインが必要になります。
私はすでにCloudflareでドメインを取得しているため、【既存のドメインを使用して設定する】を選択し、【この方法で続行する】をクリック
ドメイン未取得の場合は、このタイミングでドメインを取得するようにしてください。
![](/images/google-workspace-business-standard/gws04.png)

すでに取得しているドメインを入力して【次へ】をクリック
![](/images/google-workspace-business-standard/gws05.png)

ドメインの確認が求められるため、問題なければ【次へ】をクリック
![](/images/google-workspace-business-standard/gws06.png)

Google Workspaceアカウントを作成していきます。
【ユーザ名】と【パスワード】を聞かれるのでそれぞれ入力して【同意して続行】をクリック
![](/images/google-workspace-business-standard/gws07.png)

おすすめのプランとして【Business Standard】が表示されるので、【14日無料で試す】をクリック
このタイミングで月契約か、年契約か選択することが可能です。私は月契約にしましたが、年契約だと1600円と少し割引された金額で契約することができます。
![](/images/google-workspace-business-standard/gws08.png)

契約するプランと【お支払いプロファイル】の設定をしていきます。
【新しいお支払いプロファイルを作成する】を選択します。
![](/images/google-workspace-business-standard/gws09.png)

お支払いプロファイルとは、Googleサービスでの支払い情報を管理するためのアカウントです。
自身の個人情報の入力が求められるので入力し、【作成】をクリック
![](/images/google-workspace-business-standard/gws10.png)

支払い方法としてクレジットカードが必要なので、クレジットカード情報を入力してください。
![](/images/google-workspace-business-standard/gws11.png)

自身が作成した組織にユーザを追加するかどうか聞かれますが、個人使用のため【今回はスキップ】をクリック
今回スキップしても管理コンソール上からユーザは適宜追加することができます。
![](/images/google-workspace-business-standard/gws12.png)

契約するドメインの所有権の証明とGmailを利用する設定を行う必要があるため、その設定をしてきます。【始める】をクリック
![](/images/google-workspace-business-standard/gws13.png)

私はCloudflareでドメインを取得しているため、【Cloudflareにログイン】をクリック
![](/images/google-workspace-business-standard/gws14.png)

Cloudflareの画面に遷移し、ドメインを管理しているDNSレコードを追加して認証していきます。
TXTレコードの追加を求められるため、【承認】をクリック
![](/images/google-workspace-business-standard/gws15.png)

承認後、Google Workspace側でドメインのDNS設定の確認が行われるため少し待機します。
![](/images/google-workspace-business-standard/gws16.png)

確認が完了すると、ドメインの所有権が証明されます。次にGmailの設定を行うため、【Gmailを有効にする】をクリック
![](/images/google-workspace-business-standard/gws17.png)

Gmailも利用していきたいため、【続行】をクリック
![](/images/google-workspace-business-standard/gws18.png)

個人アカウントでしかGmailは利用しないため、ユーザは自身のみを選択した状態で【有効化を進める】をクリック
![](/images/google-workspace-business-standard/gws19.png)

ドメインの所有権の証明と同様にDNSレコード(MXレコード)の追加を行う必要があるため【Cloudflareにログイン】をクリック
![](/images/google-workspace-business-standard/gws20.png)

Cloudflareの画面に遷移し、ドメインを管理しているDNSレコードを追加して認証していきます。
MXレコードとTXTレコードの追加を求められるため、【承認】をクリック
![](/images/google-workspace-business-standard/gws21.png)

これでドメインの所有権の証明とGmailの利用設定が完了しGoogle Workspaceを利用することができるようになりました。
![](/images/google-workspace-business-standard/gws22.png)

Google Workspaceで作成したアカウントへ初回ログインを行うと、以下が表示されるため【理解しました】をクリック
![](/images/google-workspace-business-standard/gws28.png)

Google Workspaceで作成したアカウントへログインし、管理コンソールへログインすると契約状況を確認することができます。
無事にGoogle Workspace Business Standardで契約することができました。
![](/images/google-workspace-business-standard/gws29.png)

# おわりに
今回はGoogle Workspace Business Standardを契約するまでを記事にしてみました。
会社でもGoogle Workspaceのアカウントを払い出してもらって1ユーザを使うことはあっても、Google Workspace自体の契約や管理者としてアカウントや組織を管理することはなかなかないと思います。そのため、自身でGoogle Workspaceを契約して管理して行くことはいい経験だと思いました。
Google WorkspaceはBusiness Standardでも月額1900円といい値段するので、これから使い倒していこうと思います。
この機会に、Associate Google Workspace Administrator試験も受けてみようかと思います。