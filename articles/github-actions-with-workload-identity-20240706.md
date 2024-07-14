---
title: "Workload Identityを利用してGitHub ActionsからTerraform実行"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GoogleCloud","GitHubActions","Terraform"]
published: true
---

# 初めに
こんにちは。hikaruです。
今回は、Workload Identityを利用してGitHub ActionsからTerraformを実行するまでをハンズオン形式で記載します。
外部リソースからGoolge Cloudへアクセスする場合には、サービスアカウントキーを利用する必要がありましたが、Workload Identityを利用することで、サービスアカウントキーを発行することなくGoolge Cloudへアクセスすることができます。チーム開発をするうえで必要不可欠なGitHub上のサービスである、**GitHub Actions**をWorkload Identityを用いて利用することで、いままで手動で実行していたterraformコマンドを自動化し、セキュアにGoogle Cloudへアクセスしたいと思います。


# Workload Identityとは
AWSやAzureなどの外部のワークロードや、GitHubにGoogle Cloudのサービスアカウントの権限を借用することでアクセス権を付与することができるサービスです。サービスアカウントの**サービスアカウントキーを発行する必要がない**ため、セキュリティ上のリスクも最小限に抑えることができます。

Workload Identityを利用したGitHub Actionsの概要図としては以下の通りです。

![](/images/github-actions-with-workload-identity-20240706/github-actions-with-workload-identity.png)
以下の公式ブログから引用
> https://cloud.google.com/blog/ja/products/identity-security/enabling-keyless-authentication-from-github-actions


# Workload Identityを利用するにあたっての準備
## 必要なAPIの有効化
Workload Identityを利用するにあたって、必要なAPIは事前に有効化しておきましょう。
Cloud Shellにアクセスし、以下のコマンドを実行することで有効化することができます。

```
gcloud services enable \
  iam.googleapis.com \
  cloudresourcemanager.googleapis.com \
  iamcredentials.googleapis.com \
  sts.googleapis.com
```

## サービスアカウントの作成
GitHub ActionsからGoogle Cloudリソースを操作するためのサービスアカウントを作成します。
```
gcloud iam service-accounts create github-actions-cicd \
  --project="${GOOGLE_CLOUD_PROJECT}"
```

## サービスアカウントへのロールの付与
上記で作成したサービスアカウントへIAMロールを付与します。
今回は、検証用ということで基本ロールであるowner権限を付与していますが、
本番で利用される場合は**必要最低限の権限**だけを付与してください。
```
gcloud projects add-iam-policy-binding "${GOOGLE_CLOUD_PROJECT}" \
  --member="serviceAccount:github-actions-cicd@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com" \
  --role="roles/owner"
```

## Workload Identity Pool / Provider の作成
Workload Identity PoolとWorkload Identity Providerを作成します。
Workload Identity Poolは、**外部IDを管理できる**エンティティです。複数のOIDC IDプロバイダを所属させることが可能で、Poolに対してサービスアカウントを借用する許可を与えます。
Workload Identity Providerは、**Google Cloud と IdP の間の関係を記述する**エンティティです。

```
gcloud iam workload-identity-pools create "github-actions-cicd-pool" \
  --project="${GOOGLE_CLOUD_PROJECT}" \
  --location="global" \
  --display-name="github-actions-cicd-pool" \
  --description="github-actions with workload identity"

 gcloud iam workload-identity-pools providers create-oidc "github-actions" \
  --project="${GOOGLE_CLOUD_PROJECT}" \
  --location="global" \
  --workload-identity-pool="github-actions-cicd-pool" \
  --display-name="github-actions provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"
```

後の設定で利用するため、
上記で作成したWorkload Identity PoolのIDと、自身のGitHubリポジトリを環境変数にエクスポートします。
```
 export WORKLOAD_IDENTITY_POOL_ID=$( \
    gcloud iam workload-identity-pools describe "github-actions-cicd-pool" \
      --project="${GOOGLE_CLOUD_PROJECT}" --location="global" \
      --format="value(name)" \
  )
 export GITHUB_REPO="xxxxxxxxxxxxxxx"
```

## Workload Identity Poolにサービスアカウントの権限借用の設定
これまでに作成した、Workload Identity Poolにサービスアカウントの権限借用の設定を行います。
ここでmemberに指定している値は、自分のGitHubの特定のリポジトリのみを指定しています。
```
gcloud iam service-accounts add-iam-policy-binding "github-actions-cicd@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com" \
  --project="${GOOGLE_CLOUD_PROJECT}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${GITHUB_REPO}"
```
ここまでで、Workload Identityを利用するにあたっての事前準備は完了です。

# GitHub Actionsのワークフロー定義
GitHubリポジトリの`.github/workflows/`にGitHub Actionsで利用するためのワークフロー定義ファイルを配置します。

