---
title: "kubernetesのWorkloadリソースについて整理してみた"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [kubernetes]
published: true
---
# はじめに
一年前まではkubernetes（主にGKE）の構築・運用を行っていましたが、ここ一年くらい全く触らなくなってしまったので、復習もかねて再入門したいと思います。
今回は、kubernetesの基礎にあたるWorkloadリソースについて整理します。

## 動作環境
本記事の検証環境は以下の通りです。
- クラウド：Google Cloud
- サービス：GKE
- GKEバージョン：1.27.11-gke.1062001


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

各Workloadカテゴリに分類されるリソースの階層関係
![](/images/kubernetes-workload-20240512/workload.drawio.png)

## Pod
Workloadsリソースの最小単位は**Pod**と呼ばれるリソースです。Podは1つ以上のコンテナから構成されており、同じPod内に存在するコンテナ同士はIPアドレスを共有します。そのため、Pod内のコンテナ間の通信にはlocalhostが利用できます。IPアドレスが同じなため、Pod内のコンテナそれぞれはポート番号が重複してはいけません。

![](/images/kubernetes-workload-20240512/Pod.drawio.png)

Podを作成するmanifestファイル
```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod
spec:
  containers:
  - name: nginx-container
    image: nginx:1.26
    ports:
    - containerPort: 80
```

Podを作成するmanifestファイルの適用
```
$ vi pod.yaml 

$ kubectl apply -f pod.yaml 
pod/pod created

$ kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          23s
```


## ReplicationController/ReplicaSet
ReplicationController/ReplicaSetは、Podのレプリカを作成し、指定した数のPodを維持し続けるリソースです。以前は**ReplicationController**がPodのレプリカを作成するために用いられていましたが、現在は**ReplicaSet**が利用されており、ReplicationControllerは廃止の流れになっています。
ReplicaSetでは、ノードやPodに障害が発生した場合でも、Podの数がReplicaSetで指定した数を満たすように別のノード上でPodを起動してくれます。この機能を**セルフヒーリング**と言います。
また、ReplicaSetで指定するPodの数を増減させることによって、容易にPodをスケーリングさせることもできます。

![](/images/kubernetes-workload-20240512/ReplicaSet.drawio.png)


ReplicaSetを作成するmanifestファイル
```replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.26
        ports:
        - containerPort: 80
```
ReplicaSetは特定のラベルがつけられたPodの数をカウントすることでPodの数を調整しています。
spec.template.metadata.labelsの部分に対応する「app:nginx」がPodに付与されるラベルで、spec.selector.matchLabelsの部分に対応する「app:nginx」でReplicaSetで制御するPodを指定しています。


ReplicaSetを作成するmanifestファイルの適用
```
$ vi replicaset.yaml

$ kubectl apply -f replicaset.yaml 
replicaset.apps/replicaset created

$ kubectl get replicasets
NAME         DESIRED   CURRENT   READY   AGE
replicaset   3         3         3       19s

$ kubectl get replicasets -o wide
NAME         DESIRED   CURRENT   READY   AGE   CONTAINERS        IMAGES       SELECTOR
replicaset   3         3         3       27s   nginx-container   nginx:1.26   app=nginx

$ kubectl get pods -l app=nginx -o wide
NAME               READY   STATUS    RESTARTS   AGE   IP             NODE                                      NOMINATED NODE   READINESS GATES
replicaset-fxp5p   1/1     Running   0          50s   192.168.1.20   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
replicaset-krf5z   1/1     Running   0          50s   192.168.1.19   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
replicaset-pvtdd   1/1     Running   0          50s   192.168.1.18   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
```

セルフヒーリングの確認
Pod名が「replicaset-fxp5p」のPodを1つ手動で削除してみます。
削除後すぐにPodを確認すると、ReplicaSetによって別名のPodが再作成されていることが確認できました。
このセルフヒーリングの機能によってPod数が指定の数に保たれます。

```
$ kubectl get pods
NAME               READY   STATUS    RESTARTS   AGE
replicaset-fxp5p   1/1     Running   0          7m43s
replicaset-krf5z   1/1     Running   0          7m43s
replicaset-pvtdd   1/1     Running   0          7m43s

$ kubectl delete pods replicaset-fxp5p
pod "replicaset-fxp5p" deleted

$ kubectl get pods -l app=nginx -o wide
NAME               READY   STATUS    RESTARTS   AGE     IP             NODE                                      NOMINATED NODE   READINESS GATES
replicaset-8p4l4   1/1     Running   0          4s      192.168.1.21   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
replicaset-krf5z   1/1     Running   0          7m58s   192.168.1.19   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
replicaset-pvtdd   1/1     Running   0          7m58s   192.168.1.18   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
```


