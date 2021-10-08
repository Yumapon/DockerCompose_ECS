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

* Deploy

    docker-composeファイルでCDは不可。
    最初の一回だけECSに展開可能。

    やるのであれば以下コマンドで出力したCFnTemplateを使用して管理していくしかなさそう。

    ```sh
    docker conpose convert >> convert.yaml
    ```

* docker-composeで最初のECSへのデプロイ

    ```sh
    docker context use [上で作成したAWS用コンテキストを使用]

    export SPRING_DATASOURCE_URL={作成したRDSのエンドポイント}
    export SPRING_DATASOURCE_USERNAME={RDSのユーザネーム}
    export SPRING_DATASOURCE_PASSWORD={RDSのパスワード}
    export SPRING_DATASOURCE_DRIVERCLASSNAME={RDSで使用しているRDBMSのドライバー名}

    docker compose up
    ```

## RDS設定

Amazon RDSの画面から『データベースの作成』を押下

データベースの設定画面で、下記の通り設定

* データベース作成方法を選択
    標準作成

* エンジンのオプション

    MySQL (Oracleを選択するとライセンス料等でかなり高額になる)

* エディション

    MySQL Community

* バージョン
    MySQL 8.0.23

* テンプレート
    無料利用枠

* DBインスタンス識別子
    taskappdatabase

* マスターユーザー
    admin

* マスターパスワード
    ******

* DBインスタンスクラスとストレージ、可用性と耐久性はデフォルトのまま

* 接続
    パブリックアクセスの設定を『あり』にし、
    VPCセキュリティグループは新規作成、名前は適当に決める
    その他はデフォルトのまま

* データベース認証
    パスワード認証

上記設定ができれば、データベースの作成を押下

DB作成後、セキュリティグループのインバウンドが/32のアドレスになっているので
ECSからの通信を許可するために全アケしちゃう

データベースの一覧画面で、作成できたことが確認できたら『接続とセキュリティ』タブから
エンドポイントとポートをメモ

* DB作成クエリ

```sql
create database taskappdatabase;

use taskappdatabase
```

* テーブル作成クエリ

```sql
CREATE TABLE TASK_LIST
(
    NUM VARCHAR(500) NOT NULL,
    DEADLINE DATE,
    NAME VARCHAR(200) NOT NULL,
    CONTENT VARCHAR(2000),
    CLIENT VARCHAR(100),
    PRIMARY KEY (NUM)
);

CREATE TABLE USER_ID
(
    ID int NOT NULL,
    PASSWORD VARCHAR(20),
    PRIMARY KEY (ID)
);
```

* データ作成のクエリ

```sql
INSERT INTO USER_ID (ID, PASSWORD) VALUES(100, 'password');
```

## 残課題

 Dockerhubに格納した時点でDB接続情報を入れてしまっているため、
 Docker compose up　の時点で入れ込むようにSpring側の改修が必要

 backendはv.1.1.8がうまく動くことを確認できてる。
 （CIとかいじって動かなくなったらこれ使う）

→ close
