---
title: "【Docker再入門】～Dockerネットワーク偏～"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["コンテナ", "Docker"]
published: false
---
# はじめに
こんにちは。championです。
普段は、Google CloudやAWSを中心としたクラウドインフラの設計～保守運用を行なっています。

前回の記事では、インストールしたDockerを実際に動作させてコンテナの基本コマンドについて解説しました。
https://zenn.dev/hi_ka_ru/articles/docker-command-20250202

今回は、Docker環境ののネットワークについて解説します。

# Dockerネットワーク
## Dockerネットワークの概要
Dockerをインストールすると、Docker専用のネットワーク空間 (172.17.0.0/16) と、ネットワークドライバがインストール時に作成されます。

```bash
$ ip address show | grep docker0
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
15: vethb9fdb2a@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 

$ ip link show | grep docker0
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
15: vethb9fdb2a@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP mode DEFAULT group default 

$ ip route show | grep docker0
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
```

## Dockerネットワークドライバ
Dockerのネットワークドライバには以下の6種類存在します。

| ネットワークドライバ | 機能 |
| ---- | ---- |
| bridge | コンテナ作成時に、デフォルトで使用されるネットワークドライバー。同じホスト上の他のコンテナと通信する必要があるコンテナでよく利用されます。 |
| host | コンテナと Docker ホスト間のネットワーク分離を削除し、ホストのネットワークを直接使用します。 |
| overlay | 複数の Docker デーモン間を接続し、Docker Swarmとコンテナがノード間で通信できるようにします。 |
| ipvlan |  IPv4とIPv6の両方のアドレス指定を完全に制御できます。 |
| macvlan | コンテナにMACアドレスを割り当てて、ネットワーク上の物理デバイスとして表示することができます。Dockerデーモンは、トラフィックをMACアドレスによってコンテナにルーティングします。 |
| none | コンテナをホストおよび他のコンテナから完全に分離します。 |

Dockerコンテナをローカル環境で使用する場合は、**bridge**、**host**、**none**の3種類について基本を押さえておけばよいと思います。
Dockerインストール時には、デフォルトで以下のネットワークドライバを作成します。それぞれについて説明していきます。
```bash
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
3d66a57a65ad   bridge    bridge    local
645e777d3a0c   host      host      local
99a964c8093b   none      null      local
```

### bridgeネットワーク
bridgeネットワークは、同じネットワークセグメント間での通信を可能にするリンク層 (L2層)の仮想デバイスです。ブリッジは、通常の物理ネットワーク機器に用いられることもありますが、ソフトウェアでbridgeの機能を提供しているので仮想ブリッジと言います。bridgeネットワークは、同じbridgeネットワークに接続されたコンテナ同士は通信できるようにし、接続されていないコンテナは通信できないように自動的にホストマシンにルールを設定してくれます。

コンテナを起動するとデフォルトの**bridge**ネットワーク上に作成されます。
alpine linuxを300秒Sleepするコンテナを作成してみます。ネットワークの指定はオプションで指定していないのでデフォルトの**bridge**ネットワークが利用されるはずです。
```bash
$ docker container run -d --name alpine-linux --rm alpine sleep 300

$ docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
6f58bcd0bcea   alpine    "sleep 300"   44 seconds ago   Up 44 seconds             alpine-linux
```

**docker network inspect**コマンドを実行することで、ネットワークデバイスの詳細情報を確認することができます。bridgeネットワークの詳細を確認してみます。**Containers**の箇所を確認すると、作成したコンテナ（alpine-linux）がIPアドレス 172.17.0.3/16 としてbridgeネットワークに所属していることが確認できます。
```bash
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "3d66a57a65ad26769b11bb624ddd8d89b50ee8c825b9c465b12a387dcdafefc6",
        "Created": "2025-01-24T23:45:41.480007785Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "6f58bcd0bceaf460afc31683d78e610624061a5275048c2543a8e0f869cfa026": {
                "Name": "alpine-linux",
                "EndpointID": "2ff8e5dadf8131d4956e95ac94de404cc5e12f7b3f0691dc0308573c5b21c157",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

この時にホスト側のネットワーク状況を確認すると、コンテナ起動時にコンテナと通信するための仮想ネットワークインターフェースが作成されています。
```bash
$ ip address show

19: vethc6a8080@if18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether b6:71:fd:a6:b5:9a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::b471:fdff:fea6:b59a/64 scope link 
       valid_lft forever preferred_lft forever
```

これまでを図で表すと以下のような状態になっていることになります。
![bridgeネットワーク01](/images/docker-network-20250202/bridge-network01.drawio.png)


次に、複数のコンテナを作成したときにはどうなるでしょうか？
新しく**alpine-linux01**と**alpine-linux02**を作成してbridgeネットワークの状態を確認してみます。

```bash
$ docker container run -d --name alpine-linux01 --rm alpine sleep 300
929b7849fb1206108c6b32cbe9fa4f28b243a68f64fd9402396bcdbbe4fefb51

