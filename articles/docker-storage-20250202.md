---
title: ""
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---
# はじめに
こんにちは。championです。
普段は、Google CloudやAWSを中心としたクラウドインフラの設計～保守運用を行なっています。

前回の記事では、Docker環境のネットワークについて解説しました。
https://zenn.dev/hi_ka_ru/articles/docker-network-20250202

今回は、Dockerでのデータ管理について解説します。

# Docker Storage
## Dockerでのデータ管理方法
Dockerコンテナ内で作成されたファイル等のデータは、コンテナが削除されると一緒に消えてしまいます。これはLinuxの名前空間という機能によりコンテナのプロセスごとに独立したファイルシステムを持つため、コンテナ（プロセス）が削除され、新たにコンテナ（プロセス）が作成されると全く別の名前空間が作成され、そのファイルシステムを利用するためファイルの引継ぎはできないです。
コンテナを作成し、コンテナ内にファイルを作成したあと、コンテナ内から抜けます。
```bash
$ docker container run -dit --name alpine-linux alpine
20f9fdf613b700adb2424d157c5b04dcd24a25daea42e1f015af6a3691cb262d

$ docker container ls
CONTAINER ID   IMAGE     COMMAND     CREATED         STATUS         PORTS     NAMES
20f9fdf613b7   alpine    "/bin/sh"   2 seconds ago   Up 2 seconds             alpine-linux

$ docker container exec -it alpine-linux /bin/sh
/ # touch ~/text.txt
/ # ls ~/text.txt
/root/text.txt
/ # exit
```
コンテナを停止し、削除します。

```bash
$ docker container stop alpine-linux
alpine-linux

$ docker container rm alpine-linux
alpine-linux
```

再度、同じコンテナを作成してコンテナ内に入って先ほど作成したファイルを確認しようとすると、**/root/text.txt: No such file or directory**と表示されファイルが引き継がれていないことが確認できます。
```bash
$ docker container run -dit --name alpine-linux alpine
770665894aaf569c4d06a84312888321db59dfdaa24cf6e4eb643416e5eecfbe

$ docker container ls
CONTAINER ID   IMAGE     COMMAND     CREATED         STATUS         PORTS     NAMES
770665894aaf   alpine    "/bin/sh"   7 seconds ago   Up 6 seconds             alpine-linux

$ docker container exec -it alpine-linux /bin/sh
/ # ls ~/text.txt
ls: /root/text.txt: No such file or directory
/ # exit
```

コンテナ内で作成したデータをホスト上に保存したい、別のコンテナからも共有して使用したいといったデータの永続性を実現するための技術としてDockerはいくつかのマウント機能を提供しています。

## マウント
### volumeマウント
volumeマウントは、Docker によって管理されているホストファイルシステム上の一部にデータを保管します。Linuxホスト上では **/var/lib/docker/volumes**ディレクトリ配下を使用します。**docker volume ls**コマンドを実行することで、volumeの一覧を確認することができます。また、**docker volume create**コマンドを実行することで、新規のボリュームを作成することができ、ボリュームの詳細情報を確認したい場合は**docker volume inspect**コマンドを実行することで、確認することができます。

```bash
$ docker volume ls
DRIVER    VOLUME NAME

$ docker volume create vm1
vm1

$ docker volume ls
DRIVER    VOLUME NAME
local     vm1

$ docker volume inspect vm1
[
    {
        "CreatedAt": "2025-01-31T21:07:16Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/vm1/_data",
        "Name": "vm1",
        "Options": null,
        "Scope": "local"
    }
]
```

ボリュームを作成すると、ホスト側の**/var/lib/docker/volumes**ディレクトリ配下に新たに**vm1**ディレクトリが作成され、このディレクトリとコンテナ内がマウントされます。
```bash
$ sudo ls -ltr /var/lib/docker/volumes/vm1/
total 0
drwxr-xr-x. 2 root root 6 Jan 31 21:07 _data
```

ボリュームが作成されたので、コンテナを作成し、ボリュームマウントしていきます。コンテナにボリュームをマウントさせるためには、**docker container run**コマンドに「--mount」オプションを付与します。「--mount」オプションには、いくつか引数をとります。**type**には、マウントの種類を指定します。今回はボリュームマウントのため、**type=volume**を指定します。**src**には、作成したボリューム名を指定します。今回は**src=vm1**を指定します。**target**には、マウントに利用するコンテナのフォルダを指定します。今回は**target=/home**としました。
```bash
$ docker container run -dit --name alpine-linux1 --mount type=volume,src=vm1,target=/home alpine
9d08bc94b020ffb145fb958249c6206e6a271cce095501db47534175dfe68ef5
```

