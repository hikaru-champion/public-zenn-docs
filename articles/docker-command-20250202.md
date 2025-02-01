---
title: "【Docker再入門】～Dockerの基本コマンド～"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["コンテナ", "Docker"]
published: true
---
# はじめに
こんにちは。championです。
普段は、Google CloudやAWSを中心としたクラウドインフラの設計～保守運用を行なっています。

前回の記事では、Dockerの概要の紹介とDockerをLinuxホストマシン上にインストールしていきました。
https://zenn.dev/hi_ka_ru/articles/docker-20250125

今回は、インストールしたDockerを実際に動作させてコンテナの基本コマンドについて記載したいと思います。

# Dockerの基本コマンド
## コンテナイメージの管理
Dockerコンテナを作成するためには、コンテナのもととなるイメージが必要です。コンテナイメージがホストしてあるパブリックレジストリの**Docker Hub**からイメージを取得してみましょう。

### コンテナイメージの取得
**docker image pull**コマンドを利用することで、イメージをローカルに取得することができます。**Docker Hub**からnginxのコンテナイメージを取得してみましょう。
```bash
$ docker image pull nginx
Using default tag: latest
latest: Pulling from library/nginx
af302e5c37e9: Pull complete 
207b812743af: Pull complete 
841e383b441e: Pull complete 
0256c04a8d84: Pull complete 
38e992d287c5: Pull complete 
9e9aab598f58: Pull complete 
4de87b37f4ad: Pull complete 
Digest: sha256:0a399eb16751829e1af26fea27b20c3ec28d7ab1fb72182879dcae1cca21206a
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```
コンテナイメージが取得できました。

### コンテナイメージの確認
**docker image ls**コマンドを利用することで、ローカルの取得したコンテナイメージを確認することができます。
先ほど取得したnginxのコンテナイメージが確認できます。
```bash
$ docker image ls -a
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    9bea9f2796e2   2 months ago   192MB
```

