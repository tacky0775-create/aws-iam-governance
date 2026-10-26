# IAMロール設計書

## 1. 設計目的

EC2から、S3バケット example-bucket の reports/2026/ 配下にあるオブジェクトを読み取れるようにする。

## 2. 信頼ポリシー

AssumeRolePolicyDocument で、EC2サービス（ec2.amazonaws.com）がIAMロールを引き受けることを許可する。

sts:AssumeRole は、IAMロールを引き受けるためのアクションとして指定する。

## 3. 権限ポリシー

PolicyDocument で、s3:GetObject を次のリソースARNに対して許可する。

arn:aws:s3:::example-bucket/reports/2026/*

これにより、指定したプレフィックス配下のS3オブジェクトの読み取りを許可する。

## 4. 設計上の意図

必要な読み取り権限と対象範囲を明示し、意図しない範囲へのアクセスを避ける。

バケット全体ではなく、対象のプレフィックスを指定してアクセス範囲を限定する。