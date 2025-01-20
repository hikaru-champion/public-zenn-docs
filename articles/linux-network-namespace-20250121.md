---
title: ""
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# 

# 対象OS
今回はUbuntu 20.04上で動作確認しています。
```bash
$ cat /etc/os-release 
NAME="Ubuntu"
VERSION="20.04.6 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.6 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```

# namespaceとは
namespaceとは、Linuxカーネルに実装されている機能でプロセスをグループ化して、隔離された空間を作り出す役割があります。
隔離された空間を作り出したいリソースによっていくつかのnamespaceを持ちます。
Linuxがアクセスすることが可能な、namespaceの一覧をリスト表示するには、**lsns**コマンドを実行することで確認できる。
```bash
$ sudo lsns
        NS TYPE   NPROCS   PID USER            COMMAND
4026531835 cgroup    110     1 root            /sbin/init
4026531836 pid       110     1 root            /sbin/init
4026531837 user      110     1 root            /sbin/init
4026531838 uts       107     1 root            /sbin/init
4026531839 ipc       110     1 root            /sbin/init
4026531840 net       110     1 root            /sbin/init
4026531841 mnt       103     1 root            /sbin/init
4026531862 mnt         1    25 root            kdevtmpfs
4026532205 mnt         1   206 root            /lib/systemd/systemd-udevd
4026532206 uts         1   206 root            /lib/systemd/systemd-udevd
4026532253 mnt         1   466 systemd-network /lib/systemd/systemd-networkd
4026532254 mnt         1   470 systemd-resolve /lib/systemd/systemd-resolved
4026532255 mnt         2   513 _chrony         /usr/sbin/chronyd -F -1
4026532256 mnt         1  1053 root            /lib/systemd/systemd-logind
4026532318 uts         1  1053 root            /lib/systemd/systemd-logind
4026532319 uts         1   594 root            /lib/systemd/systemd-machined
```

また、現在のプロセスが所属しているnamespaceの一覧は **/proc/[pid]/ns**配下のディレクトリを参照することで、
確認することができます。

```bash
$ ls -l /proc/$$/ns
total 0
lrwxrwxrwx 1 user01 user01 0 Jan 20 20:07 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 user01 user01 0 Jan 20 20:07 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 user01 user01 0 Jan 20 20:07 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx 1 user01 user01 0 Jan 20 20:07 net -> 'net:[4026531840]'
lrwxrwxrwx 1 user01 user01 0 Jan 20 20:07 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 user01 user01 0 Jan 20 20:14 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 user01 user01 0 Jan 20 20:14 time -> 'time:[4026531834]'
lrwxrwxrwx 1 user01 user01 0 Jan 20 20:14 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx 1 user01 user01 0 Jan 20 20:07 user -> 'user:[4026531837]'
lrwxrwxrwx 1 user01 user01 0 Jan 20 20:07 uts -> 'uts:[4026531838]'
```
主なnamespaceの概要について以下にまとめます。

| 名前空間の名前 | 隔離されるリソース | 内容 |
| ---- | ---- | ---- |
| cgroup | cgroup | namespaceごとにcgroupを作成する |
| ipc | メッセージキュー、共有メモリ、セマフォ | namespaceごとにメッセージキュー、共有メモリ、セマフォ（プロセス間通信）を分離する |
| mnt | ファイルシステム | namespaceごとにマウントポイントを分離する |
| net | ネットワークデバイスやアドレス | namespaceごとにネットワークデバイスやアドレスなどを分離する |
| pid | Text | namespaceごとにプロセスIDを分離する |
| time | システムクロック | namespaceごとにシステムクロックを分離する |
| user | UID/GID | namespaceごとにUID/GIDを分離する |
| uts | hostname | namespaceごとにhostnameを分離する |

# network namespaceとは
network namespaceとは、Linuxシステム内でネットワーク関連のリソース（インターフェース、ルーティングテーブル等）を分離することができるカーネルの機能です。network namespaceを利用することで、Linux環境において、独立したネットワーク環境を構築することができます。

## network namespaceを使ってみよう
network namespaceの機能を実際に使ってみたいと思います。
**ip netns list**コマンドを利用することで、現在のnetwork namespaceの一覧を出力することができます。
現在は、何もnetwork namespaceを作成していないので、出力されません。

```bash
$ ip netns list
```
**ip netns add**コマンドを実行することで、新規にnetwork namespaceを作成することができます。
この場合は、**ns01**というnetwork namespaceを作成しました。
先ほどのip netns listコマンドでns01が作成されたことを確認することができます。

```bash
$ sudo ip netns add ns01
$ ip netns list
ns01
```

**ip netns exec [network namespace] [実行したいコマンド]**コマンドを利用することで、
新たに作成されたns01というnetwork namespaceの環境でコマンドを実行することができます。

```bash
$ sudo ip netns exec ns01 ip address show
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
$ sudo ip netns exec ns01 ip link show
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
$ sudo ip netns exec ns01 ip route show
Error: ipv4: FIB table does not exist.
Dump terminated
```
出力内容を確認すると、IPアドレス、MACアドレス、ルートテーブルともに設定がされていないことがわかります。
また、network namespaceを使用しないで、IPアドレスを確認するコマンドを実行すると、出力内容が大きく違っていることがわかります。
※以下の出力結果は実行環境によって異なります。