### コンテナイメージの詳細確認
**docker image inspect**コマンドを利用することで、コンテナイメージの詳細情報を確認することができます。
先ほど取得したnginxのコンテナイメージの詳細情報を確認できます。
```bash
$ docker image inspect nginx
[
    {
        "Id": "sha256:9bea9f2796e236cb18c2b3ad561ff29f655d1001f9ec7247a0bc5e08d25652a1",
        "RepoTags": [
            "nginx:latest"
        ],
        "RepoDigests": [
            "nginx@sha256:0a399eb16751829e1af26fea27b20c3ec28d7ab1fb72182879dcae1cca21206a"
        ],
        "Parent": "",
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2024-11-26T18:42:08Z",
        "DockerVersion": "",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.27.3",
                "NJS_VERSION=0.8.7",
                "NJS_RELEASE=1~bookworm",
                "PKG_RELEASE=1~bookworm",
                "DYNPKG_RELEASE=1~bookworm"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 191717838,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/c62106fa57349d94eba4e963b21800c22f8a52eab4590791913f1e7b8267b19d/diff:/var/lib/docker/overlay2/e5e14a96d8c8b642b1fc7d10fba4fc2cd655393d064110f1586df8da55df67e3/diff:/var/lib/docker/overlay2/1acda929ef130c2d441400007789f5a200c28f44cc7a329d69af92a5a34b4b26/diff:/var/lib/docker/overlay2/a97d6d46ca7cfdc6356a17cf13e50926900c1447e50a780a3aa7da5076d4a119/diff:/var/lib/docker/overlay2/f647c002739a739b7ab5a917f9dac86d0a92b578fa2148435279a686cb44cc9f/diff:/var/lib/docker/overlay2/f0bbaa46d35f96c04648c3200aa7bc376cd762a90f565725fcdd8fb51fb9c6cf/diff",
                "MergedDir": "/var/lib/docker/overlay2/9af8df103a8f7396f513e37bfa98ed46a2f77d1fcba4984756c94ed221f7da65/merged",
                "UpperDir": "/var/lib/docker/overlay2/9af8df103a8f7396f513e37bfa98ed46a2f77d1fcba4984756c94ed221f7da65/diff",
                "WorkDir": "/var/lib/docker/overlay2/9af8df103a8f7396f513e37bfa98ed46a2f77d1fcba4984756c94ed221f7da65/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:f5fe472da25334617e6e6467c7ebce41e0ae5580e5bd0ecbf0d573bacd560ecb",
                "sha256:88ebb510d2fb3ed50f4268455a38443074cad5c6957f6d2cd0126c899a159e6e",
                "sha256:9431321431991c4e64246007e04602160bca8984439ff96461fb992072dd49af",
                "sha256:32c977818204dc910140b3b7abcad06a6613dd1f511ce8f33626a364a3bb68b6",
                "sha256:541cf9cf006d6c9920e5897bf63a4dce0ae1a8388bc82bfa1abedc48b8eb1de9",
                "sha256:58045dd06e5b2c1220ab200c36b450ce3adbfd3fa0f8e3c2c17ffaf7f2906455",
                "sha256:b57b5eac2941c7c1f1f5bc2391123e553d0082eb8e1c6675101aab47a11b26ee"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

### コンテナイメージの履歴の確認
**docker image history**コマンドを利用することで、コンテナイメージの履歴情報を確認することができます。
先ほど取得したnginxのコンテナイメージの履歴情報を確認できます。
```bash
$ docker image history nginx
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
9bea9f2796e2   2 months ago   CMD ["nginx" "-g" "daemon off;"]                0B        buildkit.dockerfile.v0
<missing>      2 months ago   STOPSIGNAL SIGQUIT                              0B        buildkit.dockerfile.v0
<missing>      2 months ago   EXPOSE map[80/tcp:{}]                           0B        buildkit.dockerfile.v0
<missing>      2 months ago   ENTRYPOINT ["/docker-entrypoint.sh"]            0B        buildkit.dockerfile.v0
<missing>      2 months ago   COPY 30-tune-worker-processes.sh /docker-ent…   4.62kB    buildkit.dockerfile.v0
<missing>      2 months ago   COPY 20-envsubst-on-templates.sh /docker-ent…   3.02kB    buildkit.dockerfile.v0
<missing>      2 months ago   COPY 15-local-resolvers.envsh /docker-entryp…   389B      buildkit.dockerfile.v0
<missing>      2 months ago   COPY 10-listen-on-ipv6-by-default.sh /docker…   2.12kB    buildkit.dockerfile.v0
<missing>      2 months ago   COPY docker-entrypoint.sh / # buildkit          1.62kB    buildkit.dockerfile.v0
<missing>      2 months ago   RUN /bin/sh -c set -x     && groupadd --syst…   117MB     buildkit.dockerfile.v0
<missing>      2 months ago   ENV DYNPKG_RELEASE=1~bookworm                   0B        buildkit.dockerfile.v0
<missing>      2 months ago   ENV PKG_RELEASE=1~bookworm                      0B        buildkit.dockerfile.v0
<missing>      2 months ago   ENV NJS_RELEASE=1~bookworm                      0B        buildkit.dockerfile.v0
<missing>      2 months ago   ENV NJS_VERSION=0.8.7                           0B        buildkit.dockerfile.v0
<missing>      2 months ago   ENV NGINX_VERSION=1.27.3                        0B        buildkit.dockerfile.v0
<missing>      2 months ago   LABEL maintainer=NGINX Docker Maintainers <d…   0B        buildkit.dockerfile.v0
<missing>      2 months ago   # debian.sh --arch 'amd64' out/ 'bookworm' '…   74.8MB    debuerreotype 0.15
```

### コンテナイメージへのタグ付け
**docker image tag**コマンドを利用することで、コンテナイメージへタグを付与することができます。
先ほど取得したnginxのコンテナイメージへv1タグを付与してみます。
```bash
$ docker image tag nginx:latest nginx:v1
$ docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    9bea9f2796e2   2 months ago   192MB
nginx        v1        9bea9f2796e2   2 months ago   192MB
```
latestタグとは別のv1タグが付与されたコンテナイメージが作成できました。タグは別物ですが、IMAGE IDが同一なので同じコンテナイメージになります。

### コンテナイメージの削除
**docker image rm**コマンドを利用することで、特定のコンテナイメージや複数のコンテナイメージを削除することができます。
```bash
$ docker image rm nginx
Untagged: nginx:latest
$ docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        v1        9bea9f2796e2   2 months ago   192MB
```
latestタグが付与されたコンテナイメージが削除されました。

また、**docker image prune**コマンドを利用することで、使用されていないコンテナイメージをすべて削除することができます。
```bash
$ docker image prune -a
WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y
Deleted Images:
untagged: nginx:v1
untagged: nginx@sha256:0a399eb16751829e1af26fea27b20c3ec28d7ab1fb72182879dcae1cca21206a
deleted: sha256:9bea9f2796e236cb18c2b3ad561ff29f655d1001f9ec7247a0bc5e08d25652a1
deleted: sha256:ad858b4eee2d8f479065d1b8c650e13c815e75b2e4503c05a67cdc8880b0975a
deleted: sha256:a8d62a6e94dbe8920ecfafa3b3aff0ef70aa26ca929f90b110db998eec09bab5
deleted: sha256:8ad081ee0d46efdea0492cd0bb527c50a8891276fd408ec1269522ac9d6d5580
deleted: sha256:d9ff8651e34fffb11d2a29317ec8404dcd4481bfcc1883b495113365dba8a461
deleted: sha256:d351808aa3da5992f01a613745394c289e51a673d7809800322475b413e627e4
deleted: sha256:5ec183755d09a0a54d0f83e5a705f28f1606fe75660c13148fe125e4f12af39f
deleted: sha256:f5fe472da25334617e6e6467c7ebce41e0ae5580e5bd0ecbf0d573bacd560ecb

