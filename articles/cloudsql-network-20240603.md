---
title: "Cloud SQLのネットワークと接続方法"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GoogleCloud", "CloudSQL"]
published: false
---
# はじめに
こんにちは。hikaruです。
今回は**Cloud SQLのネットワークと接続方法**について解説していきます。
前回記載した[**Cloud SQL基礎**](https://zenn.dev/hi_ka_ru/articles/cloudsql-20240602)に続く記事になります。


AWSでは、RDSを構築する際はユーザ管理のネットワークに構築しますが、
Google CloudのCloud SQLは、ユーザ管理のネットワークに存在しなく、Google Cloud側のネットワーク上（サービスプロデューサVPC）に構築されるのが特徴です。

Cloud SQLには接続方法として2つの方法があります。
- インターネットを経由する**Public IP**での接続
- VPC内部を経由する**Private IP**での接続

それぞれの接続方法について深堀って説明していきたいと思います。

# Public IPでの接続
Public IPでの接続の場合、Cloud SQL自体にIPv4のPublic IPアドレスが付与されます。
Public IPアドレスが付与されているということは、公共のインターネットを経由してCloud SQLに接続することができるということです。
公共のネットワークを経由するため、セキュリティを担保するためにCloud SQLへの接続には、接続元を制限することができる**承認済みネットワーク**の機能や、通信の暗号化を行う**セルフマネージド SSL / TLS 証明書**や、**Cloud SQL Auth Proxy**の機能を利用することが推奨されています。

![](/images/cloudsql-network-20240603/cloudsqlpublicip.png)

## 承認済みネットワーク
Public IPを利用してCloudSQL接続を行う場合、特定のIPアドレスまたはアドレス範囲からの接続を受け入れる**承認済みネットワーク**に設定する必要があります。
承認済みネットワークは、以下の制約があります。
- IPv4でしか利用できない（IPv6はサポートされていない）
- RFC 1918（プライベートIP）のアドレスは利用できない


### 承認済みネットワークに登録しない場合
Cloud Shellから、構築済みのCloudSQLへPublic IPでの接続を試してみます。
```
$ mysql -uroot -p -h 35.200.xxx.xxx
Enter password: 
ERROR 2003 (HY000): Can't connect to MySQL server on '35.200.xxx.xxx:3306' (110)
```
上記の通り、承認済みネットワークに接続元IPアドレスを登録しない場合は、CloudSQLへ接続することができません。


### 承認済みネットワークに登録した場合
Cloud Shell上で以下のコマンドを実行し、自身のIPアドレスを確認します。
```
$ curl httpbin.org/ip
{
  "origin": "35.185.xxx.xxx"
}
```

確認できたIPアドレスをCloudSQLの承認済みネットワークに登録します。
![](/images/cloudsql-network-20240603/cloudsql01.png)


承認済みネットワークに登録後、再度接続を試みるとCloud ShellからCloudSQLへ接続することができました。
```
$ mysql -uroot -p -h 35.200.xxx.xxx
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 22928
Server version: 8.0.31-google (Google)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

# Private IPでの接続
CloudSQLでPrivate IP接続を行いたい場合は、CloudSQLが存在するGoogle Cloud側のネットワーク（VPC）とユーザ側のネットワーク（VPC）と**VPCピアリング**をする必要があります。

このようなGoogle Cloud側のVPCと、ユーザー側のVPCを接続し、プライベート接続が利用できることを**プライベートサービスアクセス**と言います。Private IP接続を使用すると、インターネットを利用せず、Google Cloud内のネットワークを経由してCloudSQLへ接続することができ、Public IP接続よりもネットワーク レイテンシを低く抑えることができます。

Cloud SQLでPrivate IP接続を行う場合は、インスタンスIPの割り当てを**プライベートIP**に設定し、プライベートサービスアクセスを用いて接続するVPCを選択する必要があります。

![](/images/cloudsql-network-20240603/cloudsqlprivateip.png)

![](/images/cloudsql-network-20240603/cloudsql02.png)

プライベートサービスアクセスを初めて利用する場合は、以下のような設定を求められます。
- Service Networking APIの有効化
- IP範囲を割り振る
今回は、**自動的に割り当てられたIPアドレスを使用する**を選択しますが、本番で利用される場合はCIDR設計を行った後最適なIP範囲を設定してください。以下に、サブネットマスクと利用できるIPアドレス数、使用可能なCloudSQLインスタンスについて記載します。

| サブネットマスク | アドレス数 | 使用可能なCloudSQLインスタンス |
| ---- | ---- | ---- |
| /24 | 256 | 50 |
| /23 | 512 | 100 |
| /22 | 1024 | 200 |
| /21 | 2048 | 400 |
| /20 | 4096 | 800 |

- 接続を作成する

![](/images/cloudsql-network-20240603/cloudsql03.png)

以上の設定が完了すると、Private IPが付与されたCloudSQLインスタンスが作成されます。

![](/images/cloudsql-network-20240603/cloudsql04.png)

## 実際に接続してみる
VPC上に構築したGCEインスタンスにmysql clientをインストールし、Cloud SQLへPrivate IPを用いて接続してみます。

```
$ mysql -uroot -p -h 10.42.208.3
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2413
Server version: 8.0.31-google (Google)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
パスワードを入力後、Cloud SQLへPrivate IPを用いて接続することができました。


# Cloud SQL Auth Proxy
## Cloud SQL Auth Proxyとは
Cloud SQL Auth Proxyとは、承認済みネットワークやSSLの設定が不要で安全にCloudSQLへの接続をプロキシしてくれるソフトウェアになります。
Cloud SQL Auth Proxyを利用したCloudSQLへの接続フローは以下の通りです。

- ローカルのCloud SQL Auth Proxyへ接続を行う（この時接続するプロトコルは標準プロトコル）
- Cloud SQL Auth Proxyは、セキュアなトンネルを使用して、サーバー上で動作しているProxyサーバと接続を行う
- 既存の接続がある場合は、既存の接続を利用し、新規接続の場合は、Cloud SQL Admin APIを利用し一時的なSSL証明書を取得し、それを利用してCloud SQLと接続します。一時的なSSL証明書は約一時間で更新されます。

![](/images/cloudsql-network-20240603/cloudsqlauthproxy.png)

## メリット
Cloud SQL Auth Proxyを利用するメリットは以下の通りです。
- 暗号化された接続

Cloud SQL Auth Proxyは、TLS1.3と256ビットAES暗号を使用し、クライアントとCloudSQLとの間で送受信されるトラフィックを自動的に暗号化できます。また、SSL証明書は管理する必要はありません。
- IAMを利用した認可

Cloud SQL Auth Proxyは、IAM権限を利用することでCloudSQLへ接続するユーザを制御します。また、Public IP接続の場合は、先述した**承認済みネットワーク**の設定をせずとも接続が可能です。
:::message
Private IP接続でCloud SQL Auth Proxyを利用する場合は、Cloud SQL Auth Proxyがユーザ管理のVPCネットワーク上に存在する必要があります。
:::


## Cloud SQL Auth Proxyを利用したプライベート接続
Cloud SQL Auth Proxy をGCEインスタンスへダウンロードします。
```
$ curl -o cloud-sql-proxy https://storage.googleapis.com/cloud-sql-connectors/cloud-sql-proxy/v2.11.0/cloud-sql-proxy.linux.amd64
```

Cloud SQL Auth Proxyに実行権を付与します。
```
$ chmod +x cloud-sql-proxy
```

Cloud SQL Auth Proxyを起動する
プライベート接続でCloud SQL Auth Proxyを利用する場合には、**--private-ip**オプションが必要です。
```
$ ./cloud-sql-proxy --port 3306 xxxxxxxxxxx:asia-northeast1:test-instance-01 --private-ip
2024/06/03 21:11:17 Authorizing with Application Default Credentials
2024/06/03 21:11:17 [deep-thought-422207-j1:asia-northeast1:test-instance-01] Listening on 127.0.0.1:3306
2024/06/03 21:11:17 The proxy has started successfully and is ready for new connections!
```

別のコンソール画面を立ち上げ、Cloud SQLへCloud SQL Auth Proxy経由で接続してみる
```
$ mysql -u root -p --host 127.0.0.1 --port 3306
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5482
Server version: 8.0.31-google (Google)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
無事接続することができました。
Cloud SQL Auth Proxyのログには**Accepted connection from 127.0.0.1:45586**と表示され接続があったログが表示されていました。
```
2024/06/03 21:11:44 [deep-thought-422207-j1:asia-northeast1:test-instance-01] Accepted connection from 127.0.0.1:45586
```

# さいごに
今回はCloud SQLのネットワークと接続方法についてまとめました。
Cloud SQLには、Public IP接続とPrivate IP接続の2種類のネットワーク構成をとることができますが、データベースには大切なデータが入っていることが多くインターネット越しにアクセスさせることはセキュアではないので、Private IP接続でのネットワーク構成をとることが多いと思います。
また、Cloud SQL Auth Proxyを利用した接続も試してみました。クライアントとCloud SQL間の接続が自動的に暗号化されるため、Public IP接続でCloud SQLへアクセスする場合は必須の機能になると思います。
最後までご覧いただきありがとうございました。