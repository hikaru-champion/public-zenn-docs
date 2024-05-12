---
title: "kubernetesのリソースについて整理してみた"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [kubernetes]
published: false
---
# はじめに
kubernetesのリソースについて改めて整理してみます。

# Workloadsカテゴリ
Workloadsカテゴリに所属するリソースは、kubernetesクラスタ上にコンテナを起動させるために利用します。以下、8種類のリソースがあります。
- Pod
- ReplicationController
- ReplicaSet
- Deployment
- DeamonSet
- StatefulSet
- Job
- CronJob

## Pod
Workloadsリソースの最小単位は【Pod】と呼ばれるリソースです。Podは1つ以上のコンテナから構成されており、同じPod内に存在するコンテナ同士はIPアドレスを共有します。そのため、Pod内のコンテナ間の通信にはlocalhostが利用できます。IPアドレスが同じなため、Pod内のコンテナそれぞれはポート番号が重複してはいけません。

## ReplicationController/ReplicaSet
ReplicationController/ReplicaSetは、Podのレプリカを作成し、指定した数のPodを維持し続けるリソースです。以前はReplicationControllerがPodのレプリカを作成するために用いられていましたが、現在はReplicaSetが利用されており、ReplicationControllerは廃止の流れになっています。
ReplicaSetでは、ノードやPodに障害が発生した場合でも、Podの数がReplicaSetで指定した数を満たすように別のノード上でPodを起動してくれます。この機能を「セルフヒーリング」と言います。
また、ReplicaSetで指定するPodの数を増減させることによって、容易にPodをスケーリングさせることもできます。

## Deployment
Deploymentは、複数のReplicaSetを管理することで、ローリングアップデートやロールバックなどを実現するリソースです。Deployment→ReplicaSet→Podと3層の親子構造になっています。Deploymentのローリングアップデートの仕組みとしては以下のフローになります。
1. 新しいReplicaSetを作成
2. 新しいReplicaSet上のレプリカ数（Pod数）を徐々に増やす
3. 古いReplicaSet上のレプリカ数（Pod数）を徐々に減らす
4. （2、3）を繰り返す
5. 古いReplicaSetはレプリカ数0で保持する
Deploymentはkubernetes上でもっとも推奨されているコンテナの起動方法になります。Pod単体でデプロイした場合は、Podに障害があった場合は再作成されませんし、ReplicaSetでデプロイした場合は、ローリングアップデート等の機能を利用することができません。

## DeamonSet
DeamonSetは、各kubenetesノード上で1つずつPodを配置するリソースです。ReplicaSetでは、各kubenetesノード上に合計N個のPodを配置しますが、全てのkubenetesノード上に確実にPodが配置されるとも限りません。DeamonSetでは、レプリカ数は指定できず、1つのノードにPodを複数配置することもできません。ノードが増えた場合は、DeamonSetのPodは自動的にノード上に起動されます。用途としてはログをホスト単位で収集したり、Datadogエージェント等すべてのノード上で必ずどうさせたいプロセスのために利用されます。

## StatefulSet
StatefulSetは、データベースコンテナ等ステートフルなワークロードに対応するためのリソースです。主な特徴として、
- 作成されるPod名のサフィックスは数字のインデックスが付与されたものになる。
    - statefulset-0、statefulset-1
    - Pod名が変わらない
- データを永続化するためにの仕組みがある。
    - PersistentVolumeを使用している場合は、StatefulSetのPodが再起動した際にも同じディスクを利用して再作成される。

## Job
Jobは、コンテナを利用して一度限りの処理を実行させるリソースです。ReplicaSetではPodが停止することは予期せぬエラーですが、Jobの場合は、Podの停止が正常終了として期待される用途に向いています。バッチ処理などで利用されます。一度だけ実行するタスクにも利用できますし、並列実行するタスクにも利用することができます。

## CronJob
CronJobは、CronのようにスケジューリングされたJobを実行したい場合に利用します。CronJobがJobを管理し、JobがPodを管理するような3層の親子関係になっています。定期的なバッチ処理で利用する場合はJobよりもCronJobを利用します。

