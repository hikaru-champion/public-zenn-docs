---
title: "【Google Cloud】Cloud Buildを利用したCloud Runをデプロイ"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GoogleCloud", "CloudRun", "CloudBuild"]
published: false
---
# はじめに
こんにちは。とある事業会社でGoogle Cloudを扱っているHikaruです。
Google Cloudの認定資格の一つである「professional cloud developer」の受験を控えているので、試験範囲でもある以下のサービスについて実際に手を動かして理解していきたいと思います。
- Cloud Source Repositories
- Cloud Build
- Artifact Registory
- Cloud Run

以下の認定資格についてはすでに取得済みです。
- Associate Cloud Engineer
- Professional Cloud Architect
- Professional Cloud Network Engineer

# Cloud Source Repositories
## Cloud Source Repositoriesとは
Cloud Source Repositoriesとは、Google Cloudが提供するプライベートGitリポジトリです。
Google Cloud上でGitを使ったソースコードのバージョン管理を行うことができます。一般的なGitコマンドに対応しているため、普段Githubをお使いの方は、Githubと同じ感覚で利用することができます。
Cloud BuildとのCI/CD連携やApp Engine/Cloud Functionsへの直接デプロイ等Google Cloudの他のサービスとの連携に強みがあります。

## 設定してみる
Google Cloudのコンソール画面の検索欄から「Cloud Source Repositories」と入力し、遷移します。
リポジトリの追加の設定で「新しいリポジトリを作成」をクリックします。
![](/images/cloudrun-build-deploy-20240511/CloudSourceRepositories01.png)

- リポジトリ名：sample-application
- プロジェクト：自身のプロジェクト
を入力し、作成をクリックします。
![](/images/cloudrun-build-deploy-20240511/CloudSourceRepositories02.png)

「ローカルGitリポジトリへのリポジトリのクローンを作成する」を選択し、認証方法として以下の3つから選択します。
- SSH認証
- Google Cloud SDK
- 手動で作成した認証情報
今回は「Google Cloud SDK」を選択しました。

![](/images/cloudrun-build-deploy-20240511/CloudSourceRepositories03.png)

Cloud Source Repositoriesで作成したリポジトリをローカルにクローンしたら、
空のDockerfileを作成し、pushしてみます。
```
cd sample-application
touch Dockerfile
git add Dockerfile
git commit -m "init"
git push origin master
```
上記の作業が完了すると、Cloud Source RepositoriesにDockerfileがPushされていることが確認できました。

![](/images/cloudrun-build-deploy-20240511/CloudSourceRepositories04.png)

## 課題感
Cloud Source Repositoriesを使用してみての課題感ですが、Githubと比較して以下が不足しています。
- UIが見ずらい
- プルリクエストに対応していない
- Issue管理機能がない
このため、チームでの共同開発にはCloud Source Repositoriesの利用はおすすめできないと思います。
チームでの共同開発をするのであれば、Githubを利用した方が良いですね。

# Artifact Registory
## Artifact Registoryとは

## Artifact Analysis
Artifact Analysisとは、Artifact Registoryに格納されたDockerイメージをスキャンし、イメージないに含まれる脆弱性をレポートとして出力してくれる機能になります。以前は、Container Analysisと呼ばれていました。

## Binary Authorization

## 設定してみる
Artifact Registryのコンソール画面から「リポジトリを作成」をクリックします。
- 名前：nginx-repo
- 形式：Docker
- モード：標準
- ロケーションタイプ：asia-northeast1
- 説明：nginxイメージ用のリポジトリ
- 暗号化：Googleが管理する暗号鍵
- 不変のイメージタグ：無効
- クリーンアップポリシー：ドライラン

![](/images/cloudrun-build-deploy-20240511/ArtifactRegistory01.png)

「作成」をクリックすると、リポジトリが作成されます。
※ほぼデフォルト値で構築します。
![](/images/cloudrun-build-deploy-20240511/ArtifactRegistory02.png)

nginx-repoリポジトリが作成されました。
![](/images/cloudrun-build-deploy-20240511/ArtifactRegistory03.png)

脆弱性スキャンの設定もオンにします。
左のタブから「設定」をクリックすると、脆弱性スキャンの設定ができます。
スキャンがデフォルトでは無効になっているため、オンにします。
![](/images/cloudrun-build-deploy-20240511/ArtifactRegistory04.png)

