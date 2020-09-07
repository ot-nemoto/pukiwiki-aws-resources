# pukiwiki-instance

## deploy

### environments

|Name|Type|Default|Description|
|--|--|--|--|
|ImageId|`AWS::EC2::Image::Id`|ami-0cc75a8978fbbc969|EC2インスタンスのAMI|
|InstanceType|String|t2.micro|EC2インスタンスのInstanceType|
|SnapshotId|String|(*required*)|`/wiki`にマウントするボリュームの[スナップショット](https://ap-northeast-1.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#Snapshots:sort=desc:startTime)|
|AvailabilityZone|`AWS::EC2::AvailabilityZone::Name`|ap-northeast-1a|EC2インスタンスを構築するAZ|
|DockerComposeVersion|String|1.26.2|利用するdocker-composeのバージョン|
|Relayhost|String|(*required*)|送信サーバ|
|RelayhostPort|String|587|送信サーバのポート番号|
|RelayhostUser|String|(*required*)|送信サーバのユーザID|
|RelayhostPass|String|(*required*)|送信サーバのパスワード|

```sh
# e.g.
SNAPSHOT_ID=snap-0927dbdc6113398f0
RELAYHOST=smtp.example.com
RELAYHOST_USER=user@smtp.example.com
RELAYHOST_PASS=password
```

### create-stack

```sh
aws cloudformation create-stack \
    --stack-name pukiwiki \
    --parameters ParameterKey=SnapshotId,ParameterValue=${SNAPSHOT_ID} \
                 ParameterKey=Relayhost,ParameterValue=${RELAYHOST} \
                 ParameterKey=RelayhostUser,ParameterValue=${RELAYHOST_USER} \
                 ParameterKey=RelayhostPass,ParameterValue=${RELAYHOST_PASS} \
    --capabilities CAPABILITY_IAM \
    --template-body file://template.yaml
```