```bash
$ ip address show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
    link/ether 42:01:c0:a8:00:03 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.3/32 metric 100 scope global dynamic ens4
       valid_lft 2201sec preferred_lft 2201sec
    inet6 fe80::4001:c0ff:fea8:3/64 scope link 
       valid_lft forever preferred_lft forever
3: lxcbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 00:16:3e:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 10.0.3.1/24 brd 10.0.3.255 scope global lxcbr0
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:e1:f2:cc brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:e1:f2:cc brd ff:ff:ff:ff:ff:ff
6: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:59:45:dc:3c brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```
このことから、network namespaceを使用することで、ネットワーク的に独立した環境をLinux内に作り出すことができます。
network namespaceを削除したい場合は、**ip netns delete**コマンドを実行することで削除することができます。

```bash
$ sudo ip netns delete ns01
$ ip netns list
```

## network namespace同士の接続

network namespace (ns01 と ns02)を作成していきます。
```bash
whikaru9@ubuntu:~$ sudo ip netns add ns01
whikaru9@ubuntu:~$ sudo ip netns add ns02
whikaru9@ubuntu:~$ ip netns list
ns02
ns01
```
この時のnetwork namespaceのns01とns02はそれぞれ独立したネットワークとなっており、
IPアドレスや、ルーティングテーブルも別々に持っていることになります。

このns01とns02を接続させるためには、**veth (Virtual Ethernet Device)**という仮想ネットワークインターフェースを
作成する必要があります。
**ip link add **コマンドを実行することで、仮想ネットワークインターフェースを作成することができます。
作成した仮想ネットワークインターフェースは、2つのネットワークインターフェースがペアになっており、
1本のLANケーブルで接続された状態になっていると言えます。
```bash
whikaru9@ubuntu:~$ sudo ip link add ns01-veth0 type veth peer name ns02-veth0
whikaru9@ubuntu:~$ ip link show | grep veth
7: ns02-veth0@ns01-veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
8: ns01-veth0@ns02-veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
```
この状態では、まだnetwork namespaceに所属しておらず、Linuxシステム上に存在するため、
**ip link set**コマンドを利用することで、各network namespaceに所属させます。
```bash
whikaru9@ubuntu:~$ sudo ip link set ns01-veth0 netns ns01
whikaru9@ubuntu:~$ sudo ip link set ns02-veth0 netns ns02
```

上記のコマンドを実行した後は、Linuxシステム上からは先ほど確認できた仮想ネットワークインターフェースは確認できなくなり、
ns01とns02のnetwork namespace上にそれぞれ仮想ネットワークインターフェースが移動していることが確認することができました。

```bash
whikaru9@ubuntu:~$ ip link show | grep veth
whikaru9@ubuntu:~$ sudo ip netns exec ns01 ip link show | grep veth
8: ns01-veth0@if7: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
whikaru9@ubuntu:~$ sudo ip netns exec ns02 ip link show | grep veth
7: ns02-veth0@if8: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
```
この状態では、network namespaceが仮想LANケーブルで接続されただけなので、接続することはできません。
次は、**ip address add**コマンドを実行することで仮想ネットワークインターフェースに対して、
IPアドレスを設定していきます。
```bash
whikaru9@ubuntu:~$ sudo ip netns exec ns01 ip address add 192.168.0.1/24 dev ns01-veth0
whikaru9@ubuntu:~$ sudo ip netns exec ns02 ip address add 192.168.0.2/24 dev ns02-veth0
```
network namespaceのそれぞれの仮想ネットワークインターフェースにIPアドレスを付与することができました。

```bash
whikaru9@ubuntu:~$ sudo ip netns exec ns01 ip address show
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
8: ns01-veth0@if7: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether c2:d7:eb:75:aa:da brd ff:ff:ff:ff:ff:ff link-netns ns02
    inet 192.168.0.1/24 scope global ns01-veth0
       valid_lft forever preferred_lft forever
whikaru9@ubuntu:~$ sudo ip netns exec ns02 ip address show
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
7: ns02-veth0@if8: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 9a:0a:57:97:f0:b9 brd ff:ff:ff:ff:ff:ff link-netns ns01
    inet 192.168.0.2/24 scope global ns02-veth0
       valid_lft forever preferred_lft forever
```

次は、仮想ネットワークインターフェースを有効（UP）なステータスに変更します。
仮想ネットワークインターフェースは、作成初期の状態のステータスは無効（DOWN）になっていることがあるので、
有効（UP）なステータスに変更します。
```bash
whikaru9@ubuntu:~$ sudo ip netns exec ns01 ip link show ns01-veth0 | grep state
8: ns01-veth0@if7: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
whikaru9@ubuntu:~$ sudo ip netns exec ns02 ip link show ns02-veth0 | grep state
7: ns02-veth0@if8: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
```

**ip link set <仮想ネットワークインターフェース> up**コマンドで有効にします。
```bash
whikaru9@ubuntu:~$ sudo ip netns exec ns01 ip link set ns01-veth0 up
whikaru9@ubuntu:~$ sudo ip netns exec ns02 ip link set ns02-veth0 up
whikaru9@ubuntu:~$ sudo ip netns exec ns01 ip link show ns01-veth0 | grep state
8: ns01-veth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
whikaru9@ubuntu:~$ sudo ip netns exec ns02 ip link show ns02-veth0 | grep state
7: ns02-veth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
```

これで、network namespace同士が
- 仮想ネットワークインターフェースで接続
- IPアドレスも付与
することができているので、pingコマンドを実行して疎通することができるか確認してみます。

```bash
whikaru9@ubuntu:~$ sudo ip netns exec ns01 ping -c 3 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=64 time=0.084 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=64 time=0.170 ms
64 bytes from 192.168.0.2: icmp_seq=3 ttl=64 time=0.074 ms

--- 192.168.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2027ms
rtt min/avg/max/mdev = 0.074/0.109/0.170/0.043 ms

```
無事、ns01からns02へ疎通することができました。


# 参考文献
https://container-security.dev/namespace/
