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
