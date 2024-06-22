---
title: "Cloud Loggingにログが転送されるまでと、ログビューの動作検証"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GoogleCloud", "CloudLogging"]
published: true
---
# 初めに
こんにちは。hikaruです。
今回は、Cloud LoggingにおいてGoogle Cloud内のログがCloud Loggingに転送されるまでの構造とそのコンポーネントについてまとめます。
普段なんとなくCloud Loggingを利用している方も内部構造を知ることで、よりCloud Loggingを使いこなせるようになると思います。
また、ログバケット内のログへ詳細なアクセス制御を行なうことができる**ログビュー**について検証してみました。


# Cloud Loggingの構造について
Cloud Loggingは、Google Cloudの各サービスの様々なログを集約、保管、分析できるログ管理サービスです。
Cloud Loggingは、Google Cloudの各サービスのログを基本的には自動で収集します。一部例外として**Opsエージェント**を利用してCompute Engine内の任意のログデータを収集することができます。
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
ログシンクは、**包含フィルタ**を利用することで包含フィルタに一致する特定のログのみを任意の宛先に転送します。例えば、アプリケーションのログや、Load BalancerのログのみをBigQueryへ転送したりすることができます。また、**除外フィルタ**を利用することで、除外フィルタに一致する特定のログは任意の宛先に転送しないようにすることができます。
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
プロジェクト作成時に自動的に、**_Requiredバケット**と、**_Defaultバケット**が作成され、これらのバケットにログを転送するログシンクも作成されます。
_Requiredバケットと_Defaultバケットについての詳細は以下にまとめます。

## _Requiredバケット
_Requiredバケットには、以下の種類のログが自動的に転送されます。
また、ログの保持時間は**400日間**で、保持期間の変更及び、_Requiredバケットの変更や削除はできません。
- 管理アクティビティログ
- システムイベントログ
- Google Workspace管理者監査ログ
- Enterpriseグループ監査ログ
- ログインの監査ログ
- アクセスの透明性ログ

_Requiredバケットへ転送するログシンクの包含フィルタを確認すると以下のようになっています。
上記の種類のログを転送するような設定になっていました。
```
# 管理アクティビティログ
LOG_ID("cloudaudit.googleapis.com/activity") OR 
# Google Workspace管理者監査ログ
LOG_ID("externalaudit.googleapis.com/activity") OR 
# システムイベントログ
LOG_ID("cloudaudit.googleapis.com/system_event") OR
# Google Workspace管理者監査ログ 
LOG_ID("externalaudit.googleapis.com/system_event") OR 
# アクセスの透明性ログ
LOG_ID("cloudaudit.googleapis.com/access_transparency") OR
# Google Workspace管理者監査ログ 
LOG_ID("externalaudit.googleapis.com/access_transparency")
```

## _Defaultバケット
_Defaultバケットには、_Requiredバケットに保存されなかったログが転送されます。
具体的には、以下の種類のログです。
ログの保持時間はデフォルト**30日間**で、保持期間の変更は可能、_Requiredバケットの削除はできません。
- データアクセス監査ログ
- ポリシー拒否監査ログ
- プラットフォームログ
- コンポーネントログ
- ユーザ作成のログ
- マルチクラウドログとハイブリットクラウドログ

_Defaultバケットへ転送するログシンクの包含フィルタを確認すると以下のようになっています。
_Requiredバケットの包含フィルタに**NOT**が付与されているため、_Requiredバケット以外のすべてのログを転送するような設定になっていました。
```
# 管理アクティビティログ以外
NOT LOG_ID("cloudaudit.googleapis.com/activity") AND 
# Google Workspace管理者監査ログ以外
NOT LOG_ID("externalaudit.googleapis.com/activity") AND 
# システムイベントログ以外
NOT LOG_ID("cloudaudit.googleapis.com/system_event") AND 
# Google Workspace管理者監査ログ以外
NOT LOG_ID("externalaudit.googleapis.com/system_event") AND 
# アクセスの透明性ログ以外
NOT LOG_ID("cloudaudit.googleapis.com/access_transparency") AND 
# Google Workspace管理者監査ログ以外
NOT LOG_ID("externalaudit.googleapis.com/access_transparency")
```