$ docker container run -d --name alpine-linux02 --rm alpine sleep 300
a83bab22d215f900b2e6c5271d95d3408640d0ab7531994441eaf2fb975319b2

$ docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
a83bab22d215   alpine    "sleep 300"   5 seconds ago   Up 4 seconds             alpine-linux02
929b7849fb12   alpine    "sleep 300"   8 seconds ago   Up 8 seconds             alpine-linux01

$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "3d66a57a65ad26769b11bb624ddd8d89b50ee8c825b9c465b12a387dcdafefc6",
        "Created": "2025-01-24T23:45:41.480007785Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "929b7849fb1206108c6b32cbe9fa4f28b243a68f64fd9402396bcdbbe4fefb51": {
                "Name": "alpine-linux01",
                "EndpointID": "94532654b4f11232941bcb8521a45ef41c1251790ac79ea55d2c9f326a1cb4f3",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "a83bab22d215f900b2e6c5271d95d3408640d0ab7531994441eaf2fb975319b2": {
                "Name": "alpine-linux02",
                "EndpointID": "cccb8c9a7a86d0bd5dcd2cf3664fb4f35460784ebd066546f679b22aca40d445",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```
二つのコンテナは、bridgeネットワークに接続され、alpine-linux01には172.17.0.2/16、alpine-linux02には172.17.0.3/16のIPアドレスが設定されていることが確認できました。
これまでを図で表すと以下のような状態になっていることになります。
![bridgeネットワーク02](/images/docker-network-20250202/bridge-network02.drawio.png)

同じbridgeネットワークに接続されているコンテナ同士は相互に通信することが可能です。それぞれのコンテナからコンテナへpingを実行すると正常に通信できました。

```bash
$ docker container exec alpine-linux01 ping -c 3 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.186 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.077 ms
64 bytes from 172.17.0.3: seq=2 ttl=64 time=0.125 ms

--- 172.17.0.3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.077/0.129/0.186 ms

$ docker container exec alpine-linux02 ping -c 3 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.074 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.124 ms
64 bytes from 172.17.0.2: seq=2 ttl=64 time=0.097 ms

--- 172.17.0.2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.074/0.098/0.124 ms
```

bridgeネットワークは、ユーザ自身で定義して作成することもできます。ユーザで定義して作成したbridgeネットワークでは、コンテナ間の通信に**DNSによる名前解決機能**を利用することできます。DNSによる名前解決機能を利用することでコンテナ間の通信を行う際に、IPアドレスを直接指定することなく、コンテナ名で通信を行うことができます。
また、異なるbridgeネットワークに属しているコンテナ間では、互いに通信することはできません。そのため、ネットワークを分離することで不要なコンテナ間の通信を防ぐことができ分離性が向上します。コンテナ自身はコマンド一つでネットワークに接続・切断することができます。

実際に、bridgeネットワークを作成してみましょう。ユーザ定義のネットワークを作成するためには、**docker network create**コマンドを実行します。
```bash
$ docker network create my-net --driver bridge
300b094e6d5d37d7d59719045a4e3c1ae7bdd8cd7b44097c8ef36e07cf84e051
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
3d66a57a65ad   bridge    bridge    local
645e777d3a0c   host      host      local
300b094e6d5d   my-net    bridge    local
99a964c8093b   none      null      local
```

ユーザ定義のbridgeネットワーク（my-net）を作成することができたので、詳細情報を確認していきます。**docker network inspect**コマンドを実行することで、ネットワークの詳細情報を確認することができます。
```bash
$ docker network inspect my-net
[
    {
        "Name": "my-net",
        "Id": "300b094e6d5d37d7d59719045a4e3c1ae7bdd8cd7b44097c8ef36e07cf84e051",
        "Created": "2025-01-30T20:21:36.397509121Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

作成したネットワーク内にコンテナを立てて、コンテナ間の通信に**DNSによる名前解決機能**を利用してみましょう。
```bash
$ docker container run -d --name alpine-linux01 --network my-net --rm alpine sleep 300 
2952b84733460370e53fd0b803145b1699bc6e23637b3d6d867b52f11e9ee8b7
$ docker container run -d --name alpine-linux02 --network my-net --rm alpine sleep 300 
759a82366abe67c17a6ff2d48501e0782e858ea0237f2f7e243df2169e201b49

$ docker container exec alpine-linux01 ping -c 3 alpine-linux02
PING alpine-linux02 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.071 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.122 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.115 ms

--- alpine-linux02 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.071/0.102/0.122 ms

$ docker container exec alpine-linux02 ping -c 3 alpine-linux01
PING alpine-linux01 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.139 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.088 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.231 ms

--- alpine-linux01 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.088/0.152/0.231 ms
```
DNSによる名前解決機能を利用することでコンテナ間の通信を行う際に、IPアドレスを直接指定することなく、コンテナ名を利用し通信を行うことができました。

![bridgeネットワーク03](/images/docker-network-20250202/bridge-network03.drawio.png)

それでは、作成したネットワーク内のコンテナと、デフォルトのbridgeネットワーク内のコンテナ間の通信はどうなるでしょうか？

```bash
$ docker container run -d --name alpine-linux03 --rm alpine sleep 300 
52b3da9559a35953c829742cd452142e66fc8abed6f8ffb2242d055b7bd74695

$ docker container exec alpine-linux02 ping -c 3 alpine-linux03
ping: bad address 'alpine-linux03'

$ docker container exec alpine-linux01 ping -c 3 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes

--- 172.17.0.2 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss
```
作成したネットワーク内のコンテナから、デフォルトのbridgeネットワーク内のコンテナへの通信は、「ping: bad address 'alpine-linux03'」と表示され、**DNSによる名前解決機能**が利用できませんでした。また、IPアドレスを直指定しての通信もできないことがわかりました。これにより、異なるbridgeネットワークに属しているコンテナ間では、互いに通信することはできないことが確認できました。
![bridgeネットワーク04](/images/docker-network-20250202/bridge-network04.drawio.png)

自身で作成したネットワークを削除したい場合は、**docker network rm**コマンドを実行します。
```bash
$ docker network rm my-net
my-net

$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
3d66a57a65ad   bridge    bridge    local
645e777d3a0c   host      host      local
99a964c8093b   none      null      local
```

### hostネットワーク
hostネットワークは、dockerホスト側のネットワークを利用することになります。コンテナをhostネットワークに接続した場合、dockerホスト側のネットワークと同じ設定になります。コンテナ自身にはIPアドレスは付与されず、またポートの割当ても無効になります。nginxコンテナをhostネットワークに接続することで、dockerホストのIPアドレス、ポート80でnginxが起動します。そのため、curlコマンドを**localhost:80**に実行することでnginxのhtmlが返ってきます。

![hostネットワーク](/images/docker-network-20250202/host-network.drawio.png)

```bash
$ docker container run -d --name nginx --network host nginx 
95a8a17597590d09e4f62e1e43e54dcdaec54b9241469bb44617454babeb6402

$ docker container ls
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
95a8a1759759   nginx     "/docker-entrypoint.…"   4 seconds ago    Up 3 seconds              nginx

$ curl localhost:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### noneネットワーク
noneネットワークは、コンテナのネットワーク機能を無効化し、完全に独立した状態にすることができます。コンテナ自身にIPアドレスは割り振られず、コンテナ間の通信や、外部のインターネットに通信をしたりすることはできません。
noneネットワークに接続しているコンテナ内で**各種ipコマンド**を実行するとループバックインターフェースしか存在せず、ルートテーブルは存在しないです。また、IPアドレス8.8.8.8へpingコマンドを実行しても**Network unreachable**と表示され隔離されていることがわかります。

![noneネットワーク](/images/docker-network-20250202/none-network.drawio.png)

```bash
$ docker container run -d --name alpine-linux01 --network none --rm alpine sleep 300 
e9b82e397e550835c369013cbcc9ab1a046e670f908aeecef54f7751ec53d0f8

$ docker network inspect none
[
    {
        "Name": "none",
        "Id": "99a964c8093b7b0b10eb5d43f643f6f5831456bb18f01a9d1aee2230eb5a89cc",
        "Created": "2025-01-24T23:17:48.861986575Z",
        "Scope": "local",
        "Driver": "null",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": null
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "e9b82e397e550835c369013cbcc9ab1a046e670f908aeecef54f7751ec53d0f8": {
                "Name": "alpine-linux01",
                "EndpointID": "139b7879587a0481cde520ead77e3a3b083171c8ead5ce902ceaeecac34dce7d",
                "MacAddress": "",
                "IPv4Address": "",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
$ docker container exec alpine-linux01 ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

$ docker container exec alpine-linux01 ip address show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever

$ docker container exec alpine-linux01 ip route show

$ docker container exec alpine-linux01 ping -c 3 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
ping: sendto: Network unreachable
```

Dockerのネットワーク環境について**bridge**、**host**、**none**の3つのネットワークドライバの説明を行ってきました。開発を行っていくうえでコンテナ間で通信することは当たり前にあるので、Docker環境のネットワークについて何が起こっているのか理解する助けになれば幸いです。
次回はDockerのストレージについて説明していきます。

# 参考
https://docs.docker.jp/engine/reference/commandline/network_toc.html
https://docs.docker.jp/network/bridge.html
https://docs.docker.jp/network/host.html
https://docs.docker.jp/network/none.html