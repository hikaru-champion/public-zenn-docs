---
title: ""
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---
# 初めに
こんにちは。hikaruです。
今回は、Cloud LoggingにおいてGoogle Cloud内のログがCloud Loggingに転送されるまでの構造とそのコンポーネントについてまとめます。
普段なんとなくCloud Loggingを利用している方も内部構造を知ることで、よりCloud Loggingを使いこなせるようになると思います。


# Cloud Loggingの構造について
Cloud Loggingは、Google Cloudの各サービスの様々なログを集約、保管、分析できるログ管理サービスです。
Google Cloudの各サービスのログを基本的には自動で収集します。一部例外として**Opsエージェント**を利用してCompute Engine内の任意のログデータを収集することができます。
https://cloud.google.com/logging/docs/overview?hl=ja

Cloud Loggingでは、**ログエクスプローラ**を利用することで、ログの表示や分析をすることができます。
ログエクスプローラで表示できるログは、**ログバケット**に保存されているログのみになります。

![](/images/cloudlogging-20240620/logexplorer.png)
*ログエクスプローラ*

Cloud Loggingのログバケットにログが転送されるまでは、以下のプロセスを経て転送されます。Cloud Loggingの各コンポーネントについての詳細は後述します。
![](/images/cloudlogging-20240620/cloudlogging_structure.png)
*Cloud Loggingのログバケットにログが転送されるまでのプロセス*

https://cloud.google.com/logging/docs/routing/overview?hl=ja

# ログシンクとログルータ
Google Cloudの各サービスのログエントリは、Cloud Logging APIでまず受信し、**ログルータ**を経由し、ログバケットに転送されます。その際どのログをどこに転送するかは**ログシンク**によって決定されます。
ログルータはログを一時的に保存し、バッファのような役割をします。ログシンクが転送先にログをルーティングするときに発生する可能性がある一時的な中断や停止から保護してくれます。
ログシンクは、**包含フィルタ**を利用することで包含フィルタに一致する特定のログのみを任意の宛先に転送します。例えば、アプリケーションのログや、Load BalancerのログのみをBigQueryへ転送したりすることができます。また、**除外フィルタ**を利用することで、除外フィルタに一致する特定のログは任意の宛先に転送し内容にすることができます。
包含フィルタ、除外フィルタはともに[Loggingクエリ言語](https://cloud.google.com/logging/docs/view/logging-query-language?hl=ja)で記述します。

ログルータで転送できる宛先は以下になります。
ログを分析したいのか、監査のために長期保管するのか、サードパーティのSIEMへ送信したいのか等ユースケースによって転送する宛先を選択する必要があります。
- Cloud Loggingバケット
- BigQueryデータセット
- Cloud Storageバケット
- Pub/Subトピック
- Splunk
- 他のGoogle Cloudプロジェクト

# ログバケット
Cloud Loggingでは、ログルータを経由したログデータは**ログバケット**に保存されます。
プロジェクト作成時に自動的に、**_Requiredバケット**と**_Defaultバケット**が作成され、これらのバケットにログを転送するログシンクも作成されます。
_Requiredバケットと_Defaultバケットについての詳細は以下にまとめます。


# アクセス制御

## IAMでのアクセス制御

## ログビュー

