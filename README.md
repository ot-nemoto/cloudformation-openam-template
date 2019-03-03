# cloudformation-openam-template

## openam-template

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
aws cloudformation create-stack \
  --stack-name public-bucket \
  --template-body file://public-bucket-template.yml

BUCKET_NAME=$(aws cloudformation describe-stacks \
  --stack-name public-bucket \
  --query 'Stacks[].Outputs[?OutputKey==`BucketName`].OutputValue' \
  --output text)
echo ${BUCKET_NAME}
  #

aws s3 cp OpenAM-13.0.0.war s3://${BUCKET_NAME}/

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

## sso-app-template

### 前提条件

- openam-templateで環境構築済みであること
- **aws-cli** がインストールされ、プロファイルの設定がされていること
- HostedZoneに **ドメイン** が登録されていること
- 対象のRegionに **キーペア** が作成されていること
- 実行するIAMユーザに十分な権限があること

### 構築手順

- 環境変数を設定
  - 作成済み（作成した）キーペアのキー名
  - Route53に登録済みのHostedZone名
  - OpenAMのURI
  - OpenAMのAdminユーザのユーザ名
  - OpenAMのAdminユーザのパスワード
  - OpenAMユーザがログインする際のAWSロールのARN
  - OpenAMユーザのロールの信頼されたエンティティのARN
  - onloginのURLを指定
  - Adminユーザでログインし DEVELOPERS > API Credentials で登録したクライアントID
  - Adminユーザでログインし DEVELOPERS > API Credentials で登録したシークレットID
  - Adminユーザでログインし USERS > Roles > Role のURLから確認できるロールID
  - Adminユーザでログインし APPS > Company Apps > App のURLから確認できるアプリID

```sh
# 例)
KEY_NAME=openam-key
HOSTED_ZONE_NAME=example.com.
OPENAM_URL=http://opneam.example.com/openam
OPENAM_ADMIN_USER=amAdmin
OPENAM_ADMIN_PASS=password
OPENAM_AWS_ROLE_ARN=arn:aws:iam::<Account Id>:role/<Role Name>
OPENAM_AWS_ID_PROVIDER_ARN=arn:aws:iam::<Account Id>:saml-provider/<Provider Name>
ONELOGIN_URI=https://<OneLogin Domain>/onlgoin.com
ONELOGIN_CLIENT_ID=0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
ONELOGIN_CLIENT_SECRET=0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
ONELOGIN_ROLE_ID=12345
ONELOGIN_APP_ID=67890
```

- 環境構築

```sh
aws cloudformation create-stack \
  --stack-name sso-app \
  --parameters ParameterKey=KeyName,ParameterValue=${KEY_NAME} \
               ParameterKey=HostedZoneName,ParameterValue=${HOSTED_ZONE_NAME} \
               ParameterKey=OpenAMUri,ParameterValue=${OPENAM_URI} \
               ParameterKey=OpenAMAdminUser,ParameterValue=${OPENAM_ADMIN_USER} \
               ParameterKey=OpenAMAdminPass,ParameterValue=${OPENAM_ADMIN_PASS} \
               ParameterKey=OpenAMAwsRoleArn,ParameterValue=${OPENAM_AWS_ROLE_ARN} \
               ParameterKey=OpenAMAwsIDProviderArn,ParameterValue=${OPENAM_AWS_ID_PROVIDER_ARN} \
               ParameterKey=OneloginUri,ParameterValue=${ONELOGIN_URI} \
               ParameterKey=OneloginClientId,ParameterValue=${ONELOGIN_CLIENT_ID} \
               ParameterKey=OneloginClientSecret,ParameterValue=${ONELOGIN_CLIENT_SECRET} \
               ParameterKey=OneloginRoleId,ParameterValue=${ONELOGIN_ROLE_ID} \
               ParameterKey=OneloginAppId,ParameterValue=${ONELOGIN_APP_ID} \
  --template-body file://sso-app-template.yml
```

### Parameters

|Name|Type|Default|Required|
|--|--|--|--|
|ProjectName|String|sso-app|*No*|
|DeveloperCIDR|String|0.0.0.0/0|*No*|
|InstanceType|String|t2.micro|*No*|
|KeyName|AWS::EC2::KeyPair::KeyName|-|*Yes*|
|HostedZoneName|String|-|*Yes*|
|OpenAMStackName|String|openam|*No*|
|OpenAMUri|String||*No*|
|OpenAMAdminUser|String||*No*|
|OpenAMAdminPass|String||*No*|
|OpenAMAwsRoleArn|String||*No*|
|OpenAMAwsIDProviderArn|String||*No*|
|OneloginUri|String||*No*|
|OneloginClientId|String||*No*|
|OneloginClientSecret|String||*No*|
|OneloginRoleId|String||*No*|
|OneloginAppId|String||*No*|
