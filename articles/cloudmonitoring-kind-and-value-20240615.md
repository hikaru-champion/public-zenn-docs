---
title: ""
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GoogleCloud", "CloudMonitoring"]
published: false
---
# 値の型
測定値のデータ型を表し、Cloud Monitoringでは、**ValueType**と表記されます。

| 型 | 説明 |
| ---- | ---- |
| BOOL | 真偽値型 |
| INT64 | 64ビット整数型 |
| DOUBLE | 浮動小数点型 |
| STRING | 文字列型 |
| DISTRIBUTION | 分布測定型（値のグループを表す） |


# 指標の種類
各時系列には、そのデータポイントの指標の種類が含まれます。Cloud Monitoringでは、**MetricKind**と表記されます。


| 種類 | 説明 | 用途 |
| ---- | ---- | ---- |
| GAUGE | 特定の時刻を測定するための指標 | CPU使用率 |
| DELTA | 時間間隔の変化を測定するための指標 | リクエスト数 |
| CULUMATIVE | 累積指標を測定するための指標 | ネットワーク送信・受信バイト数 |

# リソースグループ
Cloud Monitoringでは、各リソースをグループに所属させる形で定義することができます。
グループを作成することで、そのグループごとに**アラートポリシー**や、**グラフ**、**ダッシュボード**を設定することができます。

## メンバー条件
グループに所属させるための条件を定義することができます。メンバー条件には以下の条件を利用することができます。
- ラベル
- リージョン
- アプリケーション
- etc...
メンバー条件に一致するリソースは自動的に、一致するグループに追加されるため、変化する環境のモニタリングに役立ちます。

## グループの作成
実際にリソースグループを作成してみます。
CloudMonitoringのコンソール画面から**グループ**を選択します。
**CREATE GROUP**を選択します。
![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-group01.png)

グループを作成するにあたり必要なメンバー条件を入力します。
東京リージョンに存在するすべてのGCEをグループに所属させたいので、以下のメンバー条件を指定します。

- Name : vm-instance-asia-northeast1
- Criteria : Resource Type equals gce_instance
- Criteria : Region equals asia-northeast1
- Combine criteria operator : AND

**CREATE**ボタンをクリックすると、グループが作成されます。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-group02.png)

作成されたグループに、既に東京リージョンに作成済みの**test-instance**がVM Instancesに記載されていることがわかります。
また、CPU使用率や、受信バイト数などのメトリクスも閲覧できています。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-group03.png)

追加で、東京リージョンにVM名：**instance-20240615-000613**を構築してみます。
グループのメンバー条件である
- Resource Type equals gce_instance
- Region equals asia-northeast1
と一致することから、GCEインスタンスを作成後、しばらくするとグループに**instance-20240615-000613**が追加されていることが確認できました。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-group04.png)

# 複数プロジェクトの指標を表示する
複数プロジェクトの指標を集約して1つのモニタリング専用プロジェクトで閲覧することができます。
Google Cloudの公式ドキュメントでは、モニタリング専用プロジェクトを作成し、モニタリング専用プロジェクトには、その他のリソースは作成しないことが推奨されています。
モニタリング専用プロジェクトで、指標を閲覧するためには、Monitoring閲覧者（**roles/monitoring.viewer**）ロールが必要です。

## リソースコンテナ
Google Cloudのモニタリング対象プロジェクトのことを言います。

## 指標スコープ
プロジェクトでグラフ化およびモニタリングできる時系列データを持つセットのリソースコンテナ（モニタリング対象プロジェクト）を定義します。デフォルトでは、Google Cloudプロジェクトの指標スコープには自身のプロジェクトのみが含まれます。

## スコーピングプロジェクト
複数プロジェクトを集約して閲覧することのできるプロジェクトのことを**スコーピングプロジェクト**と言います。ハブのような役割を果たします。

# 複数プロジェクトの指標を表示してみる。
モニタリング専用プロジェクトを用意し、CloudMonitoringの画面から**Monitoringの設定**をクリックします。
コンソール下部の**GCPプロジェクトの追加**をクリックします。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-scoping01.png)

**プロジェクトを選択**をクリックして、モニタリング対象プロジェクトを選択していきます。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-scoping02.png)

選択後、**ADD PROJECTS**をクリックします。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-scoping03.png)

以上で、複数プロジェクトの指標を表示する設定が完了しました。
モニタリング専用プロジェクト（いわゆるスコーピングプロジェクト）は、プロジェクトの役割に**Scoping Project**と表示され、モニタリング対象プロジェクト（いわゆるリソースコンテナ）は、プロジェクトの役割に**Monitored Project**と表示されています。
その後は、Metrics Explorerから閲覧したい指標を指定することで、モニタリング対象プロジェクトの指標を閲覧することができます。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-scoping04.png)

# 稼働時間チェック
HTTP、HTTPS、TCP のリクエストに応答するアプリケーションに対して定期的にリクエストを送信することができます。稼働時間チェックでは、**パブリックエンドポイント（公開稼働時間チェック）**または**プライベートエンドポイント（非公開稼働時間チェック）**に対してリクエストを送信することができ、レスポンスデータを検証することもできます。
稼働時間チェックには、**稼働時間チェックのステータス**や、**レイテンシ**等の詳細なデータを確認することができます。
また、何らかの理由で稼働時間チェックが失敗した場合は、アラートポリシーを設定することでslackやE-mailに失敗を通知することも可能です。

