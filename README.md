# Boilerplate CFN Frontend

aws cloudformation template

## Setup

### cloudformation デプロイ用バケットの作成

環境構築で最初に 1 回だけ実行して、cloudformation でデプロイする用のバケットを作成する

下記項目は各自読み替えてください

| 項              | 値                                        |
| --------------- | ----------------------------------------- |
| --stack-name    | cloudformation stack name                 |
| --template-body | setup/cfn.template.yml までの path を指定 |

```
$ aws cloudformation create-stack \
  --stack-name naughtldy-artifact \
  --template-body file://path/to/infra/setup/cfn.template.yml \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile {デプロイ用IAMユーザー（自分のユーザーを指定）}
```

## Acm Certificate Arn

CloudFront で AWS の SSL 証明書を使用するためには、AWS Certificate Manager のリージョンは米国東部 (バージニア北部)にする必要があります。
そのため、本リポジトリには SSL 証明書の取得は含んでいません。

https://aws.amazon.com/jp/premiumsupport/knowledge-center/install-ssl-cloudfront/

Certificate Manager の region=us-east-1 で SSL 証明書発行して `params/params.prod.json` の `AcmCertificateArn` に `ARN` を追加します。

## Deploy

下記項目は各自読み替えてください

| 項           | 値                                   |
| ------------ | ------------------------------------ |
| --s3-bucket  | Setup で作成したバケット名を指定する |
| --stack-name | cloudformation stack name            |
| --region     | デプロイしたい region を指定する     |

1. build and upload template

```
aws cloudformation package \
  --template-file src/cfn.template.yml \
  --s3-bucket naughtldy-artifact \
  --s3-prefix template/infra \
  --output-template-file .cfn/packaged.yml
```

2. Run CloudFormation

```
aws cloudformation deploy \
  --stack-name naughtldy-infra-prod \
  --template-file .cfn/packaged.yml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides `cat params/params.prod.json | jq -r '.Parameters | to_entries | map("\(.key)=\(.value|tostring)") | .[]' | tr '\n' ' ' | awk '{print}'` \
  --region ap-northeast-1
```
