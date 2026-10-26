# IAM権限ポリシー比較

## 1. 比較目的

S3オブジェクトの読み取りを許可する2種類のIAMポリシーを比較し、Resourceの指定範囲による権限の違いを確認する。

## 2. 比較対象

- `policies/s3-read-policy.json`
- `policies/s3-restricted-policy.json`

## 3. 比較結果

| 項目 | s3-read-policy.json | s3-restricted-policy.json |
|---|---|---|
| Effect | Allow | Allow |
| Action | s3:GetObject | s3:GetObject |
| Resource | arn:aws:s3:::example-bucket/reports/* | arn:aws:s3:::example-bucket/reports/2026/* |

両方のポリシーでEffectとActionは同じだが、Resourceの指定範囲が異なる。

通常版は `reports/` 配下のオブジェクトを対象とする。一方、制限版は `reports/2026/` 配下のオブジェクトに対象を限定する。

## 4. 設計上の考察

必要なオブジェクトの読み取りだけを許可する場合、Resourceの範囲を適切に限定することで、不要なアクセスを許可するリスクを抑えられる。

ただし、この比較は各ポリシー単独での許可範囲を示すものであり、実際のアクセス可否はほかのポリシーやリソース側の設定にも依存する。