Total reclaimed space: 191.7MB
$ docker image ls
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```


## コンテナの管理
**docker container**コマンドは、コンテナを管理する基本コマンドです。

### コンテナの作成
コンテナを作成するためには、**docker container create**コマンドを実行します。
取得したnginxのコンテナイメージからnginxコンテナを作成してみましょう。
```bash
$ docker container create -it --name nginx nginx
7b133740fabbd2225449d081ab045a2ce46b70e16502379fb4e8e59c3fa40b82
```
この段階では、コンテナが作成されただけで起動はしていません。

### コンテナの確認
コンテナを一覧表示して確認するためには、**docker container ls**コマンドを実行します。
デフォルトでは、実行中のコンテナのみ表示するので、すべてのコンテナを表示するためには「-a」オプションを付与します。コンテナのSTATUSが**Created**になっています。
```bash
$ docker container ls -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS    PORTS     NAMES
7b133740fabb   nginx     "/docker-entrypoint.…"   27 seconds ago   Created             nginx
```

### コンテナの起動
作成したコンテナを起動するためには、**docker container start**コマンドを実行します。
先ほど作成したnginxコンテナを起動します。コンテナのSTATUSが**Up**になっており起動したことが確認できます。
```bash
$ docker container start nginx
nginx

$ docker container ls
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
7b133740fabb   nginx     "/docker-entrypoint.…"   5 minutes ago   Up 5 seconds   80/tcp    nginx
```

### コンテナの一時停止
作成したコンテナの一時停止をするためには、**docker container pause**コマンドを実行します。
先ほど作成したnginxコンテナを一時停止します。STATUSが**Paused**になっていることが確認できます。
```bash
$ docker container pause nginx
nginx

