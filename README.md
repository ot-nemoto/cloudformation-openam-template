# cloudformation-openam-template

## OpenAM

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

- OpenAMのURI取得

```sh
DOMAIN=$(aws cloudformation describe-stacks \
  --stack-name openam \
  --query 'Stacks[].Outputs[?OutputKey==`PublicDns`].OutputValue' \
  --output text)
echo http://${DOMAIN}/openam
  #
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

### Outputs

|Key|ExportName|Description|
|--|--|--|
|OpsWorksInstanceProfileArn|\<stack-name>-opsworks-instance-profile-arn|インスタンスプロファイルArn|
|OpsWorksServiceRoleArn|\<stack-name>-opsworks-service-role-arn|OpsWorksのサービスロールArn|
|PublicDns|\<stack-name>-public-dns|ドメイン|
|PublicIp|\<stack-name>-public-ip|パブリックIPアドレス|
|PublicSubnet1|\<stack-name>-public-subnet-1|パブリックサブネットID<br>（OpenAMが配置されているサブネット）|
|PublicSubnet2|\<stack-name>-public-subnet-2|パブリックサブネットID<br>（OpenDJも構築する場合はこのIPを利用）|
|SecurityGroup|\<stack-name>-security-group|セキュリティグループID|
|Vpc|\<stack-name>-vpc|VPC ID|

## OpenDJ

### 前提条件

- OpenAMが上記スタックで構築されていること
- OpenDJ
  - Deployする場合はForgeRockからダウンロードします(要アカウント登録)
  - ダウンロード可能な場所に配置して下さい
  - 以下の手順でS3へパブリックな環境を構築することも可能です(`opendj-3.0.0-1.noarch.rpm`がカレントにあること)

```sh
# OpenDJのrpmファイルをS3にアップロード
aws s3 cp opendj-3.0.0-1.noarch.rpm s3://${BUCKET_NAME}/

DONWLOD_URI=$(aws cloudformation describe-stacks \
  --stack-name public-bucket \
  --query 'Stacks[].Outputs[?OutputKey==`DownloadUri`].OutputValue' \
  --output text)
echo ${DONWLOD_URI}
```

### 環境構築

- 環境変数を設定
  - 作成済み（作成した）キーペアのキー名
  - Route53に登録済みのHostedZone名
  - OpenDJがダウンロード可能なURI
  - OpenDJのルートパスワード
  - OpenDJのベースドメイン名

```sh
# 例)
KEY_NAME=openam-key
HOSTED_ZONE_NAME=example.com.
OPENDJ_RPM_URI=${DONWLOD_URI}/opendj-3.0.0-1.noarch.rpm
ROOT_PW=secret
BASE_DN=dc=example\\\,dc=com
```

- 環境構築

```sh
aws cloudformation create-stack \
  --stack-name opendj \
  --capabilities CAPABILITY_IAM \
  --parameters ParameterKey=KeyName,ParameterValue=${KEY_NAME} \
               ParameterKey=HostedZoneName,ParameterValue=${HOSTED_ZONE_NAME} \
               ParameterKey=OpendjRpmUri,ParameterValue=${OPENDJ_RPM_URI} \
               ParameterKey=RootPw,ParameterValue="${ROOT_PW}" \
               ParameterKey=BaseDn,ParameterValue="${BASE_DN}" \
  --template-body file://opendj-template.yml
```

### Parameters

|Name|Type|Default|Required|
|--|--|--|--|
|ProjectName|String|opendj|*No*|
|OpenamStackName|String|openam|*No*|
|DeveloperCIDR|String|0.0.0.0/0|*No*|
|InstanceType|String|t2.micro|*No*|
|KeyName|AWS::EC2::KeyPair::KeyName|-|*Yes*|
|HostedZoneName|String|-|*Yes*|
|OpendjRpmUri|String|-|*Yes*|
|RootPw|String|-|*Yes*|
|BaseDn|String|-|*Yes*|

### Outputs

|Key|ExportName|Description|
|--|--|--|
|PublicDns|\<stack-name>-public-dns|ドメイン|
|PublicIp|\<stack-name>-public-ip|パブリックIPアドレス|