## Deployment
**Deployment**は、複数のReplicaSetを管理することで、ローリングアップデートやロールバックなどを実現するリソースです。Deployment→ReplicaSet→Podと3層の親子構造になっています。Deploymentのローリングアップデートの仕組みとしては以下のフローになります。
1. 新しいReplicaSetを作成
2. 新しいReplicaSet上のレプリカ数（Pod数）を徐々に増やす
3. 古いReplicaSet上のレプリカ数（Pod数）を徐々に減らす
4. （2、3）を繰り返す
5. 古いReplicaSetはレプリカ数0で保持する

Deploymentはkubernetes上でもっとも推奨されているコンテナの起動方法になります。Pod単体でデプロイした場合は、Podに障害があった場合は再作成されませんし、ReplicaSetでデプロイした場合は、ローリングアップデート等の機能を利用することができません。

![](/images/kubernetes-workload-20240512/Deployment.drawio.png)

Deploymentを作成するmanifestファイル
```deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.26
        ports:
        - containerPort: 80
```

Deploymentを作成するmanifestファイルの適用
```
$ vi deployment.yaml

$ kubectl apply -f deployment.yaml 
deployment.apps/deployment created

$ kubectl get deployment -o wide
NAME         READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS        IMAGES       SELECTOR
deployment   3/3     3            3           74s   nginx-container   nginx:1.26   app=nginx

$ kubectl get replicaset -o wide
NAME                   DESIRED   CURRENT   READY   AGE   CONTAINERS        IMAGES       SELECTOR
deployment-5b4699b5f   3         3         3       83s   nginx-container   nginx:1.26   app=nginx,pod-template-hash=5b4699b5f

$ kubectl get pod -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP             NODE                                      NOMINATED NODE   READINESS GATES
deployment-5b4699b5f-dn754   1/1     Running   0          89s   192.168.1.22   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
deployment-5b4699b5f-vw88s   1/1     Running   0          89s   192.168.1.23   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
deployment-5b4699b5f-z5lgq   1/1     Running   0          89s   192.168.1.24   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
```

## DaemonSet
DaemonSetは、各kubenetesノード上で1つずつPodを配置するリソースです。ReplicaSetでは、各kubenetesノード上に合計N個のPodを配置しますが、全てのkubenetesノード上に確実にPodが配置されるとも限りません。DaemonSetでは、レプリカ数は指定できず、1つのノードにPodを複数配置することもできません。ノードが増えた場合は、DaemonSetのPodは自動的にノード上に起動されます。用途としてはログをホスト単位で収集したり、Datadogエージェント等すべてのノード上で必ずどうさせたいプロセスのために利用されます。

![](/images/kubernetes-workload-20240512/Daemonset.drawio.png)



DeamonSetを作成するmanifestファイル
```deamonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.26
        ports:
        - containerPort: 80
```

DeamonSetを作成するmanifestファイルの適用
```
$ vi daemonset.yaml 

$ kubectl apply -f daemonset.yaml 
daemonset.apps/daemonset created

$ kubectl get pod -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP             NODE                                      NOMINATED NODE   READINESS GATES
daemonset-x427q   1/1     Running   0          12s   192.168.1.26   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
```
deploymentやreplicasetと異なり、レプリカ数を特に指定していませんが、ノードに対して1つPodが立ち上がるような動作になります。


## StatefulSet
StatefulSetは、データベースコンテナ等ステートフルなワークロードに対応するためのリソースです。主な特徴として、
- 作成されるPod名のサフィックスは数字のインデックスが付与されたものになる。
    - statefulset-0、statefulset-1
    - Pod名が変わらない
- データを永続化するためにの仕組みがある。
    - PersistentVolumeを使用している場合は、StatefulSetのPodが再起動した際にも同じディスクを利用して再作成される。


StatefulSetを作成するmanifestファイル
```statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: statefulset
spec:
  serviceName: statefulset
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.26
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1G
```

StatefulSetを作成するmanifestファイルの適用
```
$ vi statefulset.yaml
$ kubectl delete -f daemonset.yaml 
daemonset.apps "daemonset" deleted

$ kubectl apply -f statefulset.yaml 
statefulset.apps/statefulset created

$ kubectl get statefulsets
NAME          READY   AGE
statefulset   3/3     6m57s

$ kubectl get pod -o wide
NAME            READY   STATUS    RESTARTS   AGE     IP             NODE                                      NOMINATED NODE   READINESS GATES
statefulset-0   1/1     Running   0          7m11s   192.168.1.27   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
statefulset-1   1/1     Running   0          6m55s   192.168.1.28   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
statefulset-2   1/1     Running   0          6m40s   192.168.1.29   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>

$ kubectl get persistentvolumeclaims
NAME                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
www-statefulset-0   Bound    pvc-06ae1ba3-7153-4b25-9d7c-bec00d1a459e   1Gi        RWO            standard-rwo   7m34s
www-statefulset-1   Bound    pvc-ec72faac-a870-4ee8-9497-4ed3bc6f5cc5   1Gi        RWO            standard-rwo   7m18s
www-statefulset-2   Bound    pvc-d7e80ae7-5572-45b9-a93b-0dc73c461ad4   1Gi        RWO            standard-rwo   7m3s

$ kubectl get persistentvolume
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                       STORAGECLASS   REASON   AGE
pvc-06ae1ba3-7153-4b25-9d7c-bec00d1a459e   1Gi        RWO            Delete           Bound    default/www-statefulset-0   standard-rwo            7m52s
pvc-d7e80ae7-5572-45b9-a93b-0dc73c461ad4   1Gi        RWO            Delete           Bound    default/www-statefulset-2   standard-rwo            7m21s
pvc-ec72faac-a870-4ee8-9497-4ed3bc6f5cc5   1Gi        RWO            Delete           Bound    default/www-statefulset-1   standard-rwo
```