# Serviceカテゴリ
Serviceカテゴリに所属するリソースは、kubernetesクラスタ上にコンテナにエンドポイントを提供したり、ラベルに一致するコンテナのディスカバリに利用されるリソースです。大きく、L4レベルのロードバランシングを提供するServiceリソースと、L7レベルのロードバランシングを提供するIngressリソースの2種類のリソースがあります。以下、8種類のリソースがあります。
- Service
    - ClusterIP
    - ExternalIP
    - NodePort
    - LoadBalancer
    - Headless
    - ExternalName
    - None-Selector
- Ingress

## ClusterIP
ClusterIPは、kubernetesの最も基本のServiceです。ClusterIPのServiceを作成すると、kubernetesクラスタ内からのみ疎通が可能な内部ネットワークに作り出される仮想IPが割り当てられます。ClusterIPは、kubernetesクラスタ外からのトラフィックを受ける必要がない箇所で、クラスタ内ロードバランサとして機能します。

## ExternalIP
ExternalIPは、特定のKubernetesノードのIPアドレス:Portで受信したトラフィックをコンテナに転送する形で外部疎通性を確立するServiceです。

## NodePort
NodePortは、ExternalIPに類似したServiceです。ExternalIPは、指定したkubernetesノードのIPアドレス:Portで受信したトラフィックをコンテナに転送していましたが、NodePortは、全てのkubernetesノードのIPアドレス:Portで受信したトラフィックをコンテナに転送します。NodePortで利用できるポートの範囲は30000~32767となっており、この範囲外の値を設定しようとするとエラーになってしまいます。

## LoadBalancer
LoadBalancerは、クラスタ外からトラフィックを受ける場合に、一番実用的で使い勝手が良いServiceです。
LoadBalancerは、kubernetesクラスタ外のロードバランサーに外部疎通性のある仮想IPを払い出すことが可能です。Google Cloud/AWS/Azure当のクラウドプロバイダ上で、LoadBalancer Serviceを利用することが主流です。LoadBalancerは、kubernetesノード群とは別に外部のロードバランサを利用するため、ノードの障害に強い特徴があります。ノードに障害があった場合は、そのノードに対してトラフィックが転送しないようにすることで自動的に復旧するような動作をします。

## Headless
Headlessは、対象となる個々のPodのIPアドレスが直接返却されるServiceです。今まで記載したServiceで負荷分散のために提供されるエンドポイントは、仮想IPによるロードバランシングの動作をし、複数Podへ通信を行うためのIPエンドポイントでしたが、headlessはロードバランシングするためのIPアドレスは提供されず、DNSラウンドロビン方式を使ったエンドポイントを提供します。転送先のPodのIPアドレスがクラスタ内DNSから帰ってくる形で負荷分散が行われることが特徴になります。
また、StatefulSetでHeadlessを利用する場合、Pod名によるIPアドレスのディス賀張が可能になります。stetaful-0等のPod名から直接IPアドレスの名前解決ができます。
通常、クラスタ内DNSでPod名を直接指定した名前解決はできない仕様になっています。ClusterIPを作成した場合は、複数Podに対応するClusterIPのエンドポイントが作成され、そのエンドポイントに対しての名前解決は可能ですが、個々のPod名での名前解決はできません。StatefulSetかつ、Headless Serviceを利用している場合のみPod名での名前解決が可能になります。

## ExternalName
ExternalNameは、Service名の名前解決に対して、外部ドメイン宛のCNAMEを返却します。別の名前を指定したい場合や、クラスタ内からのエンドポイントを切り替えやすくしたい場合などに利用します。

## None-Selector
None-Selectorは、Service名で名前解決を行うと自分で指定したメンバーに対してロードバランシングを行うことができます。kubernetesクラスタ内に自由な宛先でロードバランサを作成できる機能です。例えば、Kubernetesクラスタ外のアプリケーションサーバに対してリクエストを分散したい場合に、KubernetesのServiceを使用して行うことができます。ExternalNameでは、CNAMEが返却されますが、None-Selectorでは、ClusterIPのAレコードが返却されます。kubernetesのServiceの使用感のまま、外部向けにロードバランシングを行うことができます。

