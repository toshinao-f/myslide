## SpringBootアプリをGKEにデプロイ

#### アカウント作成
- [公式サイト](https://console.cloud.google.com/)でアカウント作成
- 12ヶ月は無料で利用可能(300ドル分)
- クレジットカード登録が必要

#### Cloud SDKインストール
- [公式サイト](https://cloud.google.com/sdk/docs/quickstarts?hl=ja)より、アーカイブ ファイルをダウンロード
- ダウンロード後は、任意のフォルダに解凍
    - この例では、MaxOX用を``` ~/inst ```配下に解凍
    ```
    wget https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-170.0.1-darwin-x86_64.tar.gz?hl=ja
    tar zxvf　google-cloud-sdk-170.0.1-darwin-x86_64.tar.gz
    mv google-cloud-sdk ~/inst
    cd ~/inst/google-cloud-sdk
    ./install.sh
    echo 'export PATH="/Users/[ユーザID]/inst/google-cloud-sdk/bin:$PATH"' >> ~/.bash_profile
    source ~/.bash_profile
    ```
- プロジェクトを初期化
```
gcloud init
# 以降対話形式　途中でログインする
```

#### Cloud SQL
- CloudSQLとは
    - Google Cloud SQL は、Google Cloud Platform 上のリレーショナル データベースの設定、保守、運用、管理を簡単にできるようにするフルマネージド データベース サービス
    - MySQLとPostgreSQLより選択
    - なお、通常のインスタンスと比べ、サポート機能に[制約](https://cloud.google.com/sql/docs/features?hl=ja)あり
- Cloud SQLを作成する（MySQL）
```
gcloud sql instances create dploydemo-db --region=asia-northeast1 --database-version=MYSQL_5_7 --tier=db-n1-standard-1
```

- ユーザを作成する
```
gcloud sql users create mysql % --instance=dploydemo-db --password=mysql
```

- データベースを作成する
```
gcloud sql databases create spring_cloud_db --instance=dploydemo-db
```

#### 資源をチェックアウトする
- アプリケーション
```
mkdir ~/github
cd ~/github
git clone git@github.com:toshinao-f/deploydemo.git
```

- Dockerfileなど
```
cd ~/github
git clone git@github.com:toshinao-f/docker-workshop.git
```


#### コンテナクラスタ作成
- Google Container Engine API を有効にしておく
- Google Cloud SQL API を有効にしておく
- kubectlをインストールし、クラスタを作成
```
gcloud components install kubectl
gcloud container clusters create deploydemo-cluster --zone=asia-northeast1-a
```

#### CloudSQL接続設定

- IAM管理コンソールにて、新規サービスアカウントを作成する
    - 役割は、CloudSQLクライアント、CloudSQL編集者
    - 新しい秘密鍵を作成(JSON)
    - JSONファイルがダウンロードされる

- アプリからCloudSQLへ接続可能にする
```
sudo mkdir /secrets
sudo mkdir /secrets/cloudsql
sudo mkdir /cloudsql
sudo mkdir /etc/ssl/certs
sudo chown xxx:xxx -R /secrets/
sudo chown xxx:xxx /cloudsql
sudo chown xxx:xxx /etc/ssl/certs
cp [ダウンドードした秘密鍵] /secrets/cloudsql/credentials_client.json
kubectl create secret generic cloudsql-instance-credentials --from-file=/secrets/cloudsql/credentials_client.json
kubectl create secret generic cloudsql-db-credentials --from-literal=username=mysql --from-literal=password=mysql
```

#### Dockerイメージを作成し、ContainerRegistryに登録
- Jarファイルを作成する
```
cd ~/github/deploydemo/
./mvnw package spring-boot:repackage -Dmaven.test.skip=true

```

- ビルドする
```
# jarファイルをDockerfile用のフォルダにコピー
cp ~/github/deploydemo/target/deploydemo-0.0.1-SNAPSHOT.jar ~/github/docker-workshop/PG/Dockerfiles/gke/app
# ビルド
export PROJECT_ID=[プロジェクトID]
docker build --tag=gcr.io/$PROJECT_ID/deploydemo:1.0.0 ./ --build-arg SPRING_JAR_FILE=deploydemo-0.0.1-SNAPSHOT.jar

```
- なおレジストリ形式は以下の通りのため、この形式でTagを設定する必要がある
```
[HOSTNAME]/[YOUR-PROJECT-ID]/[IMAGE]
> [HOSTNAME]= gcr.io
> [YOUR-PROJECT-ID]=プロジェクトID
> [IMAGE]=イメージ名
```

- Google Container Registryに登録する
```
gcloud docker -- push gcr.io/$PROJECT_ID/deploydemo:1.0.0
```

#### デプロイ
- Podを作成する
```
kubectl create -f deployment.yml --record
# 状況確認
kubectl get deployments,replicasets,pods --show-labels
```

- Serviceを作成する
```
kubectl create -f service.yml
# 状況確認
kubectl get services
```

- EXTERNAL-IPのIPアドレスにアクセスして、アプリが表示されることを確認する

- アップデートする
```
# 事前にdeployment.ymlを変更する
#
kubectl apply -f deployment.yml --record
# 状況確認
kubectl get deployments,replicasets,pods --show-labels
```

- ロールアウト履歴を確認する
```
kubectl rollout history deployment deploydemo
kubectl rollout history deployment deploydemo --revision=1
```

- ロールバックする
```
kubectl rollout undo deployment deploydemo --to-revision=1
```

- 後始末
```
kubectl delete service deploydemo-lb
kubectl delete deployment deploydemo
gcloud container clusters delete deploydemo-cluster --zone=asia-northeast1-a
```
