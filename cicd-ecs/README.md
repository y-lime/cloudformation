# cicd-ecs

CodePipeline を構築する CloudFormation。
プロジェクトをコンテナビルドし、ECR にコンテナイメージをプッシュする。オプションとして、ECS Service を起動する。

## 使い方

1. テンプレートソースに`../string-functions/string.yml`を指定して、CloudFormation を実行する。
   - `cicd-ecs.yml`において`string.yml`で作成する CloudFormation マクロを使用している。
2. テンプレートソースに`./cicd-ecs.yml`を指定して、CloudFormation を実行する。

## Parameter

| パラメータ              | 型     | 説明                                                    |
| ----------------------- | ------ | ------------------------------------------------------- |
| ProjectName             | String | ビルド対象のアプリケーションプロジェクト名              |
| RepositoryName          | String | CodeCommit のレポジトリ名                               |
| BranchName              | String | ビルド対象の CodeCommit のブランチ名                    |
| EcrRepositoryName       | String | コンテナイメージをプッシュする ECR レポジトリ名         |
| BuildDockerImage        | String | CodeBuild で使用するビルド環境のイメージ                |
| EcsClusterName          | String | (オプション)起動する ECS サービスの ECS クラスター名    |
| EcsServiceName          | String | (オプション)起動する ECS サービス名                     |
| EcsContainerName        | String | (オプション)起動する ECS コンテナ名                     |
| EcsDeployTimeoutMinutes | String | (オプション)ECS デプロイのタイムアウト時間 (最大 60 分) |

## Build

CodeCommit 直下の`buildspec.yml` を使用してビルドを実行する。

## Deploy

Amazon ECS アクションを使用して、Amazon ECS クラスターにコンテナアプリケーションをデプロイする。
パラメータの`EcsClusterName`、`EcsServiceName`、`EcsContainerName`の全てが入力されている時、Deploy フェーズを作成し、デプロイを実行する。

## 環境変数

CodeBuild で設定されている環境変数。`buildspec.yml` 内で使用できる。

| 環境変数名      | 値                             |
| --------------- | ------------------------------ |
| PROJECT_NAME    | パラメータの ProjectName       |
| REPOSITORY_NAME | パラメータの EcrRepositoryName |
| CONTAINER_NAME  | パラメータの EcsContainerName  |

## 参考

### IAM Policy

source:https://docs.aws.amazon.com/ja_jp/codepipeline/latest/userguide/security-iam.html#how-to-custom-role

Amazon ECS では、Amazon ECS デプロイアクションを使用してパイプラインを作成するために必要な最小限のアクセス権限は次のとおりです。

```json
{
    "Effect": "Allow",
    "Action": [
        "ecs:DescribeServices",
        "ecs:DescribeTaskDefinition",
        "ecs:DescribeTasks",
        "ecs:ListTasks",
        "ecs:RegisterTaskDefinition",
        "ecs:UpdateService"
    ],
    "Resource": "resource_ARN"
},
```

また、タスクに IAM ロールを使用するための iam:PassRole アクセス許可を追加する必要があります。詳細については、「Amazon ECS タスク実行 IAM ロールそしてタスクの IAM ロール」を参照してください。以下のポリシーテキストを使用します。

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": [
        "arn:aws:iam::aws_account_ID:role/ecsTaskExecutionRole_or_TaskRole_name"
      ]
    }
  ]
}
```

### ECS Deploy

https://docs.aws.amazon.com/codepipeline/latest/userguide/action-reference-ECS.html