ボリュームマウントをしてコンテナを起動したあと、コンテナ内に入り、マウントしたディレクトリ配下で**test.txt**ファイルを作成し、コンテナから抜けます。その後、コンテナを停止・削除した後、再度ボリュームマウントをしてコンテナを起動し先ほど作成したファイルを確認すると無事に**test.txt**ファイルが確認できました。
```bash
$ docker container exec -it alpine-linux1 /bin/sh
/ # cd /home
/home # touch test.txt
/home # ls test.txt
test.txt
/home # exit

$ docker container stop alpine-linux1
alpine-linux1

$ docker container rm alpine-linux1
alpine-linux1

$ docker container run -dit --name alpine-linux1 --mount type=volume,src=vm1,target=/home alpine
f4647b31d7eead2d4a24dc9306dec81989446816814b397edb4dcac5d23482b2

$ cd /home

$ docker container exec -it alpine-linux1 /bin/sh
/ # cd /home
/home # ls test.txt 
test.txt
```

Docker によって管理されているホストファイルシステム上（**/var/lib/docker/volumes**ディレクトリ配下）を確認すると、コンテナ内で作成した**test.txt**ファイルが存在することが確認できます。よって、コンテナ内で作成されたファイルはDocker によって管理されているマウント先に存在することになり、コンテナが削除され、再度別のコンテナを起動しても同じマウント先を指定することにより永続化することができます。
```bash
$ sudo ls -ltr /var/lib/docker/volumes/vm1/_data
total 0
-rw-r--r--. 1 root root 0 Jan 31 21:26 test.txt
```

![volume](/images/docker-network-20250202/docker-volume01.drawio.png)

作成したボリュームは、**docker volume rm**コマンドを利用することで削除することができます。
```bash
$ docker volume ls
DRIVER    VOLUME NAME
local     vm1

$ docker volume rm vm1
vm1

$ docker volume ls
DRIVER    VOLUME NAME
```

### bind mount
bindマウントは、ホストマシン上のディレクトリをコンテナにマウントします。volumeマウントを使用することが推奨されていますが、コンテナに設定ファイルを即時反映させたい場合などにbindマウントは利用されます。
bindマウントを利用したコンテナを作成してみましょう。
ホストマシン上に作業ディレクトリ**work**を作成し、workディレクトリ内に**text.txt**ファイルを作成しておきます。次にホストマシン上のworkディレクトリとコンテナのhomeディレクトリをbindマウントさせて起動します。
bindマウントさせるためには、volumeマウントと同様に**docker container run**コマンドに「--mount」オプションを付与します。「--mount」オプションには、いくつか引数をとります。**type**には、マウントの種類を指定します。今回はボリュームマウントのため、**type=bind**を指定します。**src**には、作成したボリューム名を指定します。今回は**src=$(pwd)**を指定します。**target**には、マウントに利用するコンテナのフォルダを指定します。今回は**target=/home**としました。

```bash
$ mkdir work

$ cd work

$ echo "Hello" >> text.txt

$ cat text.txt 
Hello

$ docker container run -it --name alpine-linux --mount type=bind,src="$(pwd)",target=/home alpine
/ # ls /home
text.txt
/ # cat /home/text.txt 
Hello
```

![volume](/images/docker-network-20250202/docker-bind01.drawio.png)

ホストマシン上で作成したファイルがコンテナからも確認することができました。このようにbindマウントを使うとホストマシンの好きなディレクトリとコンテナを接続させることができます。


### tmpfs
tmpfsマウントは、Linuxホストマシン上のメモリ空間上にマウントします。ホストマシン上の**メモリ**を利用するためアクセスは高速になりますが、コンテナを停止したり、ホストマシン上も停止や再起動をするとデータが消えてしまいます。
tmpfsマウントさせるためには、**docker container run**コマンドに「--mount」オプションに**type=tmpfs**を指定します。**target**は、今回も**target=/home**としました。コンテナを起動後、コンテナ内で**df -h**コマンドを実行すると**/home**配下がtmpfsをマウントしていることがで確認できます。
```bash
$ docker container run -dit --name alpine-linux1 --mount type=tmpfs,target=/home alpine
c1ce74087a6284c6f2dafa849aaf1c562adb6353de2cbffc9226d56d72a5e9f4

$ docker container exec -it alpine-linux1 /bin/sh
/ # df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                  19.7G      3.7G     16.1G  19% /
tmpfs                    64.0M         0     64.0M   0% /dev
shm                      64.0M         0     64.0M   0% /dev/shm
tmpfs                     1.8G         0      1.8G   0% /home
/dev/sda2                19.7G      3.7G     16.1G  19% /etc/resolv.conf
/dev/sda2                19.7G      3.7G     16.1G  19% /etc/hostname
/dev/sda2                19.7G      3.7G     16.1G  19% /etc/hosts
tmpfs                     1.8G         0      1.8G   0% /proc/acpi
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/keys
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                     1.8G         0      1.8G   0% /proc/scsi
tmpfs                     1.8G         0      1.8G   0% /sys/firmware
```

Dockerのデータ管理について解説を行ってきました。マウント方法としては**volume**、**bind**、**tmpfs**と三種類ありますが、基本的には**volume**を使う方がよいでしょう。

# 参考
https://docs.docker.jp/engine/reference/commandline/volume_toc.html
https://docs.docker.jp/storage/toc.html
https://docs.docker.jp/storage/volumes.html
https://docs.docker.jp/storage/bind-mounts.html
https://docs.docker.jp/storage/tmpfs.html