# Cloud Build
## Cloud Buildとは
Cloud Buildとは、Google Cloud上でビルドを行なうことのできるフルマネージドサービスです。
フルマネージドのため、ユーザがビルド環境を用意・運用する必要はありません。
Cloud Buildではyamlまたはjson形式のビルド構成ファイルを作成する必要があります。
ビルド構成ファイルに記述された指示によって、Cloud Build上でビルドが実行されます。ビルドを実行するためには、コンソールでの手動実行やAPIによる実行の他にも、Cloud Source Repositories、Github、Bitbucketなど多数のコードリポジトリと連携し、ソースコードに変更があったことをトリガーにCloud Buildを実行してビルドを自動化できます。

## Cloud Source RepositryとCloud Buildとの連携
先ほど作成した、Cloud Source RepositryとCloud Buildを連携したいと思います。
Cloud Source RepositryCloud Source RepositryCloud Source RepositryCloud Source RepositryCloud Source Repositryと同じプロジェクトでCloud Buildの設定を行う場合、リポジトリの接続が既に完了していますので、追加の設定は必要ありません。GithubやGitlabなどサードパーティ製のリポジトリと連携する場合はリポジトリを接続する設定が別途必要になります。
![](/images/cloudrun-build-deploy-20240511/CloudBuild01.png)

## トリガーの設定
トリガーを設定するためには、sample-applicationのリポジトリから、「トリガーを追加」
をクリックします。
![](/images/cloudrun-build-deploy-20240511/CloudBuild02.png)


トリガーの名前やイベントなどを任意の値に設定し、ソースに Cloud Source Repositry のリポジトリを選択します。
ここでは、masterブランチへのプッシュを検知してトリガーが発火するように設定します。
CloudBuild構成ファイルでビルドステップを制御したいので、形式は「Cloud Build構成ファイル」を選択し、Cloud Build構成ファイルの配置場所は、Cloud Source Repositryの「/cloudbuild.yaml」にします。
今回は検証のため、サービスアカウントはデフォルトのCloudBuildサービスアカウントを利用します。
※本番で利用される場合は、最小権限の原則にのっとるようにカスタムサービスアカウントを利用してください。


![](/images/cloudrun-build-deploy-20240511/CloudBuild03.png)


![](/images/cloudrun-build-deploy-20240511/CloudBuild04.png)


![](/images/cloudrun-build-deploy-20240511/CloudBuild05.png)


![](/images/cloudrun-build-deploy-20240511/CloudBuild06.png)

## cloudbuild.yamlの作成
cloudBuild構成ファイルのcloudbuild.yamlでビルドステップを制御します。

```cloudbuild.yaml
steps:
# Docker Build
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'asia-northeast1-docker.pkg.dev/${PROJECT_ID}/nginx-repo/nginx', '.' ]

# Docker Push
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'asia-northeast1-docker.pkg.dev/${PROJECT_ID}/nginx-repo/nginx']

```

# Cloud Run
## Cloud Runとは


## 初期イメージのデプロイ


# 継続的デプロイ

## cloudbuild.yamlの変更

```cloudbuild.yaml
steps:
# Docker Build
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'asia-northeast1-docker.pkg.dev/${PROJECT_ID}/nginx-repo/nginx', '.' ]

# Docker Push
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'asia-northeast1-docker.pkg.dev/${PROJECT_ID}/nginx-repo/nginx']

# Cloud Run Deploy
- name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
  entrypoint: gcloud
  args:
  - 'run'
  - 'deploy'
  - 'nginx'
  - '--image'
  - 'asia-northeast1-docker.pkg.dev/${PROJECT_ID}/nginx-repo/nginx:latest'
  - '--region'
  - 'asia-northeast1'
images:
- 'asia-northeast1-docker.pkg.dev/${PROJECT_ID}/nginx-repo/nginx:latest'
```

## nginxの設定変更

```index.html
<h1>Hello Nginx</h1>
```

## dockerfileの変更

```dockerfile
FROM nginx:latest

COPY html/index.html /usr/share/nginx/html
```

## 継続的デプロイの実行
```
watanabehikaru@UBUNTU:~/sample-application$ git add .
watanabehikaru@UBUNTU:~/sample-application$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   Dockerfile
        modified:   cloudbuild.yaml
        new file:   html/index.html

watanabehikaru@UBUNTU:~/sample-application$ git commit -m "add index.html"
[master a9bd436] add index.html
 3 files changed, 18 insertions(+), 1 deletion(-)
 create mode 100644 html/index.html
watanabehikaru@UBUNTU:~/sample-application$ git push origin master
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 20 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 711 bytes | 711.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Waiting for private key checker: 3/3 objects left
To https://source.developers.google.com/p/deep-thought-422207-j1/r/sample-application
   c97bc67..a9bd436  master -> master
```