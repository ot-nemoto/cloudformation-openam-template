# cloudformation-openam-template

### 前提条件

- **aws-cli** がインストールされ、プロファイルの設定がされていること
- HostedZoneに **ドメイン** が登録されていること
- 対象のRegionに **キーペア** が作成されていること
- 実行するIAMユーザに十分な権限があること
- OpenAM
  - Deployする場合は[ForgeRock](https://backstage.forgerock.com/downloads/browse/am/archive/productId:openam)からダウンロードします(**要アカウント登録**)
  - ダウンロード可能な場所に配置して下さい
  - 以下の手順でS3へパブリックな環境を構築することも可能です(`OpenAM-13.0.0.war`がカレントにあること)

```sh
# Warを配置するS3バケットを作成
aws cloudformation create-stack \
  --stack-name public-bucket \
  --template-body file://public-bucket-template.yml

# バケット名を取得
BUCKET_NAME=$(aws cloudformation describe-stacks \
  --stack-name public-bucket \
  --query 'Stacks[].Outputs[?OutputKey==`BucketName`].OutputValue' \
  --output text)
echo ${BUCKET_NAME}
  #

# OpenAMのwarファイルをS3にアップロード
aws s3 cp OpenAM-13.0.0.war s3://${BUCKET_NAME}/

# アップロードしたWarファイルのURIを取得
DONWLOD_URI=$(aws cloudformation describe-stacks \
  --stack-name public-bucket \
  --query 'Stacks[].Outputs[?OutputKey==`DownloadUri`].OutputValue' \
  --output text)
echo ${DONWLOD_URI}
  #
```

### 構築手順

- 環境変数を設定
  - 作成済み（作成した）キーペアのキー名
  - Route53に登録済みのHostedZone名
  - OpenAMがダウンロード可能なURI

```sh
# 例)
KEY_NAME=openam-key
HOSTED_ZONE_NAME=example.com.
OPENAM_WAR_URI=${DONWLOD_URI}/OpenAM-13.0.0.war
```

- 環境構築

```sh
aws cloudformation create-stack \
  --stack-name openam \
  --capabilities CAPABILITY_IAM \
  --parameters ParameterKey=KeyName,ParameterValue=${KEY_NAME} \
               ParameterKey=HostedZoneName,ParameterValue=${HOSTED_ZONE_NAME} \
               ParameterKey=OpenamWarUri,ParameterValue=${OPENAM_WAR_URI} \
  --template-body file://openam-template.yml
```

### Parameters

|Name|Type|Default|Required|
|--|--|--|--|
|ProjectName|String|openam|*No*|
|VpcCIDR|String|10.100.0.0/16|*No*|
|PublicSubnet1CIDR|String|10.100.0.0/24|*No*|
|PublicSubnet2CIDR|String|10.100.1.0/24|*No*|
|DeveloperCIDR|String|0.0.0.0/0|*No*|
|InstanceType|String|t2.micro|*No*|
|KeyName|AWS::EC2::KeyPair::KeyName|-|*Yes*|
|HostedZoneName|String|-|*Yes*|
|OpenamWarUri|String|-|*Yes*|
