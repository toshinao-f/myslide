## FITAAT : Docker vs PCF チーム
第3回 コンテナまとめ


---

### Docker vs PCF ?

---

### ~~Docker vs PCF ?~~

---

### Docker = コンテナ = PCF
### つまり、コンテナ

---

### 今期は、コンテナ関連のまとめということで

---

## そもそもコンテナとは
- 1つのサーバーの上で、複数のサーバー環境をそれぞれ分離して実行する仮想化技術
- 仮想マシンではなく、コンテナ
    - 1つのOSの中を複数の区画に分離する技術
    - 物理マシンのエミュレートはしない
    - 1つのカーネルの上で複数のOS環境が動くイメージ

---

## Dockerとは
- 端的いうと、コンテナー管理ツール
- ファイルシステムの差分管理(イメージ=層の積み重ね)
- 公開されたイメージリポジトリが存在する
- アプリケーションのポータビリティ
- PaaSやクラウドサービスとの相性がよい

---

## Docker Composeツール
- 複数のコンテナを使うアプリケーションを、定義・実行するツール
- 開発環境、テスト環境ではいい働きをする
- 単一ホストへのデプロイ

---

## Docker管理サービス（本家？）
- Docker Cloud(https://cloud.docker.com/)
    - Dockerイメージ管理のSaaS(マルチクラウド)
- Docker Enterprise Edition(https://www.docker.com/enterprise-edition)
    - CaaS(Container as a Service)
    - Dockerイメージの展開環境を作るソフト（オンプレ、クラウド）

---

## Docker管理サービス（PaaS）
- OpenShift（k8sベース）
    - RedHat主導のコンテナプラットフォーム
- Deis workflow（k8sベース）
    - Microsoftが買収 => Azureと連携（Draft）
    - 現在、更新なし？？
- Pivotal Cloud Foundary（CFベース）
    - みなさんおなじみ、Pivotal社のPaaS

---

## Docker管理サービス（PaaS）
- 各コンテナ型のPaaSの違いがよくわからない
    > 例えば、gitやCLIツールからデプロイができ、DockerfileやBuildpackのような複数言語を扱う仕組みが使えて、コマンド一発で起動・停止・スケールアウトができ、リクエストルータもあってマルチホストに展開ができる。 ー＞[ひしめき合うOpen PaaSを徹底解剖！ PaaSの今と未来](https://www.slideshare.net/jacopen/openpaas-paas)
- 12 Factor app が前提にあるので、自ずと似通ったものになる・・のか
- 差別化するのであれば、CF系 vs k8s系 みたいな構図か？ ([CF vs OpenShift](https://comparisons.financesonline.com/openshift-vs-cloud-foundry) )

---

## Docker管理サービス（クラウドサービス）
- Amazon EC2 Container Service (ECS)
- Google Container Engine (GKE)

...etc

---

## Docker管理サービス（クラウドサービス）
#### GKE とは
- k8sを用いたマネージドサービス
- CLI で操作が可能
    - gcloud,kubectl,docker
- クラスタ構築は、GCE のインスタンスを用いる
- Docker コンテナをサポート

---

## Docker管理サービス（クラウドサービス）
#### ECS とは
- Amazonが独自開発したコンテナのオーケストレーションサービス
- CLI で操作が可能
    - AWS CLI,docker
- クラスタ構築は、EC2 のインスタンスを用いる
- Docker コンテナをサポート

---

## 違いがわからない

---

## Docker管理サービス（クラウドサービス）
#### 構成要素（ざっくり）
- GKE　[k8sアーキテクチャ](https://github.com/nkhare/container-orchestration/blob/master/kubernetes/README.md) 
    - コンテナ（Dockerイメージ）
    - Pod (1〜nコンテナの集まり）
    - Node（=VMインスタンス,Podの集まり+k8s実行環境）
    - Service（Podとの接続を抽象化）
    - ReplicaSet（Pod内のコンテナ数を維持/保証）
    - Master(Podの割り当て（Scheduler）など)
    - Cluster（1のMasterと1〜nのNodeで構成）
        - Pod, Service, ReplicaSetの基盤

=> k8sベースのためか、複雑。。

---

## Docker管理サービス（クラウドサービス）
#### 構成要素（ざっくり）
- ECS　[アーキテクチャ](https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/Welcome.html)
    - コンテナ（Dockerイメージ）
    - task（1〜10のコンテナの集まり,Jsonで定義）
    - task scheduler（ cluster への task 配置を担当）
    - cluster（コンテナインスタンスの論理グループ化）
    - container agent（taskの開始停止を管理）

=> k8sと比べたら、比較的シンプル？

---

## Docker管理サービス（クラウドサービス）
#### クラスタ数を比較
|GKE|ECS|
|:---:|:---:|
|ゾーンあたりの最大クラスタ数: 50|リージョンあたりの最大クラスタ数: 1,000|
|クラスタあたりの最大ノード数: 2,000|クラスタあたりの最大コンテナインスタンス数: 1,000|

---

## Docker管理サービス（クラウドサービス）
#### コスト
一概には難しいので、参考情報だけ
- GKE
    - GCEインスタンスはクラスタ内のノードとして使用される
    - これらのインスタンスごとに課金される
- ECS
    - AWS リソース（EC2インスタンス、EBSボリュームなど）に対してのみ料金が発生

---

## Docker管理サービス（クラウドサービス）
#### レジストリサービス
|サービス|利用料金(per GB per Month)|
|:---:|:---:|
|Google Container Registry|$0.026|
|Amazon EC2 Container Registry|$0.10|

---

## Docker管理サービス（クラウドサービス）
#### オートスケール
- GKE
    - Horizontal Pod Autoscaling（水平Podオートスケール）
    - Cluster Autoscaler
- ECS
    - Service Auto Scaling
        - task, clusterのスケールアウトイン


=> どちらも、クラスタレベル、コンテナレベルのオートスケール可能

---

## Docker管理サービス（クラウドサービス）
#### アクセス制御
- GKE
    - クラスタに IAM のロールを設定することが可能
    - 役割ベースのアクセス制御（RBAC）
        - クラスタ内のきめ細かなアクセス制御が可能
        - ただし、現在(2018/2)はベータ版のみ提供
- ECS
    - タスクに IAM のロールを設定することが可能
    - コンテナ単位で細かい制御が可能

=> ECSのほうが、より細かいリソース制御が可能？

---

## Docker管理サービス（クラウドサービス）
#### ロギング
- GKE
    - Stackdriver logging
- ECS
    - Cloud watch logs

=> Stackdriver loggingは見やすい？

---

## 雑感
#### 結局何を使うのか
- まず、ECSかGKEのようなクラウドサービスを検討する？
- オンプレ必要性や要件等で、PaaS製品とか？
- 開発環境はもっぱらDockerを使うともので、Dockerベースで考えていきたい。。
- コンテナ地獄ってなんですか？？

---

## 雑感
#### コンテナのメリット
- わずかな時間で検証環境の構築が可能（特にローカルでは感動的）
    - 試したい時に構築し、終わったら即削除
- ポータビリティ性が高い
    - docker-compose.ymlをリポジトリ管理することで、ソース取得環境ごとに同等の環境を用意できる
- 起動が高速！

---

## 雑感
#### コンテナのデメリット
- ローカルでは神がかっているが、それ以降の環境での影響は不明
    - 実績がない（よくわからない）
- Volumeを使用する場合、ホストOS側に依存する？
    - docker-composeではCOPYやADDが使いづらいので,Volumeを使う。。

---

## 雑感
#### PaaS
- PaaS製品はどれも有用に思うが、PCF以外実績がない
    - 大きな違いがわからないので、実績（PCF）ベースで採用か
    - OpenShiftとかどうなのか？

---

## 雑感
#### ECSとGKE
 - オペレーションの使い勝手は、GKEの方が良さげ
    - gcloud,kubectlの方が直感的で扱いやすい
    - aws cliはjsonだし、挙動が不安定だし
    - Google Cloud Shell が便利
- ECSはガッチリしている（イメージ）
    - アクセス制御が厳密
    - DB（Aulora）の安定性が高い？
- 簡単に試すならGKE, ちゃんとやるならECS？

---

## 最後に

---

# 今すぐ、Docker　を　いれましょう！
自席に戻ったらいれてから、帰りましょう

---

以上

---
