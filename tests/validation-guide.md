# IAMポリシー・CloudFormationの検証手順

## 1. IAMポリシーのJSON構文を確認する

**コマンド：**

```bash
python3 -m json.tool <対象ファイル>
```

例：

```bash
python3 -m json.tool policies/s3-read-policy.json
python3 -m json.tool policies/s3-restricted-policy.json
```

**確認観点：**
- JSONとして正常に解析できること
- エラーが発生せず、JSONが出力されること

※この確認ではJSONの構文を検証する。IAMポリシーとしての権限設定の妥当性は別途確認する。

## 2. IAM Access Analyzerでポリシーを検証する

**操作手順：**

AWSマネジメントコンソールで「IAM ＞ ポリシー ＞ ポリシーの作成」を開き、JSONエディタに検証対象のポリシーJSONを貼り付けて、検証結果を確認する。

**確認観点：**
- セキュリティ上の検出結果が0件であること
- エラーが0件であること
- 警告が0件であること
- 提案が0件であること

※検証結果を確認した後、必要に応じてJSONを修正し、再度検証する。

## 3. IAM Policy Simulatorでアクションとリソースの組み合わせを評価する

**操作手順：**

AWSマネジメントコンソールでIAM Policy Simulatorを開き、対象のポリシーを選択する。必要に応じてカスタムポリシーを設定し、アクションとリソースを指定してシミュレーションを実行する。

- サービス：S3
- アクション：`s3:GetObject`
- リソース：検証対象のS3オブジェクトARN

**確認観点：**
- 許可を想定したアクションとリソースの組み合わせで、評価結果が `Allowed` となること

※シミュレーション結果は指定した条件での評価結果であり、実環境でのアクセス成功を保証するものではない。

## 4. CloudFormationのJSON版・YAML版を検証する

**コマンド：**

```bash
aws cloudformation validate-template \
  --template-body file://cloudformation/iam-role.json

aws cloudformation validate-template \
  --template-body file://cloudformation/iam-role.yaml
```

**確認観点：**
- コマンドが正常終了すること
- テンプレートの情報が出力されること

※この検証はテンプレートの形式などを確認するものであり、実際のリソース作成や権限の動作を保証するものではない。

## 5. 信頼ポリシーを検証する

**コマンド：**

```bash
aws accessanalyzer validate-policy \
  --policy-document file://tests/trust-policy.json \
  --policy-type RESOURCE_POLICY \
  --validate-policy-resource-type AWS::IAM::AssumeRolePolicyDocument
```

**確認観点：**
- コマンドが正常終了すること
- `findings` が空配列（`[]`）であること

※検証結果は、対象ポリシーに対して検出された指摘事項を示す。必要に応じて内容も確認する。

## 6. 検証結果を記録する

検証を実施したら、結果と必要な補足事項を `docs/policy-validation.md` に記録する。

- 検証対象のファイル
- 実施した検証方法・コマンド
- 検証結果
- エラーや指摘事項があった場合の対応内容
- 検証上の制約や注意事項