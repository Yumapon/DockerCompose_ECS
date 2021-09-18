# このリポジトリに必要なもの

## secret

* GIT_ACCESS_TOKEN  
    Githubのアクセストークン。  
    取得方法は[こちら](https://docs.github.com/ja/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token)  
    ちなみに、与える権限はrepoだけで良さそうだけど未検証なので全権限与えといてください  

## ECSへのデプロイ（仕掛中）

### 事前準備

* コンテキストの作成

[こちら](https://docs.docker.com/cloud/ecs-integration/)確認

* ECSの設定確認

```sh
aws ecs list-account-settings --effective-settings

# nameにLongArnFormatが入っている設定が全てenableであることを確認。
# もしdisableになっていれば下記コマンドで有効化
$ aws ecs put-account-setting-default --name containerInstanceLongArnFormat --value enabled
$ aws ecs put-account-setting-default --name serviceLongArnFormat --value enabled
$ aws ecs put-account-setting-default --name taskLongArnFormat --value enabled

#結果
# {
#    "settings": [
#        {
#           "name": "awsvpcTrunking",
#           "value": "disabled",
#           "principalArn": "arn:aws:iam::030073904594:user/VSCodeUser"
#        },
#        {
#            "name": "containerInsights",
#            "value": "disabled",
#            "principalArn": "arn:aws:iam::030073904594:user/VSCodeUser"
#        },
#        {
#            "name": "containerInstanceLongArnFormat",
#            "value": "enabled",
#            "principalArn": "arn:aws:iam::030073904594:user/VSCodeUser"
#        },
#        {
#            "name": "containerLongArnFormat",
#            "value": "enabled",
#            "principalArn": "arn:aws:iam::030073904594:user/VSCodeUser"
#        },
#        {
#            "name": "dualStackIPv6",
#            "value": "enabled",
#            "principalArn": "arn:aws:iam::030073904594:user/VSCodeUser"
#        },
#        {
#            "name": "serviceLongArnFormat",
#            "value": "enabled",
#            "principalArn": "arn:aws:iam::030073904594:user/VSCodeUser"
#        },
#        {
#            "name": "taskLongArnFormat",
#            "value": "enabled",
#            "principalArn": "arn:aws:iam::030073904594:user/VSCodeUser"
#        }
#    ]
# }

```

* dockerhubのアクセストークン作成

[こちら](https://docs.docker.com/docker-hub/access-tokens/)確認

* docker composeイメージのプッシュ(CIでやってるのでこれいらない)

```sh
docker login --username keropon48
#さっきのアクセストークンを聞かれるので、入力

docker-compose push
```

## ECSのデプロイ

```sh
#AWS用のコンテキストに切り替え
docker context use dcdeploy

#Tokenの作成
docker secret create dockerhubAccessToken token.json
# なんかでけた
# arn:aws:secretsmanager:ap-northeast-1:030073904594:secret:dockerhubAccessToken-O31KAg

#デプロイ
docker compose up

#削除
docker compose down
```