## Ingress
Ingressは、L7レベルのロードバランシングを提供するリソースです。Ingressには大きく以下の2種類に分類できます。
- クラスタ外のロードバランサを利用したIngress
    - GKE Ingress

GKEを例に説明します。GKEでクラスタ外のロードバランサを利用したIngressの場合、Ingressリソースを作成するだけでGoogle CloudがLoadBalancer用の外部IPを払い出してくれます。
Ingressのトラフィックは、Google Cloud上のLoadBalancerがトラフィックを受信し、HTTPS終端やパスベースのルーティングを行い、NodePortへトラフィックを転送することで対象のPodまで到達します。

- クラスタ内にIngress用のPodをデプロイするIngress
    - Nginx Ingress
Nginx Ingressでは、Ingress用のPodをクラスタ内に作成する必要があります。また、Ingress用のPodに対してクラスタ外からアクセスできるように別途loadBalancer Serviceを作成する必要があります。
Ingress用のPodにNginxを使用した場合、ロードバランサがNginx Podまで転送し、その後NginxがL7レベルの処理を行い、対象のPodへ転送します。この時、Nginx Podから対象のPodまではNode Portを通らず、直接PodのIPアドレス宛に送られます。


# ConfigMap＆Storageカテゴリ
ConfigMap＆Storageカテゴリに所属するリソースは、コンテナに対して設定ファイルや、パスワードなどの機密情報を読み込んだり、永続化ボリュームを提供したりするためのリソースです。以下、3種類のリソースがあります。
- Secret
- ConfigMap
- PersistentVolumeClaim

## Secret
Sercetは、

## ConfigMap
ConfigMapは、設定情報などのKey-Valueで保持できるデータを保存しておくリソースです。nginx.confやhttpd.conf等の設定ファイル自体の保存にも使用できます。
ConfigMapの作成方法としては以下の3つあります。
- kubectlでファイルから値を参照して作成する
- kubectlで直接値を渡して作成する
- マニフェストから作成する

ConfigMapリソースは1つのConfigMapの中に複数のKey-Valueが保存できたり、nginx.conf全体をConfigMapの中に入れもよいです。1つのConfigMapに格納可能なデータサイズは合計1MBになります。

ConfigMapをコンテナから利用する場合は、以下の2つのパターンがあります。
- 環境変数として渡す
- Volumeとしてマウントする

## PersistentVolumeClaim
PersistentVolumeClaimは、永続化領域を利用するためのリソースです。

### Volume
Volumeは、あらかじめ用意された利用可能なボリューム（ホストの領域/NFS/ISCSI/Ceph）などを、マニフェストに直接指定することで利用可能にするものです。

### PersistentVolume
PersistentVolumeは、外部の永続ボリュームを提供するシステムと連携して、新規のボリュームの作成や既存のボリュームの削除を行うことが可能です。

### PersisitentVolumeClaim
PersisitentVolumeClaimは、作成されたPersistentVolumeリソースの中から、アサインするためのリソースになります。PersistentVolumeはkubernetesクラスタにボリュームを登録するだけなので、Podから利用するためにはPersisitentVolumeClaimを定義する必要があります。

# Cluster/Matadataカテゴリ
Clusterカテゴリに所属するリソースは、セキュリティ周りの設定やクォータ設定などクラスタの挙動を制御するためのリソースです。以下、10種類のリソースがあります。
- Node
- Namespace
- PersistentVolume
- ResourceQuota
- ServiceAccount
- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding
- NetworkPolicy

Metadataカテゴリに所属するリソースは、クラスタ上にコンテナを起動させるために利用するリソースです。以下、4種類のリソースがあります。
- LimitRange
- HorizontalPodAutoscaler
- PodDisruptionBudget
- CustomResourceDefinition

# 参考