$ docker container ls
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                  PORTS     NAMES
7b133740fabb   nginx     "/docker-entrypoint.…"   9 minutes ago   Up 3 minutes (Paused)   80/tcp    nginx
```

### コンテナの一時停止の解除
作成したコンテナの一時停止を解除するためには、**docker container unpause**コマンドを実行します。
先ほど作成したnginxコンテナの一時停止を解除します。STATUSの**Paused**がなくなっていることが確認できます。

```bash
$ docker container unpause nginx
nginx
$ docker container ls
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS     NAMES
7b133740fabb   nginx     "/docker-entrypoint.…"   15 minutes ago   Up 9 minutes   80/tcp    nginx
```

### コンテナのログ確認
起動しているコンテナのログを確認するためには、**docker container logs**コマンドを実行します。
先ほど作成したnginxコンテナのログを確認します。nginxコンテナの起動ログ等が確認できました。

```bash
$ docker container logs nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/01/28 01:04:27 [notice] 1#1: using the "epoll" event method
2025/01/28 01:04:27 [notice] 1#1: nginx/1.27.3
2025/01/28 01:04:27 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14) 
2025/01/28 01:04:27 [notice] 1#1: OS: Linux 5.14.0-503.21.1.el9_5.x86_64
2025/01/28 01:04:27 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1073741816:1073741816
2025/01/28 01:04:27 [notice] 1#1: start worker processes
2025/01/28 01:04:27 [notice] 1#1: start worker process 29
2025/01/28 01:04:27 [notice] 1#1: start worker process 30
```

### コンテナ内でのコマンド実行
起動しているコンテナ内でコマンドを実行するためには、**docker container exec**コマンドを実行します。
先ほど作成したnginxコンテナ内で**ls**コマンドを実行してみます。
```bash
$ docker container exec nginx /bin/ls
bin
boot
dev
docker-entrypoint.d
docker-entrypoint.sh
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

### コンテナの詳細情報の確認
起動しているコンテナの詳細情報を確認するためには、**docker container inspect**コマンドを実行します。
先ほど作成したnginxコンテナの詳細情報を確認してみます。
```bash
$ docker container inspect nginx
[
    {
        "Id": "7b133740fabbd2225449d081ab045a2ce46b70e16502379fb4e8e59c3fa40b82",
        "Created": "2025-01-28T00:58:55.588075253Z",
        "Path": "/docker-entrypoint.sh",
        "Args": [
            "nginx",
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 8712,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2025-01-28T01:04:26.813199789Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:9bea9f2796e236cb18c2b3ad561ff29f655d1001f9ec7247a0bc5e08d25652a1",
        "ResolvConfPath": "/var/lib/docker/containers/7b133740fabbd2225449d081ab045a2ce46b70e16502379fb4e8e59c3fa40b82/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/7b133740fabbd2225449d081ab045a2ce46b70e16502379fb4e8e59c3fa40b82/hostname",
        "HostsPath": "/var/lib/docker/containers/7b133740fabbd2225449d081ab045a2ce46b70e16502379fb4e8e59c3fa40b82/hosts",
        "LogPath": "/var/lib/docker/containers/7b133740fabbd2225449d081ab045a2ce46b70e16502379fb4e8e59c3fa40b82/7b133740fabbd2225449d081ab045a2ce46b70e16502379fb4e8e59c3fa40b82-json.log",
        "Name": "/nginx",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "bridge",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                39,
                112
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": [],
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware",
                "/sys/devices/virtual/powercap"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/29cb462ba0ae5b7700fe028003880be6fe7f9b1f56b65e5ae8dae65272608cbf-init/diff:/var/lib/docker/overlay2/a67c2c59f9fd17db2d1900a928231302fb5e1f2c7dfc8df59cbd14ca495771a9/diff:/var/lib/docker/overlay2/6d814e8d9628a52eeded47e8f7572b68e21de24b47c169490fbb859ddb196b56/diff:/var/lib/docker/overlay2/8a5b34ed2e7703191b20a96d5ec387488842c043559834bfbc3d381991b3dcaf/diff:/var/lib/docker/overlay2/41586b8716f54372dd795e8b6ac8caa9c3f01082e14aeed9a6c7330fa9474940/diff:/var/lib/docker/overlay2/76ddb28f01bdcae287544c511af70941e241bb4418a9372a639ed5818fc3208e/diff:/var/lib/docker/overlay2/baf066c30d879eefda95f4586b80278efa3396650e9c92dcc7f1242c07ae9a32/diff:/var/lib/docker/overlay2/c95ee7542ec60a44101d4fca283bf13e0de284e40853b29f21e2ad4a27b58d15/diff",
                "MergedDir": "/var/lib/docker/overlay2/29cb462ba0ae5b7700fe028003880be6fe7f9b1f56b65e5ae8dae65272608cbf/merged",
                "UpperDir": "/var/lib/docker/overlay2/29cb462ba0ae5b7700fe028003880be6fe7f9b1f56b65e5ae8dae65272608cbf/diff",
                "WorkDir": "/var/lib/docker/overlay2/29cb462ba0ae5b7700fe028003880be6fe7f9b1f56b65e5ae8dae65272608cbf/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "7b133740fabb",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.27.3",
                "NJS_VERSION=0.8.7",
                "NJS_RELEASE=1~bookworm",
                "PKG_RELEASE=1~bookworm",
                "DYNPKG_RELEASE=1~bookworm"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "nginx",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "881af4be43ceb546e0b79294fa651314b1c7debab017c2647793ede3baea9ff1",
            "SandboxKey": "/var/run/docker/netns/881af4be43ce",
            "Ports": {
                "80/tcp": null
            },
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "83407366cd373fb662077b8a9af58469fcb0ed64c6e0284e74c29cc594511713",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null,
                    "NetworkID": "3d66a57a65ad26769b11bb624ddd8d89b50ee8c825b9c465b12a387dcdafefc6",
                    "EndpointID": "83407366cd373fb662077b8a9af58469fcb0ed64c6e0284e74c29cc594511713",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": null
                }
            }
        }
    }
]
```