:::message
`.github/workflows/`以外に定義ファイルを配置した場合は、GitHub Actionsが実行されないので注意してください。
:::

## terraform_ci.yaml

今回は、
- GitHubリポジトリの`featureブランチにPush`した時
- `mainブランチにPull Request`をした時

に`terraform plan`を実行してどのGoogle Cloudリソースが作成されるかを確認したいので、
ワークフロー定義ファイル`terraform_ci.yaml`を作成しました。

```yaml:terraform_ci.yaml
name: Terraform CI
on:
  push:
    branches:
      - "feature/**"
    paths:
      - "terraform/**"
  pull_request:
    branches:
      - "main"
    paths:
      - "terraform/**"
    types: [opened, synchronize]
  workflow_dispatch:

env:
  WORKLOAD_IDENTITY_PROVIER: "projects/xxxxxxxxxx/locations/global/workloadIdentityPools/xxxxxxxxxxxxx/providers/xxxxxxxxx"
  SERVICE_ACCOUNT: "github-actions-cicd@{PROJECT_ID}.iam.gserviceaccount.com"
  TERRAFORM_VERSION: "1.9.0"

jobs:
  terraform-ci:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    defaults:
      run:
        working-directory: terraform
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Authenticate to Google Cloud with Workload Identity
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ env.WORKLOAD_IDENTITY_PROVIER }}
          service_account: ${{ env.SERVICE_ACCOUNT }}

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TERRAFORM_VERSION }}
    
      - name: Check Terraform Format
        run: terraform fmt -recursive -check
      
      - name: Terraform Init
        run: terraform init
      
      - name: Terraform Check Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan
```

詳細を解説していきます。
- `name`にはワークフロー名である`Terraform CI`を設定しています。
- `on.push.branches`と`on.push.paths`は、featureブランチを対象とし、terraformディレクトリ配下で差分があった場合にワークフローが起動するといった意味になります。
- `pull_request.branches`、`pull_request.paths`はmainブランチに対して、terraformディレクトリ配下で差分があるプルリクエストがあった場合にワークフローが起動するといった意味になります。
- `workflow_dispatch`は、デバッグ用に手動でワークフローが起動できるように用意しておきました。

```yaml
name: Terraform CI
on:
  push:
    branches:
      - "feature/**"
    paths:
      - "terraform/**"
  pull_request:
    branches:
      - "main"
    paths:
      - "terraform/**"
    types: [opened, synchronize]
  workflow_dispatch:
```

GitHub Actionsのワークフロー中に利用する環境変数を定義しています。
Workload Identityを利用するうえで必要な、Workload Identity Provider IDや、サービスアカウント名、terraformのバージョン番号の指定を行っています。

```yaml
env:
  WORKLOAD_IDENTITY_PROVIER: "projects/xxxxxxxxxx/locations/global/workloadIdentityPools/xxxxxxxxxxxxx/providers/xxxxxxxxx"
  SERVICE_ACCOUNT: "github-actions-cicd@{PROJECT_ID}.iam.gserviceaccount.com"
  TERRAFORM_VERSION: "1.9.0"
```

ワークフローのジョブを定義しています。
- `runs-on`には、ジョブの実行環境を定義します。今回は`ubuntu-latest`です。
- `permissions.id-token`はOIDCトークンを取得するために`write`を指定します。`permissions.contents`は、自身のリポジトリ内のソースコードのアクセスに必要なため、`read`を指定します。
- `defaults.run.working-directory`は、作業ディレクトリを指定します。terraformディレクトリ配下に、tfファイルが存在するため、`terraform`と指定します。
- 

```yaml
jobs:
  terraform-ci:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    defaults:
      run:
        working-directory: terraform
```

- 自身のリポジトリをGitHub Actions実行環境にチェックアウトします。
```yaml
    steps:
      - name: Checkout
        uses: actions/checkout@v4
```

- workload identity providerとサービスアカウントを指定することで、GitHub ActionsからGoogle Cloud上への認証することができます。
```yaml   
      - name: Authenticate to Google Cloud with Workload Identity
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ env.WORKLOAD_IDENTITY_PROVIER }}
          service_account: ${{ env.SERVICE_ACCOUNT }}
```

- `Setup Terraform`では、terraformのインストール等を実行しています。
- `Check Terraform Format`では、tfファイルのフォーマットチェックをしています。フォーマットエラーの場合は、ワークフロー全体がエラーになります。
- `Terraform Init`では、terraform initコマンドを実行して初期化しています。
- `Terraform Check Validate`では、terraform validateコマンドを実行して、terraformの設定ファイルの構文を検証します。エラーの場合は、ワークフロー全体がエラーになります。
- `Terraform Plan`では、terraform planコマンドを実行して、作成・変更・削除されるリソースを確認することができます。

