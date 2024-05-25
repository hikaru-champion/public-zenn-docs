---
title: "【合格記】Google Cloud Professional Cloud Developer"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GoogleCloud"]
published: true
---
# 初めに
こんにちは。hikaruです。
業務でGoogle Cloudを触り始めて2年ほど経ちました。
最近、業務でCloud Run/GKE等のサービスを利用することが多かったため、力試しとして
Google Cloud Professional Cloud Developerの認定試験を受験し、無事合格することができました。

また、今までは以下のGoogle Cloud認定資格を取得ずみです。
- Associate Cloud Engineer
- Professional Cloud Architect
- Professional Cloud Network Engineer

# Professional Cloud Developerとは
Professional Cloud Developerの[試験ページ](https://cloud.google.com/learn/certification/cloud-developer?hl=ja
)に行くと、以下のように定義されています。


> Professional Cloud Developer は、Google が推奨するツールとベスト プラクティスを使用して、スケーラブルで可用性の高いアプリケーションを構築します。また、クラウドネイティブ アプリケーション、デベロッパー ツール、マネージド サービス、次世代データベースを利用した経験もあります。さらに、少なくとも 1 つの汎用プログラミング言語に精通し、コードを計測して指標、ログ、トレースを作成します。


Google Cloudの主要開発サービスを利用したアプリケーション開発についてや、コンテナ/データベース/デプロイ手法等を理解しているかを試す試験になります。

# 試験範囲
詳細な試験範囲については公式の資料を確認してください。
https://cloud.google.com/learn/certification/guides/cloud-developer?hl=ja

実際に受験した際の所感を以下にカテゴリごとにまとめます。

## スケーラビリティ、可用性、信頼性に優れたクラウド ネイティブなアプリケーションの設計
- Kubernetes/Google Kubernetes Engineへの理解
    - Deployment/Service/ConfigMap/Statefulsetが何を意味するのかを理解しているのか
    - PodからGoogle Cloud APIへアクセスする際のセキュリティで意識すること（Workload Identity）
    - Pod間の通信方法（NetworkPolicy/mTLS）
    - Podのスケーリング方式（水平スケーリング/垂直スケーリング）
    - IstioやAnthos Service Mesh等のサービスメッシュについて
    - 複数のチームで開発をする際に考慮するべきこと（NamespaceやRBACについて）

- セキュリティへの理解
    - 最小権限の原則を理解したIAMの利用
    - OAuth/JWT/サービスアカウントを利用したGoogle Cloudサービスへの認証
    - Secret ManagerやKMSを利用したアプリケーションの秘匿情報の管理

- データ管理への理解
    - Cloud Storageの保持ポリシーによるコンプライアンス対応
    - Cloud Storageのライフサイクルポリシーによる改廃処理
    - データベースサービス（loud SQL/Firestore/Bigtable/Spanner）のスキーマ設計・パフォーマンス改善


## アプリケーションの構築とテスト
- 開発ツール
    - Cloud ShellやCloud Codeの利用

- ビルド手法
    - Cloud Source RepositoryとCloud Buildを利用したCI/CDについての理解
    - Cloud Buildのビルド構成ファイルの理解
    - Artifact Registoryでのコンテナイメージの脆弱性スキャン
    - Binary Authorizationを利用した信頼のあるコンテナイメージの作成

- テスト手法
    - Cloud Pub/SubやCloud Functionエミュレータを利用したローカル環境での単体テスト
    - Kubernetes上でlocustを利用した負荷テストの実施

## アプリケーションのデプロイ
- デプロイ手法
    - インプレースデプロイ/ローリングアップデート/カナリアデプロイ/Blue-Grennデプロイへの理解

- Cloud Run/App Engine/GKEを利用したトラフィック分割によるテスト・ロールアウト
    - A/Bテスト
    - シャドーテスト
    - カナリアテスト

- APIの公開とバージョン管理
    - Cloud Endpointを利用したAPIの互換性を考慮したデプロイや認証・レート制限

## Google Cloud サービスの統合
- サーバレスサービスからVPCネットワーク中のリソースへのアクセス
    - App Engine/Cloud RunからサーバレスVPCアクセスコネクタを利用したCloud SQL/MemoryStoreへのプライベート接続

- イベント駆動
    - Cloud Pub/SubからCloud RunやCloud Functionの呼び出し

- Compute Engineへの理解
    - プロジェクト/インスタンスメタデータを利用したアプリケーションの起動
    - 起動/停止スクリプトを利用したアプリケーション起動・停止処理
    - Managed Instance Groupを利用したスケーリング


## デプロイされたアプリケーションの管理
- Cloud Profiler
    - 特定処理のCPU/Memory使用率の把握
- Cloud Trace/OpenTelemetry
    - アプリケーションのレイテンシ・ボトルネックの把握
- Cloud Logging
    - JSON形式でのログ出力
    - 標準出力・標準エラー出力を利用したCloud Loggingへの書き込み
    - BigQuery/Cloud PubSub/Cloud Storageへのログ連携
- Cloud Monitoring
    - メトリクス・ログベースアのラートポリシーによるアラート発砲

# どのように対策したのか
## 試験ガイドでの把握
Google公式の試験ガイドを読み込んで試験範囲を把握しました。
どのGoogle Cloudサービスについて問われるのか理解することができます。

https://cloud.google.com/learn/certification/guides/cloud-developer?hl=ja

## G-genさんの試験対策マニュアル
Google Cloudを専門的に扱っているG-genさんの試験対策マニュアルを熟読します。
試験範囲のGoogle Cloudのサービスの中でも、どのような観点について理解しておくべきかを
詳細に記載してくれています。
試験対策マニュアルに記載している公式ドキュメントへのリンクもくまなくチェックしておくことをお勧めします。
いつも解説ブログや試験対策マニュアル等、非常にお世話になっております。

https://blog.g-gen.co.jp/entry/professional-cloud-developer

## 公式模擬試験
Google公式の模擬試験を受けました。
どのような観点の問題が出題されるのかを一通り理解することができます。
https://docs.google.com/forms/d/e/1FAIpQLSc_67KaPnNwQrLZ7kuhw-aubz7gMAwY6DQwRJYcW0qlG-iajA/viewform?hl=ja

## Udemy
以下、2つのUdemyの問題集を利用して問題演習を繰り返し行いました。
私は心配性なので、合計で3周ずつして解説も含めて理解を深めました。

**詳解Google Professional Cloud Developer 模擬試験2024**
https://www.udemy.com/course/google-professional-cloud-developer-2023

**【2024年版】Google Cloud 認定資格 Professional Cloud Developer 問題集**
https://www.udemy.com/course/2023google-cloud-professional-cloud-developer


# まとめ
上記の学習順番で無事Professional Cloud Developerに合格することができました。
Google Cloudでアプリケーション開発を行うときに知っておくべき基本的なことや、ベストプラクティスについて理解を深めることができたと思います。
次はProfessional Security EngineerかProfessional DevOps Engineerを目指して頑張っていきたいと思います。
