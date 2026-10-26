# IAMポリシーの検証

## 1. 概要

本ドキュメントでは、本ポートフォリオで作成したIAMポリシーの検証結果を記録する。

ポリシー検証の目的は、ポリシーを使用する前に、構文や構造の誤り、セキュリティ上の警告、AWSのベストプラクティスからの逸脱などを検出することである。

以下のツールを使用した。

- IAM Access Analyzerによるポリシー検証
- IAM Policy Simulator
- AWS CloudFormationテンプレートの検証

## 2. 検証結果

### 2.1 s3-read-policy.json

**使用ツール：** IAM Access Analyzerによるポリシー検証

**検証結果：**

| 指摘の種類 | 件数 |
|---|---:|
| セキュリティ | 0 |
| エラー | 0 |
| 警告 | 0 |
| 提案 | 0 |

また、IAM Policy Simulatorを使用してポリシーの動作を検証した。

- アクション：`s3:GetObject`
- リソース：ポリシーで指定したS3 ARNに一致するリソース
- 結果：許可（Allowed）

### 2.2 s3-restricted-policy.json

**使用ツール：** IAM Access Analyzerによるポリシー検証

**検証結果：**

| 指摘の種類 | 件数 |
|---|---:|
| セキュリティ | 0 |
| エラー | 0 |
| 警告 | 0 |
| 提案 | 0 |

ポリシー検証では、指摘事項は報告されなかった。

### 2.3 cloudformation/iam-role.json — インラインのPolicyDocument

CloudFormationテンプレート内のインライン権限ポリシーを抽出し、IAM Access Analyzerを使用して個別に検証した。

**検証結果：** エラーは報告されなかった。

このポリシーは、以下のARNに一致するオブジェクトに対して `s3:GetObject` を許可する。

`arn:aws:s3:::example-bucket/reports/2026/*`

### 2.4 CloudFormationテンプレート

AWS CLIを使用して、テンプレート全体を検証した。

実行コマンド：

`aws cloudformation validate-template --template-body file://cloudformation/iam-role.json`

**検証結果：** 検証に成功した。

CLIからテンプレートの説明と `CAPABILITY_NAMED_IAM` が返された。これは、テンプレートに名前付きIAMリソースの作成に関する権限確認が必要なリソースが含まれていることを示す。
YAML版についても、AWS CLIを使用してテンプレート全体を検証した。

実行コマンド：

`aws cloudformation validate-template --template-body file://cloudformation/iam-role.yaml`

**検証結果：** 検証に成功した。

CLIからテンプレートの説明と `CAPABILITY_NAMED_IAM` が返された。

#### JSON版とYAML版の比較

`cloudformation/iam-role.json` と `cloudformation/iam-role.yaml` の内容を比較した。

比較対象は、以下のとおりである。

- CloudFormationリソースタイプ
- IAMロール名
- 信頼ポリシー（`AssumeRolePolicyDocument`）
- 信頼ポリシーのプリンシパルとアクション
- インライン権限ポリシー（`Policies`）
- 許可アクションと対象リソースARN

**比較結果：** JSON版とYAML版で、上記の設定内容が一致していることを確認した。

なお、`aws cloudformation validate-template` による検証成功は、テンプレートがCloudFormationで受け付けられる形式であることを確認するものであり、実際のデプロイやS3オブジェクトへのアクセス成功を保証するものではない。

## 3. 信頼ポリシーと権限ポリシー

CloudFormationテンプレートには、信頼ポリシーと権限ポリシーという2種類のポリシードキュメントが含まれている。それぞれ役割が異なるため、ポリシーの種類に応じた方法で検証する必要がある。

### 3.1 AssumeRolePolicyDocument

これは信頼ポリシーであり、どのプリンシパルがIAMロールを引き受けられるかを定義する。

本テンプレートでは、信頼するプリンシパルとしてAmazon EC2サービスを指定し、`sts:AssumeRole` アクションを許可している。

AWS CLIからIAM Access Analyzerを実行し、信頼ポリシーを検証した。

実行コマンド：

`aws accessanalyzer validate-policy --policy-document file://tests/trust-policy.json --policy-type RESOURCE_POLICY --validate-policy-resource-type AWS::IAM::AssumeRolePolicyDocument`

**検証結果：** 指摘事項は報告されなかった（`"findings": []`）。

当初、信頼ポリシーをアイデンティティベースのポリシーとしてIAMコンソールのポリシーエディタに入力したところ、リソースの指定不足と、プリンシパルがサポートされていないことを示す指摘が報告された。

これは、信頼ポリシーを適切でない種類のポリシーとして検証したために生じた結果であり、それだけで信頼ポリシー自体が無効であることを意味するものではない。

### 3.2 PolicyDocument

これは、CloudFormationテンプレートに埋め込まれたアイデンティティベースの権限ポリシーである。IAMロールが、どのAWSアクションをどのリソースに対して実行できるかを定義する。

本テンプレートでは、以下のS3 ARNに一致するオブジェクトに対して `s3:GetObject` を許可している。

`arn:aws:s3:::example-bucket/reports/2026/*`

抽出したポリシードキュメントをIAM Access Analyzerで検証した結果、エラーは報告されなかった。

### 3.3 それぞれの違い

- **信頼ポリシー（`AssumeRolePolicyDocument`）：** 誰がIAMロールを引き受けられるかを定義する。
- **権限ポリシー（`PolicyDocument`）：** IAMロールが、指定したリソースに対してどのアクションを実行できるかを定義する。

今回の構成では、両方のポリシーが必要となる。EC2サービスによるIAMロールの引き受けを許可しただけでは、S3オブジェクトへのアクセス権限は付与されない。S3へのアクセスには、適切な権限ポリシーも必要となる。

### 3.4 検証時の注意点

IAMポリシーは、その種類に応じた方法で検証する必要がある。信頼ポリシーはリソースベースのポリシーとして、埋め込まれた権限ポリシーはアイデンティティベースのポリシーとして検証する。

指摘事項がないという検証結果は、選択した検証条件のもとで、検証ツールから問題が報告されなかったことを意味する。ポリシーにすべてのセキュリティリスクが存在しないことや、実際のAWSリクエストが必ず成功することを保証するものではない。

## 4. 検証の限界

- 指摘事項がない場合でも、あらゆる状況においてポリシーの安全性が保証されるわけではない。
- IAM Access Analyzerによるポリシー検証では、ポリシーの文法やAWSのベストプラクティスに関する確認を行うが、実際の動作検証を代替するものではない。
- IAM Policy Simulatorは、指定したアクションとリソースを、ポリシー評価の条件に基づいて評価する。許可（Allowed）という結果が、実際のAWSリクエストの成功を保証するわけではない。
- CloudFormationテンプレートの検証は、デプロイされたリソースや実際のアクセス権限のテストを代替するものではない。
- 信頼ポリシーと権限ポリシーは、ロールへのアクセスを異なる観点から制御するため、それぞれ個別に確認する必要がある。

## 5. 参考資料

- IAM Access Analyzerによるポリシー検証：[https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-validation.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-validation.html)
- IAMポリシーの検証チェック：[https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-checks-validating-policies.html](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-checks-validating-policies.html)
- AWS CloudFormation ValidateTemplate：[https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_ValidateTemplate.html](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_ValidateTemplate.html)