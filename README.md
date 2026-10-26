# AWS IAM Governance Portfolio

## 1. 概要

本リポジトリは、AWS IAM（Identity and Access Management）によるアクセス権限管理について学習し、IAMポリシーの設計・比較・検証を行うことを目的としたポートフォリオです。

IAMポリシーの作成だけでなく、権限の適用範囲を比較し、AWSの検証ツールを使用して設定内容を確認することで、最小権限の原則と安全なアクセス制御への理解を深めます。

また、AWS CloudFormationを使用してIAMロールを定義し、インフラストラクチャをコードで管理する方法についても学習します。

## 2. 目的

本プロジェクトでは、以下の内容に取り組みます。

- IAMポリシーの構造と各要素の役割を理解する
- S3へのアクセス権限を定義し、ポリシーによる権限範囲の違いを比較する
- 信頼ポリシーと権限ポリシーの役割の違いを理解する
- IAM Access Analyzerを使用してポリシーを検証する
- IAM Policy Simulatorを使用して権限の評価結果を確認する
- CloudFormationでIAMロールを定義し、テンプレートを検証する
- ポリシーのレビューや検証結果をドキュメントとして記録する

## 3. 使用技術・ツール

| 技術・ツール | 用途 |
|---|---|
| AWS IAM | ユーザー、ロール、ポリシーによるアクセス制御 |
| Amazon S3 | アクセス制御の対象となるストレージサービス |
| AWS CloudFormation | IAMロールなどのAWSリソースをコードで定義 |
| IAM Access Analyzer | IAMポリシーの検証 |
| IAM Policy Simulator | IAMポリシーの権限評価 |
| AWS CLI | ポリシー検証やCloudFormationテンプレートの検証 |
| JSON / YAML | IAMポリシーおよびCloudFormationテンプレートの記述 |

## 4. ディレクトリ構成

```text
aws-iam-governance/
├── README.md
├── policies/
│   ├── s3-read-policy.json
│   └── s3-restricted-policy.json
├── cloudformation/
│   ├── iam-role.json
│   └── iam-role.yaml
├── docs/
│   ├── design.md
│   ├── permission-comparison.md
│   ├── review-checklist.md
│   └── policy-validation.md
└── tests/
    ├── trust-policy.json
    └── validation-guide.md
```

### 各ディレクトリの役割

- `policies/`：S3へのアクセス権限を定義したIAMポリシーを格納する。
- `cloudformation/`：IAMロールを定義するCloudFormationテンプレートを格納する。
- `docs/`：設計方針、ポリシーの比較、レビュー観点、検証結果を記録する。
- `tests/`：検証に使用するポリシーファイルや検証手順を格納する。

## 5. IAMポリシーの設計

### 5.1 S3読み取りポリシー

`policies/s3-read-policy.json` では、S3オブジェクトの取得に必要な `s3:GetObject` アクションを定義します。

このポリシーを基準として、許可するアクションやリソースの指定方法を確認します。

### 5.2 制限付きS3ポリシー

`policies/s3-restricted-policy.json` では、アクセス可能なリソースの範囲を制限したポリシーを定義します。

2つのポリシーを比較することで、`Action` と `Resource` の指定がアクセス権限にどのような影響を与えるかを確認します。

### 5.3 信頼ポリシーと権限ポリシー

CloudFormationテンプレートでは、IAMロールに以下の2種類のポリシーを設定します。

**信頼ポリシー（`AssumeRolePolicyDocument`）**

どのプリンシパルがIAMロールを引き受けられるかを定義します。本プロジェクトでは、Amazon EC2サービスによるロールの引き受けを許可します。

**権限ポリシー（`PolicyDocument`）**

IAMロールを使用して、どのAWSアクションをどのリソースに対して実行できるかを定義します。本プロジェクトでは、指定したS3オブジェクトに対する `s3:GetObject` を許可します。

信頼ポリシーはロールを引き受けられる主体を制御し、権限ポリシーはロールに付与する操作権限を制御します。両者を分けて定義することで、アクセス制御の役割を明確にします。

## 6. CloudFormationによるIAMロールの定義

`cloudformation/iam-role.json` および `cloudformation/iam-role.yaml` では、CloudFormationを使用してIAMロールを定義します。

テンプレートでは、信頼ポリシーとインライン権限ポリシーを設定し、EC2サービスによるロールの引き受けと、指定したS3オブジェクトの読み取り権限を定義します。

JSON形式とYAML形式のテンプレートを用意することで、CloudFormationにおける異なる記述形式を比較できるようにします。

## 7. 検証内容と結果

本プロジェクトでは、ポリシーやテンプレートの種類に応じて検証方法を使い分けます。

| 検証対象 | 使用ツール | 結果 |
|---|---|---|
| `s3-read-policy.json` | IAM Access Analyzer | 指摘事項なし |
| `s3-restricted-policy.json` | IAM Access Analyzer | 指摘事項なし |
| CloudFormation内の `PolicyDocument` | IAM Access Analyzer | エラーなし |
| `tests/trust-policy.json` | IAM Access Analyzer（AWS CLI） | 指摘事項なし |
| `cloudformation/iam-role.json` | CloudFormation `validate-template` | 検証成功 |
| `cloudformation/iam-role.yaml` | CloudFormation `validate-template` | 検証成功 |
| JSON版とYAML版の設定比較 | テンプレート内容の比較 | 主要な設定内容が一致 |
| S3読み取り操作 | IAM Policy Simulator | `s3:GetObject` が許可されることを確認 |

信頼ポリシーの検証では、AWS CLIの `validate-policy` コマンドに `--policy-type RESOURCE_POLICY` と `--validate-policy-resource-type AWS::IAM::AssumeRolePolicyDocument` を指定しました。

また、IAM Policy Simulatorでは、指定したアクションとリソースの組み合わせに対する評価結果を確認しました。

詳細な検証手順と結果は [`docs/policy-validation.md`](docs/policy-validation.md) に記載しています。

## 8. 設計・レビュー方針

本プロジェクトでは、以下の観点を重視します。

- 必要なアクションのみを許可する
- リソースの指定範囲を必要最小限にする
- 信頼ポリシーと権限ポリシーを区別する
- ポリシーの種類に適した検証方法を選択する
- 検証結果だけでなく、設定の意図とアクセス範囲も確認する
- 設計方針や検証結果をドキュメントに記録する

レビュー時の確認項目については [`docs/review-checklist.md`](docs/review-checklist.md) に、ポリシー間の比較については [`docs/permission-comparison.md`](docs/permission-comparison.md) に記載しています。

## 9. 注意事項

本プロジェクトはIAMの設計・検証方法を学習するためのポートフォリオです。

検証結果に指摘事項がない場合でも、すべてのセキュリティリスクが排除されていることを保証するものではありません。また、IAM Policy Simulatorの評価結果やCloudFormationテンプレートの検証成功だけでは、実際のAWS環境におけるすべてのアクセス動作を保証できません。

実環境で利用する際は、実際の要件、アクセス範囲、他のポリシーや組織の制御設定なども含めて確認する必要があります。

## 10. 参考資料

- [AWS IAM ポリシーとアクセス許可](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)
- [IAM Access Analyzerによるポリシー検証](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-validation.html)
- [IAMポリシーの検証チェック](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-checks-validating-policies.html)
- [AWS CloudFormation ValidateTemplate](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_ValidateTemplate.html)