### コンテナの停止
起動しているコンテナを停止するためには、**docker container stop**コマンドを実行します。
先ほど作成したnginxコンテナの停止してみます。STATUSが**Exited**に変更されていることがわかります。

```bash
$ docker container stop nginx
nginx

$ docker container ls -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                     PORTS     NAMES
7b133740fabb   nginx     "/docker-entrypoint.…"   40 minutes ago   Exited (0) 6 seconds ago             nginx
```

### コンテナの削除
停止したコンテナも削除するまでは、ローカルに残ってしまいます。停止済みのコンテナを削除するには**docker container rm**コマンドを実行します。
先ほど停止したnginxコンテナを削除してみます。

```bash
$ docker container rm nginx
nginx

$ docker container ls -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### コンテナの実行
先ほどまで以下の手順でコンテナの説明していましたが、
- イメージの取得
- コンテナの作成
- コンテナの起動

**docker container run**コマンドを実行することで、上記の手順を一つのコマンドで実行してくれます。nginxコンテナを**docker container run**コマンドで起動まで行っていきましょう。この時オプションでしている「-i」は、コンテナへ標準入力を行えるようにするオプションです。「-d」は、デタッチモードで起動＝バックグラウンドプロセスで起動するよう設定するオプションです。「-t」はコンテナに疑似ttyを割り当てるオプションです。これにより標準入力を行うことができます。また、「-p」オプションにより、ホスト側のポートとコンテナのポートをマッピングしています。今回はホスト側の8080ポートをコンテナのポート80へマッピングしています。つまり、ホスト側のポート8080へアクセスすると、コンテナの80ポートへフォワーディングされます。

```bash
$ docker container run -dit --name nginx -p 8080:80 nginx
2a42e67572a63ced9bc354ad2463a85bad8dec427d0598d0f777e207d1aedfdd

$ docker container ls
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                     NAMES
2a42e67572a6   nginx     "/docker-entrypoint.…"   3 seconds ago   Up 3 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   nginx
```

起動したnginxコンテナへアクセスしてみます。ホスト側の8080ポートへcurlコマンドを実行してみると、nginxのトップページのhtmlが返却されたことが確認できます。
```bash
$ curl localhost:8080
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

Dockerの基本コマンドについて実際にイメージの取得からコンテナの作成～削除まで行なっていきました。
次回はDockerのネットワークについて説明していきます。


# 参考
https://docs.docker.jp/engine/reference/index.html