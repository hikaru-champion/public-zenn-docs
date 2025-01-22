---
title: "【Linux】コンテナで使用されるLinuxカーネル技術~uts namespace/pid namspace偏~"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Linux", "コンテナ", "カーネル"]
published: true
---
# はじめに
こんにちは。championです。
普段は、Google CloudやAWSを中心としたクラウドインフラの設計～保守運用を行なっています。

Google CloudのCloud Runや、AWSのECSでも利用されるコンテナですが、コンテナを構成する技術について深入りしてみたく、コンテナで使用されるLinuxカーネル技術のnamespaceのuts namespaceとpid namespaceに焦点を当てて調べてみました。

前回は、network namespaceについて調べた内容について記事を書いています。
https://zenn.dev/hi_ka_ru/articles/linux-network-namespace-20250121

# 対象OS
今回もGoogle Cloud上のCompute EngineのUbuntu 20.04上で動作確認しています。
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
Linuxがアクセスすることが可能な、namespaceの一覧をリスト表示するには、**lsns**コマンドを実行することで確認できます。
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

また、現在のプロセスが所属しているnamespaceの一覧は **/proc/[pid]/ns**配下のディレクトリを参照することで、確認することができます。

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

# uts namespaceとは
uts namespaceは、namespaceごとにhostnameやNISドメイン名を分離することができるカーネルの機能です。

## uts namespaceを使ってみよう
まず、Linuxホストのホスト名とNISドメイン名を確認してみます。
```bash
$ hostname
ubuntu
$ domainname
(none)
```
ホスト名は「ubuntu」、NISドメイン名は設定されていないことがわかります。

次に、**unshare**コマンドを実行してuts名前空間を作成してみて、uts名前空間上でホスト名とドメイン名を設定してみます。

```bash
$ sudo unshare --uts
# hostname hogehoge
# domainname fugafuga
# hostname
hogehoge
# domainname
fugafuga
```
新規に作成したuts名前空間上では、ホスト名を「hogehoge」、NISドメイン名を「fugafuga」に変更してみました。
hostnameコマンド/domainnameコマンドの実行結果もそれぞれ「hogehoge」、「fugafuga」になっていることが確認できると思います。

次に、uts名前空間から抜けて、Linuxホスト上に戻り再度ホスト名とNISドメイン名を確認してみます。
```bash
# exit
logout
$ hostname
ubuntu
$ domainname
(none)
```
ホスト名、NISドメイン名ともに、元の「ubuntu」、「none」のままであることがわかります。Linuxホスト上には影響がなかったことになります。
以上のことから、uts namespaceは、namespaceごとにhostnameやNISドメイン名を分離することができることがわかりました。

# pid namespaceとは
pid namespaceとは、namespaceごとにプロセスIDを分離することができるカーネルの機能です。
通常、pidはLinuxホスト上でpid:1から始まり、新規にプロセスが生成されることでインクリメントされていきます。Linuxホスト上で一意のプロセスを特定するために利用されるため、同じpidのプロセスが複数できることはないです。

```bash
$ ps aux | head -n 10
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.5 103060 11864 ?        Ss   Jan20   0:08 /sbin/init
root           2  0.0  0.0      0     0 ?        S    Jan20   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        I<   Jan20   0:00 [rcu_gp]
root           4  0.0  0.0      0     0 ?        I<   Jan20   0:00 [rcu_par_gp]
root           5  0.0  0.0      0     0 ?        I<   Jan20   0:00 [slub_flushwq]
root           6  0.0  0.0      0     0 ?        I<   Jan20   0:00 [netns]
root           8  0.0  0.0      0     0 ?        I<   Jan20   0:00 [kworker/0:0H-events_highpri]
root          10  0.0  0.0      0     0 ?        I<   Jan20   0:00 [mm_percpu_wq]
root          11  0.0  0.0      0     0 ?        S    Jan20   0:00 [rcu_tasks_rude_]
```

ふと、Linuxホスト上でどのくらいのpidが生成できるのか気になったので調べてみました。
pidの最大値はカーネルパラメータから確認することができます。私の環境では、**4194304**が最大のようでした。
```bash
$ cat /proc/sys/kernel/pid_max 
4194304
```

## pid namespaceを使ってみよう
おなじみのunshareコマンドを利用して、pid namespaceを新規に作成してpidを確認してみます。

```bash
$ sudo unshare --pid --fork --mount-proc bash
# echo $$
1
# ps
    PID TTY          TIME CMD
      1 pts/0    00:00:00 bash
      8 pts/0    00:00:00 ps
```
新規に作成したpid namespace上起動しているbashでは、pidの値が1になっていることが確認できる。Linuxホスト上ではpidの値が1であるプロセスはinitプロセスなのでプロセスIDを分離することができています。

ここで新規に作成したpid namespace上でsleepコマンドを実行した際に、pid namespace側と、Linuxホスト側でどのようにpidが見えるのかを確認してみます。

```bash
# sleep 120 &
[1] 11
# ps
    PID TTY          TIME CMD
      1 pts/0    00:00:00 bash
     11 pts/0    00:00:00 sleep
     12 pts/0    00:00:00 ps
```
sleepプロセスはpid:11で起動していることがわかります。しかし、Linuxホスト上から確認すると、pid:9454で起動していることがわかります。

```bash
$ ps aux | grep sleep
root        9454  0.0  0.0   7236   516 pts/0    S    21:37   0:00 sleep 120
test        9457  0.0  0.0   8168   656 pts/1    S+   21:38   0:00 grep --color=auto sleep
```
新規に作成したpid namespace上では、そのpid namespace上のプロセスのみ確認することができ、Linuxホスト上からはホスト上のプロセスと新規に作成したpid namespace上のプロセス両方確認することができることがわかりました。

# まとめ
今回は、コンテナで使用されるLinuxカーネル技術のnamespaceであるuts namespaceとpid namespaceに焦点を当てて調べてみました。
1つのLinuxホスト上でもuts namespaceを利用することで、ホスト名/ドメイン名の分離、pid namespaceを利用することでプロセスIDを分離することができることが実機を通して確認することができました。



# 参考文献
- [Container Security Book](https://container-security.dev/namespace/pid.html)
- [CUBE SUGAR CONTAINER](https://blog.amedama.jp/entry/linux-pid-namespace_1)
- [コマンドを叩いて遊ぶ 〜コンテナ仮想、その裏側〜](https://tech.retrieva.jp/entry/2019/04/16/155828#pid-namespace)