# 監査ログ
Cloud Loggingには、**誰がいつ何をしたか**を記録する監査ログ (Cloud Audit Logs)が転送されます。監査ログ（データアクセス監査ログ）自体の設定については、Cloud Loggingとは独立したGoogle Cloudサービスとして存在しています。
監査ログを有効にすることで、Google Cloudのデータとシステムをモニタリングして、セキュリティ、監査、コンプライアンス、脆弱性や外部データの不正使用の可能性を確認することができます。
https://cloud.google.com/logging/docs/audit?hl=ja

監査ログの種類は以下の4つがあります。

| 監査ログの種類 | 内容 | 転送先ログバケット | 料金 |
| ---- | ---- | ---- | ---- |
| 管理アクティビティ監査ログ | リソースの構成またはメタデータを変更するAPI 呼び出しやその他の管理アクションに関するログエントリが含まれる | _Requriedバケット | 無料 |
| データアクセス監査ログ | リソースの構成やメタデータを読み取るAPI呼び出しや、ユーザー提供のリソースデータの作成、変更、読み取りを行うユーザー主導のAPI呼び出しが含まれる。<br>**BigQueryデータアクセス監査ログはデフォルト有効**になっているが、**それ以外のGoogle Cloudサービスではデフォルト無効**なので明示的に有効化する必要がある。（有効化によるコスト肥大化には注意）  | _Defaultバケット | 有料 |
| システムイベント監査ログ | リソースの構成を変更するGoogle Cloudアクションのログエントリが含まれる | _Requriedバケット | 無料 |
| ポリシー拒否監査ログ | セキュリティポリシー違反が原因で、Google Cloudサービスがユーザーやサービスアカウントへのアクセスを拒否した場合に記録される | _Defaultバケット | 有料 |

## データアクセス監査ログ
データアクセス監査ログは、「組織」、「フォルダ」、「プロジェクト」、「請求先アカウント」のスコープに対して設定することが可能です。「組織」に設定をすると組織配下の「フォルダ」や「プロジェクト」にも継承され、設定を上書きすることはできません。
また、**デフォルト構成**で上記のスコープの全てのGoogle Cloudリソースに対してデータアクセス監査ログを有効化することもできますし、**サービス個別**にデータアクセス監査ログの設定をすることが可能です。

![](/images/cloudlogging-20240620/cloudauditlogs.png)
*CloudAuditLogsのデータアクセス監査ログの設定*

データアクセス監査ログに記録されるオペレーションのタイプは3つあり、すべてを選択することも可能ですし、どれか一つ選択することも可能です。
- **ADMIN_READ**：メタデータまたは構成情報を読み取るオペレーション
- **DATA_READ**：ユーザー提供データを読み取るオペレーション
- **DATA_WRITE**：ユーザー提供データを書き込むオペレーション

また、**除外されたプリンシパル**を設定することで、特定のプリンシパルはデータアクセス監査ログを記録しないようにすることができます。

# アクセス制御
## IAMでのアクセス制御
IAMでのアクセス制御は、プリンシパル（ユーザ、グループ、サービスアカウント等）にRoleを付与してアクセス制御を行います。
ログエクスプローラで、_Requiredバケットと _Defaultバケットのすべてのログを閲覧したい場合や、監査ログの中でも**管理アクティビティ**、**ポリシー拒否**、**システムイベント**を閲覧したい場合は、**roles/logging.viewer**のIAMロールが必要になりますが、**データアクセス監査ログ**を閲覧したい場合は、**roles/logging.privateLogViewer**のIAMロールが追加で必要になります。
後述するログビューを使用してログを読み取れるようにする場合は、**roles/logging.viewAccessor**のIAMロールが必要になります。また、ログシンク、ログバケット、ログビュー等のロギング構成を作成および変更できるようにするには、**roles/logging.configWriter**のIAMロールが必要になります。


## ログビュー
ログビューは、ログバケットに保存されているログのサブセットのみに対するアクセス権をユーザに付与することができます。

https://cloud.google.com/logging/docs/logs-views?hl=ja

例えば、プロジェクトA、プロジェクトBのログをログ集約プロジェクトのログバケットへ転送しているとします。
ログ集約プロジェクト内のログエクスプローラを利用してログを確認したい場合、以下の要件があったとします。
- プロジェクトAの関係者には、プロジェクトAのログのみを閲覧させたい。他のプロジェクトのログは閲覧させたくない。
- プロジェクトBの関係者には、プロジェクトBのログのみを閲覧させたい。他のプロジェクトのログは閲覧させたくない。

