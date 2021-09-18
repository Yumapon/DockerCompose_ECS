# このリポジトリに必要なもの

## secret

* GIT_ACCESS_TOKEN  
    Githubのアクセストークン。  
    取得方法は[こちら](https://docs.github.com/ja/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token)  
    ちなみに、与える権限はrepoだけで良さそうだけど未検証なので全権限与えといてください  

## ECSへのデプロイ

### 事前準備（実施するのは最初の一回だけ。一回設定できたらそのあとは不要）

* AWS用コンテキストの作成

    [こちら](https://docs.docker.com/cloud/ecs-integration/)確認

* ECSの設定確認

    ```sh
    aws ecs list-account-settings --effective-settings

    # nameにLongArnFormatが入っている設定が全てenableであることを確認。
    # もしdisableになっていれば下記コマンドで有効化
    $ aws ecs put-account-setting-default --name containerInstanceLongArnFormat --value enabled
    $ aws ecs put-account-setting-default --name serviceLongArnFormat --value enabled
    $ aws ecs put-account-setting-default --name taskLongArnFormat --value enabled

    #結果Sample
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

* DockerhubのTokenを作成

    [こちら](https://docs.docker.com/docker-hub/access-tokens/)確認  
    ※作成したTokenはメモっといてください

* DockerHubのアクセスTokenをAWS Secret managerに登録
  
    ```sh
    #Dockerhub AccessToken作成に必要なファイルを作成
    vi token.test
    #中身はこんな感じ
    # {
    #   "username":"Dockerhubのユーザ名",
    #   "password":"先の『DockerhubのTokenを作成』で作成したトークンを貼り付け"
    # }
    

    #登録
    docker secret create dockerhubAccessToken token.json
    # なんかでけた
    # arn:aws:secretsmanager:ap-northeast-1:030073904594:secret:dockerhubAccessToken-O31KAg
    #上記で作成したsecretacccesstokenをdocker-composeのx-aws-pull_credentials:に定義する
    ```

### Deploy本番

* こっからが本当のDeploy

    ```sh
    #AWS用のコンテキストに切り替え
    docker context use dcdeploy

    #デプロイ
    docker compose up

    #削除
    docker compose down
    ```

## RDS設定

```sql

```

## 残課題

 Dockerhubに格納した時点でDB接続情報を入れてしまっているため、
 Docker compose up　の時点で入れ込むようにSpring側の改修が必要