## Job
Jobは、コンテナを利用して一度限りの処理を実行させるリソースです。ReplicaSetではPodが停止することは予期せぬエラーですが、Jobの場合は、Podの停止が正常終了として期待される用途に向いています。バッチ処理などで利用されます。一度だけ実行するタスクにも利用できますし、並列実行するタスクにも利用することができます。


![](/images/kubernetes-workload-20240512/Job.drawio.png)


Jobを作成するmanifestファイル
```job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 10
  template:
    spec:
      containers:
      - name: tools-container
        image: amsy810/tools:v2.0
        command: ["sleep"]
        args: ["60"]
      restartPolicy: Never
```

Jobを作成するmanifestファイルの適用
```
$ vi job.yaml 

$ kubectl apply -f job.yaml 
job.batch/job created

$ kubectl get jobs
NAME   COMPLETIONS   DURATION   AGE
job    0/1           7s         7s

$ kubectl get pods -o wide --watch
NAME        READY   STATUS              RESTARTS   AGE   IP       NODE                                      NOMINATED NODE   READINESS GATES
job-6nnnc   0/1     ContainerCreating   0          16s   <none>   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
job-6nnnc   1/1     Running             0          16s   192.168.1.30   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
job-6nnnc   0/1     Completed           0          77s   192.168.1.30   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
job-6nnnc   0/1     Completed           0          78s   <none>         gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
job-6nnnc   0/1     Completed           0          79s   192.168.1.30   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>
job-6nnnc   0/1     Completed           0          79s   192.168.1.30   gke-gke-cluster-node-pool-f7c3c726-4lxr   <none>           <none>

$ kubectl get jobs
NAME   COMPLETIONS   DURATION   AGE
job    1/1           79s        103s
```


## CronJob
CronJobは、CronのようにスケジューリングされたJobを実行したい場合に利用します。CronJobがJobを管理し、JobがPodを管理するような3層の親子関係になっています。定期的なバッチ処理で利用する場合はJobよりもCronJobを利用します。


![](/images/kubernetes-workload-20240512/CronJob.drawio.png)

CronJobを作成するmanifestファイル
```cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob
spec:
  schedule: "*/1 * * * *"
  concurrencyPolicy: Allow
  startingDeadlineSeconds: 30
  successfulJobsHistoryLimit: 5
  failedJobsHistoryLimit: 3
  suspend: false
  jobTemplate:
    spec:
      completions: 1
      parallelism: 1
      backoffLimit: 0
      template:
        spec:
          containers:
          - name: tools-container
            image: amsy810/random-exit:v2.0
          restartPolicy: Never
```


CronJobを作成するmanifestファイルの適用
```
$ vi cronjob.yaml

$ kubectl apply -f cronjob.yaml 

cronjob.batch/cronjob created

$ kubectl get cronjobs
NAME      SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob   */1 * * * *   False     0        <none>          9s

$ kubectl get jobs -o wide --watch
NAME               COMPLETIONS   DURATION   AGE   CONTAINERS        IMAGES                     SELECTOR
cronjob-28592599   0/1           12s        12s   tools-container   amsy810/random-exit:v2.0   batch.kubernetes.io/controller-uid=4b9f79f5-6194-4470-9bdf-261223737c3c
cronjob-28592600   0/1                      0s    tools-container   amsy810/random-exit:v2.0   batch.kubernetes.io/controller-uid=a1748ca6-47c3-4264-abd2-ad533436459d
cronjob-28592600   0/1           0s         0s    tools-container   amsy810/random-exit:v2.0   batch.kubernetes.io/controller-uid=a1748ca6-47c3-4264-abd2-ad533436459d
cronjob-28592600   0/1           3s         3s    tools-container   amsy810/random-exit:v2.0   batch.kubernetes.io/controller-uid=a1748ca6-47c3-4264-abd2-ad533436459d
cronjob-28592600   1/1           3s         3s    tools-container   amsy810/random-exit:v2.0   batch.kubernetes.io/controller-uid=a1748ca6-47c3-4264-abd2-ad533436459d
```

# 参考
- [Kubernetes完全ガイド 第2版](https://book.impress.co.jp/books/1119101148)
- [Pod](https://kubernetes.io/docs/concepts/workloads/pods/)
- [ReplicationController](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)
- [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
- [Cronjob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)