プロジェクトAの関係者に、**roles/logging.viewer**をロールを付与してしまうと、プロジェクトA、プロジェクトBのログ全てが閲覧できてしまいますが、**ログビュー**を利用することで、特定のログのみを閲覧させることができ、このような要件を満たすことができます。

![](/images/cloudlogging-20240620/logview.png)
*ログビュー*

事前に、プロジェクトA、プロジェクトBの集約プロジェクトへ転送するログシンクの設定と、集約プロジェクトのログバケットの作成は完了しているものとします。

### ログビューの作成
集約プロジェクトにて、集約プロジェクトのログバケットに対するログビュー（プロジェクトAのログのみに対応）をgcloudコマンドで作成します。
:::message
現在ログビューの作成は、gcloudコマンドのみでしか対応していません。
:::
**--log-filter**でプロジェクトAのログのみを抽出するようにフィルターしています。
```
$ gcloud logging views create projectA \
 --bucket=each-project-aggregation-bucket \ 
 --location=global \ 
 --log-filter='source("projects/projectA")' \ 
 --description="log view for projectA"
```

### ログビューの確認
ログビューが作成されたかgcloudコマンドで確認してみます。
VIEW_IDが**projectA**のログビューが作成されていることが確認できました。
```
$ gcloud logging views list \ 
 --bucket=each-project-aggregation-bucket \ 
 --location=global

VIEW_ID: _AllLogs
FILTER: 
CREATE_TIME: 
UPDATE_TIME: 

VIEW_ID: projectA
FILTER: source("projects/projectA")
CREATE_TIME: 2024-06-22T01:08:46.427171946Z
UPDATE_TIME: 2024-06-22T01:08:46.427171946Z
```

### ログビューへのアクセス制御
今回は**test-user@example.com**がプロジェクトAのログのみを閲覧でき、それ以外のプロジェクトのログは閲覧できないようにしたいため、作成したログビューに対するアクセス権（ロール：**roles/logging.viewAccessor**）を付与します。
```
$ gcloud logging views add-iam-policy-binding projectA \ 
 --member=user:test-user@example.com \ 
 --role='roles/logging.viewAccessor' \ 
 --bucket=each-project-aggregation-bucket \
 --location=global 

Updated IAM policy for logging view [projects/xxxxxxx/locations/global/buckets/each-project-aggregation-bucket/views/projectA].
bindings:
- members:
  - user:test-user@example.com
  role: roles/logging.viewAccessor
etag: BwYbcFB0PYw=
version: 1
```

ロールが付与されているかどうか確認します。
```
gcloud logging views get-iam-policy projectA \ 
 --bucket=each-project-aggregation-bucket \ 
 --location=global

bindings:
- members:
  - user:test-user@example.com
  role: roles/logging.viewAccessor
etag: BwYbcFB0PYw=
version: 1
```

### ログエクスプローラでの確認
**test-user@example.com**ユーザで集約プロジェクトのログエクスプローラへアクセスします。
集約プロジェクトには、プロジェクトA、プロジェクトBのログを集約していますが、ログのフィールドを見ると**PROJECT_ID**には、プロジェクトAしか表示されていませんでした（プロジェクトIDは隠させていただいています）。
画面上部の**範囲を絞り込む**のところにも、ログバケット（**each-project-aggregation-bucket**）内の作成したログビューのみが範囲の対象になっていることが確認できました。
これで、プロジェクトAのログはプロジェクトAの関係者のみ表示、プロジェクトBのログはプロジェクトBの関係者のみ表示させるといったことができるようになりました。

![](/images/cloudlogging-20240620/logview_explorer.png)
*ログエクスプローラでの確認*

# さいごに
今回は、Cloud LoggingにおいてGoogle Cloud内のログがCloud Loggingに転送されるまでの構造とその各コンポーネントについて解説しました。
また、ログビューを試している記事があまりなかったので実際にハンズオン形式でログビューの動作確認を行っていきました。ログを集約している場合に、閲覧できるログを限定したい等、厳密なアクセス制御が求められる時には役に立つと思うので本記事が参考になれば幸いです。