```yaml
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TERRAFORM_VERSION }}
    
      - name: Check Terraform Format
        run: terraform fmt -recursive -check
      
      - name: Terraform Init
        run: terraform init
      
      - name: Terraform Check Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan
```

## terraform_apply.yaml

terraform_ci.yamlでterraform planを実行した際に作成・変更・削除されるリソースが確認できたので、
意図したplan結果になっている場合は、mainブランチにpull_requestをマージしましょう。
`mainブランチにpull_requestがマージされる`ことをトリガーにterraform applyを実行したいので、
ワークフロー定義ファイル`terraform_apply.yaml`も作成しました。


```yaml:terraform_apply.yaml
name: Terraform Apply
on:
  push:
    branches:
      - "main"


env:
  WORKLOAD_IDENTITY_PROVIER: "projects/xxxxxxxxxx/locations/global/workloadIdentityPools/xxxxxxxxxxxxx/providers/xxxxxxxxx"
  SERVICE_ACCOUNT: "github-actions-cicd@{PROJECT_ID}.iam.gserviceaccount.com"
  TERRAFORM_VERSION: "1.9.0"

jobs:
  terraform-apply:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    defaults:
      run:
        working-directory: terraform
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Authenticate to Google Cloud with Workload Identity
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ env.WORKLOAD_IDENTITY_PROVIER }}
          service_account: ${{ env.SERVICE_ACCOUNT }}

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TERRAFORM_VERSION }}
    
      - name: Check Terraform Format
        run: terraform fmt -recursive -check
      
      - name: Terraform Init
        run: terraform init
      
      - name: Terraform Check Validate
        run: terraform validate

      - name: Terraform Apply
        run: terraform apply -auto-approve
```

ほとんどの内容は`terrraform_ci.yaml`と同じですが、
- `Terraform Apply`で、terraform apply -auto-approveコマンドを実行することで、対話型の承認なしでterraform applyが実行されます。事前にどんなリソース変更があるかは`terraform_ci.yaml`の結果で確認済みのため、そのまま実行します。

```yaml
      - name: Terraform Apply
        run: terraform apply -auto-approve
```

# Cloud Storageを作ってみる
実際に今まで説明してきたGitHub Actionsを利用してCloud Storageを作ってみます。
GitHubリポジトリのディレクトリ構成はこちらになります。
```
├── .github
│   └── workflows
│       ├── terraform_apply.yaml
│       └── terraform_ci.yaml
└── terraform
    ├── backend.tf
    ├── cloud_storage.tf
    ├── locals.tf
    ├── provider.tf
    └── versions.tf
```

今回作成するCloud Storageのtfファイルの中身です。
```tf:cloud_storage.tf
resource "google_storage_bucket" "sample_bucket" {
  name          = "github-actions-hogehoge-sample-bucket"
  location      = local.region_name
  force_destroy = true

  public_access_prevention = "enforced"
}
```

featureブランチを作成後に、上記のファイルを作成しコミット、Pushまで実行します。
```
$ git checkout -b feature/gcs_bucket
$ git add terraform/cloud_storage.tf
$ git commit -m "add cloud storage bucket"
$ git push origin feature/gcs_bucket
```

Pushすることで、`terraform_ci.yaml`に定義したワークフローが起動し、terraform planまでを実行してくれています。
![](/images/github-actions-with-workload-identity-20240706/terraform_ci01.png)

plan結果もきちんと表示されていることがわかります。

![](/images/github-actions-with-workload-identity-20240706/terraform_ci02.png)

plan結果に問題がなければプルリクエストを作成して、マージしてみます。
マージすることで、`terraform_apply.yaml`に定義したワークフローが起動し、terraform applyまで実行してくれます。

![](/images/github-actions-with-workload-identity-20240706/terraform_apply01.png)

applyも成功しました。

![](/images/github-actions-with-workload-identity-20240706/terraform_apply02.png)

実際にGoogle CloudのコンソールからCloud Storageの画面に遷移すると、バケットが作成されていることが確認できました。

![](/images/github-actions-with-workload-identity-20240706/gcs.png)



# 最後に
今回は、Workload Identityを利用してGitHub ActionsからTerraformを実行するまでをハンズオン形式で記載しました。
サービスアカウントキーを発行することなく、外部からGoogle Cloudのリソースを操作することができるWorkload Identityはセキュリティ対策にもなるため、積極的に使っていきたいサービスになります。
GitHub Actionsも定型作業を自動化できたり、チーム開発をより良いものにしてくれるので業務にも活かしていきたいと思います。
また、今回作成したterraformのGitHub Actionsですが、**flintやtrivy等のCIツールが導入できていなかったり**、**プルリクエスト上にterraform plan結果を追記できていなかったり**とまだまだ課題がありますので、対応でき次第記事にしていこうと思います。