## 公開稼働時間チェック
**公開稼働時間チェック**は、一般公開されているURLまたは、Google Cloudリソースにリクエストを送信し、正常なレスポンスが返却されるかどうかを確認する機能になります。
以下の公開稼働時間チェックが可能です。
- 稼働時間チェックURL
- VMインスタンス
- App Engineアプリケーション
- Kubernetesサービス
- AWS EC2インスタンス
- AWS ELB
- Cloud Runリビジョン

### 公開稼働時間チェックの作成
公開稼働時間チェックを作成するにあたって、監視対象のリソース（ロードバランサの背後にインスタンスグループに所属しているNginx VM）を用意しました。

![](/images/cloudmonitoring-kind-and-value-20240615/public-uptime-check.png)

今回は、こちらのリソースのエンドポイントに向けて公開稼働時間チェックを用意していこうと思います。
VMインスタンスに公開稼働時間チェックを送信する場合は、稼働時間チェックサーバーによって使用されるIPアドレスを許可する必要があるので、IPアドレスの一覧を許可してください。
https://cloud.google.com/monitoring/uptime-checks/using-uptime-checks?hl=ja




Cloud Monitoringのコンソール画面から稼働時間チェックに遷移し、**稼働時間チェックを作成**をクリックします。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic01.png)

**ターゲット**の項目から必要情報を入力します
今回は、ドメイン・証明書の用意はしていないのでHTTP、IPアドレスベースでの公開稼働時間チェックとします
本番で利用される場合はHTTPS/ホスト名での公開稼働時間チェックが推奨です。
- Protocol : HTTP
- リソースの種類 : URL
- Hostname : 払い出されたロードバランサのIPアドレス
- Path : /
- Check Frequency : 1minute
- 他のオプション : デフォルト


![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic02.png)

**レスポンスの検証**の項目から必要情報を入力します
今回は、デフォルト値をそのまま利用するためスキップします。
ステータスコードや、タイムアウト値をカスタマイズしたい場合は、設定してください。
https://cloud.google.com/monitoring/uptime-checks/response-validation?hl=ja

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic03.png)

**アラートと通知**の項目から必要情報を入力します
今回は、自身のメールアドレス宛にアラートを送信したいので設定していきます。
- Name : 任意のアラートポリシー名
- Duration : 1 minute
- Notifications : 自身のメールアドレス

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic04.png)

**確認**の項目から必要情報を入力します
稼働時間チェックを作成する前に、テストを実施することができます。
今回はステータスコード:200が返却されればOKなので、テストが成功しています。また、レイテンシも表示されています。
- Title : nginx-uptime-check


![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic05.png)

作成された稼働時間チェックを確認すると、以下の情報を確認することができます。
- Percent Uptime
- Uptime Latency
- Passed Checks
- Uptime Check Latency
- Current Status
- Configuration
- Alert Policies
 
![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic06.png)

Nginxのプロセスを停止してみて、稼働時間チェックをエラーにしてみました。
稼働時間チェックの詳細を確認すると、Current Statusはすべて赤い表示に代わっており、Passed Checksも1から0に代わっています。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic08.png)

また、Nginxのプロセスを停止してしばらくすると自身のメールアドレスに対して、アラート通知が送られてくることも確認できました。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic07.png)

Nginxのプロセスを復旧させて、しばらくすると自身のメールアドレスに対して、復旧のアラート通知が送られてくることも確認できました。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic09.png)


## 非公開稼働時間チェック
**非公開稼働時間チェック**は、Google Cloudリソースの内部 IP アドレスに対してリクエストを送信することができます。非公開稼働時間チェックでは、仮想マシン（VM）や L4 内部ロードバランサ（ILB）などのリソースにプライベートネットワーク経由でリクエストを送信するできます。
非公開稼働時間チェックを使用するには、Service Directory を利用してプライベートネットワークアクセスを構成する必要があります。

### 非公開稼働時間チェックの作成
非公開稼働時間チェックを作成するにあたっては、監視対象のリソースとして公開稼働時間チェックで作成したNginx VMをそのまま利用します。

![](/images/cloudmonitoring-kind-and-value-20240615/private-uptime-check.png)

Nginx VMのプライベートIPアドレスエンドポイントに向けて非公開稼働時間チェックを用意していこうと思います。

非公開稼働時間チェックを作成するためには事前に以下のAPIを有効化する必要があります。
- Cloud Monitoring API: monitoring.googleapis.com
- Service Directory API: servicedirectory.googleapis.com
- Service Networking API: servicenetworking.googleapis.com
- Compute Engine API: compute.googleapis.com


**ターゲット**の項目から必要情報を入力します
- Protocol : HTTP
- リソースの種類 : Internal IP

プライベート稼働時間チェックの注釈の**VIEW**をクリックすると、右側にプライベート稼働時間チェックの前提条件のリストが表示されますので、指示に従って設定していきます。
- サービスアカウントの作成
    - Cloud Monitoringが所有するサービスアカウントを使用して、Service Directory サービスとのやり取りする必要があります。
- サービスの登録
    - Service Directoryを利用して、VMインスタンスの内部IPアドレス、ポートに対応するエンドポイント、名前空間を用意する必要があります。
- ファイアウォールルールの作成
    - 35.199.192.0/19 からの受信 TCP トラフィックを有効にするファイアウォール ルールを作成する必要があります。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic10.png)

必要情報を入力し、**レスポンスの検証**、**アラートと通知**、**確認**はデフォルト値を入力し作成します。

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic12.png)


公開稼働時間チェックと同様に、以下の情報を確認することができます。
- Percent Uptime
- Uptime Latency
- Passed Checks
- Uptime Check Latency
- Current Status
- Configuration
- Alert Policies

![](/images/cloudmonitoring-kind-and-value-20240615/monitoring-synthetic13.png)

