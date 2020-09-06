# pukiwiki-instance

## pre-deploy

リカバリ用の[スナップショット](https://ap-northeast-1.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#Snapshots:sort=desc:startTime)のSnapshotIdを取得


```sh
# e.g.
SNAPSHOT_ID=snap-0927dbdc6113398f0
```

## deploy

```sh
aws cloudformation create-stack \
    --stack-name pukiwiki \
    --parameters ParameterKey=SnapshotId,ParameterValue=${SNAPSHOT_ID} \
    --capabilities CAPABILITY_IAM \
    --template-body file://template.yaml
```

## Setup

- CloudFormationの出力の `ManagedInstanceSessionUrl` から、インスタンスのSessionをスタート
- `ec2-user` に変更

```sh
sudo su - ec2-user
```

### Install libraries

```sh
sudo yum -y update
sudo yum -y install docker git
docker --version
  # Docker version 19.03.6-ce, build 369ce74
git --version
  # git version 2.23.3
```

- ec2-userをdockerグループへ追加

```sh
sudo gpasswd -a $USER docker
  # Adding user ec2-user to group docker
```

- dockerを起動

```sh
sudo systemctl enable docker
  # Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
sudo systemctl start docker
```

### Install docker-compose

```sh
DOCKER_COMPOSE_VERSION=1.26.2
sudo curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
  # docker-compose version 1.26.2, build eefe0d31
```

### Mount wiki volume

```sh
lsblk
  # NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
  # xvda    202:0    0   8G  0 disk
  # └─xvda1 202:1    0   8G  0 part /
  # xvdf    202:80   0  17G  0 disk
```

```sh
sudo mkdir /wiki
sudo mount /dev/xvdf /wiki
```

- fstabに設定（自動マウント）

```sh
sudo xfs_admin -u /dev/xvdf
  # UUID = 224bb184-ea0e-4de5-b0c7-3c159dd23882
echo "$(sudo xfs_admin -u /dev/xvdf | tr -d ' ')     /wiki       xfs    defaults          0   0" | sudo tee -a /etc/fstab
  # UUID=224bb184-ea0e-4de5-b0c7-3c159dd23882     /wiki       xfs    defaults          0   0
```

### Startup Wiki

```sh
pwd
  # /home/ec2-user
git clone https://github.com/nemodija/pukiwiki-compose.git
cd pukiwiki-compose/
git submodule init
git submodule update
```

*docker-compose up*

- https://github.com/nemodija/pukiwiki-compose
