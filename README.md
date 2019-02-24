# cloudformation-openam-template

### 前提条件

- **aws-cli** がインストールされ、プロファイルの設定がされていること
- HostedZoneに **ドメイン** が登録されていること
- 対象のRegionに **キーペア** が作成されていること
- 実行するIAMユーザに十分な権限があること

### 構築手順

```sh
# 例)
KEY_PAIR=openam-key
HOSTED_ZONE_NAME=example.com.

aws cloudformation create-stack \
  --stack-name openam \
  --capabilities CAPABILITY_IAM \
  --parameters ParameterKey=KeyPair,ParameterValue=${KEY_PAIR} \
               ParameterKey=HostedZoneName,ParameterValue=${HOSTED_ZONE_NAME} \
  --template-body file://openam-template.yml
```

### Parameters

|Name|Type|Default|
|--|--|--|
|ProjectName|String|openam|
|VpcCIDR|String|10.100.0.0/16|
|PublicSubnet1CIDR|String|10.100.0.0/24|
|PublicSubnet2CIDR|String|10.100.1.0/24|
|DeveloperCIDR|String|0.0.0.0/0|
|InstanceType|String|t2.micro|
|KeyPair|AWS::EC2::KeyPair::KeyName|-|
|HostedZoneName|String|-|
