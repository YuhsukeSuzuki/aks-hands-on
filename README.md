# Azure Kubernetes Service ハンズオン

## 概要
本ハンズオンでは、下図の構成をAKS環境上に構築することを目的とする。
![ハンズオン構成図](https://raw.githubusercontent.com/wiki/YuhsukeSuzuki/aks-hands-on/images/overview.png)

本ハンズオンでは、以下の項目を取り扱う。
- Azure Conainer Registry へのイメージの Push と Azure Kubernetes Service との連携
- Azure Kubernetes Service への Pod のデプロイ
- Azure Kubernetes Service へのサービスの定義
- 外部アクセス可能なサービスの定義と Azure Load Balancer
- Kubernetes クラスタ内での Pod 間の通信
- ConfigMap を利用したサーバ設定ファイルの注入
- Pod のスケーリング
- ノードのスケーリング

## 事前準備
- AKS クラスターの作成
- ACR レジストリの作成

## ハンズオンキットの準備
このGitリポジトリを clone する。

`$git clone https://github.com/YuhsukeSuzuki/aks-hands-on.git`

## 環境の確認
kubectl が AKS に接続できることを確認する。

`$kubectl cluster-info`

以下のような出力が得られれば正常。

```
Kubernetes master is running at https://myakscluster-dns-1283394f.hcp.japaneast.azmk8s.io:443
healthmodel-replicaset-service is running at https://myakscluster-dns-1283394f.hcp.japaneast.azmk8s.io:443/api/v1/namespaces/kube-system/services/healthmodel-replicaset-service/proxy
CoreDNS is running at https://myakscluster-dns-1283394f.hcp.japaneast.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://myakscluster-dns-1283394f.hcp.japaneast.azmk8s.io:443/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy
Metrics-server is running at https://myakscluster-dns-1283394f.hcp.japaneast.azmk8s.io:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

AKS クラスタ上のノードが正常であることを確認する。

`$kubectl get nodes`

以下のような出力が得られれば正常。

```
NAME                       STATUS   ROLES   AGE   VERSION
aks-agentpool-19694923-0   Ready    agent   18d   v1.14.6
aks-agentpool-19694923-1   Ready    agent   18d   v1.14.6
aks-agentpool-19694923-3   Ready    agent   18d   v1.14.6
```

## Nginx コンテナの準備
docker-hub から nginx のコンテナイメージを pull する。

`docker pull nginx`

pull したイメージをタグ付けする。

`$docker tag nginx <ACRレジストリ名>.azurecr.io/<リポジトリ名（任意）/nginx`

例:`$docker tag nginx myacr.azurecr.io/myrepogitory/nginx`

イメージのタグ付けの結果を確認する。

`$docker image ls`

nginx と同じハッシュ値でタグ付けしたイメージが存在しているのを確認する。



ACR レジストリにログインする。

`$az acr login -name <ACRレジストリ名>`

ACR レジストリにイメージをプッシュする。

`$docker push <ACRレジストリ名>.azurecr.io/<リポジトリ名（任意）/nginx`

ブラウザから Azure Portal にログインして、ACR 上にリポジトリが作成されていることを確認する。

## Nginx のポッドを起動する。
ハンズオンキットのディレクトリに移動し、nginx-deployment.yaml を編集する
- 16 行目の image を、先ほど ACR に push したイメージのイメージ名に変更する。

以下のコマンドを実行する。

`$kubectl apply -f ./nginx-deployment.yaml`

以下のコマンドで、Pod が起動したことを確認する。

`kubectl get pods`

表示された Pod 名をコピーして、以下のコマンドで詳細を確認する。

`kubectl describe pods <Pod名>`

Pod に bash で接続する。

`kubectl exec <Pod名> -it /bin/bash`

この状態では、まだ Nginx に外部から接続可能なサービスエンドポイントが作成されていないため、Nginx にHTTPアクセスすることができないため、次項でサービスを定義して Nginx を起動する。

## Nginx 用のサービスを定義する。

以下のコマンドを実行する。

`$kubectl apply -f ./nginx-service.yaml`

以下のコマンドで、サービスが作成されることを確認する。

`$kubectl get services -w`

サービスが作成され、External IP がアサインされるまで数分を要する。
External IP が付与されたら、ローカルPCのブラウザを使用して http://<External IP>/ へアクセスし、Nginx のページが表示されることを確認する。

# hello-world-express の準備

docker-hub から yuhsuke/hello-world-express のコンテナイメージを pull する。

`docker pull yuhsuke/hello-world-express`

pull したイメージをタグ付けする。

`$docker tag yuhsuke/hello-world-express <ACRレジストリ名>.azurecr.io/<リポジトリ名（任意）/hello-world-express`

例:`$docker tag yuhsuke/hello-world-express myacr.azurecr.io/myrepogitory/hello-world-express`

イメージのタグ付けの結果を確認する。

`$docker image ls`

yuhsuke/hello-world-express と同じハッシュ値でタグ付けしたイメージが存在しているのを確認する。



ACR レジストリにログインする。

`$az acr login -name <ACRレジストリ名>`

ACR レジストリにイメージをプッシュする。

`$docker push <ACRレジストリ名>.azurecr.io/<リポジトリ名（任意）/hellow-world-express`

ブラウザから Azure Portal にログインして、ACR 上にリポジトリが作成されていることを確認する。

# hello-world-express のポッドを起動する。

ハンズオンキットのディレクトリに移動し、app-deployment.yaml を編集する
- 16 行目の image を、先ほど ACR に push したイメージのイメージ名に変更する。

以下のコマンドを実行する。

`$kubectl apply -f ./app-deployment.yaml`

以下のコマンドで、Pod が起動したことを確認する。

`kubectl get pods`

表示された app-xxxx の Pod 名をコピーして、以下のコマンドで詳細を確認する。

`kubectl describe pods <Pod名>`

# hello-world-express のサービス定義
Nginx から内部ネットワークで通信するために、hello-world-express に対してサービスの定義を行う。

以下のコマンドを実行する。

`$kubectl apply -f ./app-service.yaml`

以下のコマンドで、サービスが作成されることを確認する。

`$kubectl get services`

ClusterIP タイプのサービスは、瞬時に作成される。

# Nginx の設定（ConfigMapの作成とDeploymentの変更）
Nginx から hello-world-express にプロキシ設定を行うために、Kubernetes の ConfigMap を使用して、起動する Nginx の Pod に設定を行う。
コンテナ内の /etc/nginx を ConfigMap で定義した内容で入れ替える。

ConfigMap を定義する前に、以下のコマンドで現在の Nginx ポッド内のディレクトリ構造を確認する。

`$kubectl exec <NginxのPod名> -it /bin/bash`
`#ls /etc/nginx`

以下のコマンドを実行する。

`$kubectl apply -f ./nginx-configmap.yaml`

以下のコマンドで、ConfigMap が定義されたことを確認する。

`$kubectl get cofigmaps`

コンフィグマップは Kubernetes クラスタ上に存在する KVS（etcd）に格納されている。

次に、Nginx のデプロイメントに以下の内容を追記する。

```
    spec:
      containers:
        ...（略）
        # ここから下を追記
        volumeMounts:
        - mountPath: /etc/nginx
          readOnly: true
          name: nginx-conf
      volumes:
      - name: nginx-conf
        configMap:
          name: nginx-configmap
          items:
            - key: nginx.conf
              path: nginx.conf

```

※ 追記済みのYAMLは nginx-deployment-proxy.yaml 

以下のコマンドで、Nginx のデプロイメントを更新する。

`$kubectl apply -f nginx-deployment.yaml`

以下のコマンドで、Nginx の Pod が更新されることを確認する。（Pod名が変更になっていれば更新完了）

`$kubectl get pods`

以下のコマンドで、Nginx の Pod 内のコンテナに接続し、/etc/nginx/nginx.conf の内容が ConfigMap のものと同一であることを確認する。

`$kubectl exec <NginxのPod名> -it /bin/bash`
`#cat /etc/nginx/nginx.conf`

以上で、ブラウザから http://<External IP>/app/ へアクセスし、Hello Worldが表示されれば正常。（URL末尾の/は必須）

## Pod のスケーリング
デプロイ済みの Hello-world-express アプリをスケーリングする。以下のコマンドを実行する。

`$kubectl scale --replicas=2 deployment/app`

`$kubectl get pods`を実行し、app の Pod が２つになっていることを確認する。

（スケーリング前に別ターミナルで、`$kubectl get pods -w` を実行することで、Pod が新しく作成され、起動するまでを確認することができる）


## Pod への負荷分散の確認
サービスは、Podに対する負荷分散の機能を有している。今回の環境でも、app のポッドが増加したことにより、負荷分散が行われている。
ここでは、新たにターミナルを二つ立ち上げ、負荷分散が行われていることを確認する。

立ち上げた2つのターミナルそれぞれで、以下のコマンドを実行する。

`$kubectl logs <Pod名> -f`
※ Pod 名は各々のターミナルで異なるものを使用する。

ブラウザ、もしくは Curl を使用して、http://<External IP>/app/ へアクセスすると、どちらかのターミナルに以下のようなアクセスログが表示される。

```
Hello World listening on port 3000!
::ffff:10.244.3.13 - - [08/Oct/2019:06:33:54 +0000] "GET / HTTP/1.0" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36"
```
何度もアクセスを繰り返すことにより、アクセスログがそれぞれのターミナルに記録されることで、ロードバランシングが行われていることを確認する。

## Node のスケール
AKSを構成しているノードプールの数を、az aks コマンドで変更することができる。この作業の完了には数分を要する。
以下のコマンドを実行する。

`$az aks scale --resource-group <リソースグループ名> --name <AKSクラスタ名> --node-count <スケールさせたいノード数>`

ノード数は増減させることが可能である。ただし、ノード数を減少させる場合はランダムで選択されたノード（通常は最も最近作成されたノード）上のPodが停止するため、レプリカ数が１の Pod が配置されていた場合には、サービス停止時間が発生してしまうことに注意する。

なお、スケール時に以下のコマンド等で外部からの応答を確認する。

`$while :; do curl http://<External IP>/app/ ; sleep 1; done` 

## Podのログの確認
コンテナは揮発性があるため、標準ではログは永続化されず、標準出力へ標準エラー出力がリダイレクトされたものがログとして出力される。
以下のコマンドを実施して、Podの標準出力を確認する。

`$kubectl logs <Pod名>`

例:`$kubectl logs nginx-c77fbcd56-67wx4 `

## fluentd + elasticsearch + Kibana によるログ収集環境の構築
[公式のアドオン](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch)を使用して、fluentd + elasticsearch + Kibana でログを収集するための環境を構築する。

Kubernetesクラスタ上に、"monitoring"という名前で**namespace**を作成する。

`$kubectl create namespace monitoring`

ハンズオンキットのloggingディレクトリをすべてクラスタに適用する。

`$kubectl apply -f logging/`

"kibana-logging" サービスに External IP が割り当てられるまで待つ。

Kibanaにアクセスする。

`http://<External IP>:5601/`

※ 必ず、chrome/fire fox で開くこと。

![Kibana画面](https://raw.githubusercontent.com/wiki/YuhsukeSuzuki/aks-hands-on/images/kibana.png)

Kibanaの画面が表示されたら、"my own ..." を選択後、Home 画面で "Manage and ..." -> "Index Patterns" -> "Create New Index" と選択。

Create Index の1ページ目で、Index Pattern  に `logstash-*` と入力し、Next Step。

2ページ目で、Time Filter Field を @timestamp を選択。


以上で、左側メニューのDiscoverから、クラスタのノード、およびPodから収集してきたログを確認することができる。


## Azure Monitor for Containers
Azure Monitor による監視を有効化する。

![Azure Portal画面](https://raw.githubusercontent.com/wiki/YuhsukeSuzuki/aks-hands-on/images/amfc.PNG)


Azure Portal から、対象のAKSクラスタのブレードを表示させ、左側のメニューで"監視"の"ログ"を選択する。

Log Analytics のワークスペースを選択し、"有効化" ボタンを押下する。

Azure Monitor for Containers の機能が有効化され、Azure Portal 上でクラスタの状態を確認することができるようになる。

## Kubernetes Dashbord
以下のコマンドでKubernetes Dashbordへ接続するローカル・プロキシが作成される。

`$az aks browse --resource-group <リソースグループ名> --name <AKSクラスタ名>`

※ Teraterm などでターミナル接続を行っている場合はSSHポート転送設定が必要

## helm を使用した Grafana の設定

以下のコマンドを実行する。
`$kubectl apply -f rbac/helm-tiller.yaml`
`$sudo snap install helm --classic`
`$helm init --service-account tiller --upgrade`
`$helm helm repo add coreos https://s3-eu-west-1.amazonaws.com/coreos-charts/stable/`
`$helm install coreos/prometheus-operator --name prometheus-operator --namespace monitoring`
`$helm install coreos/kube-prometheus --name kube-prometheus --namespace monitoring`

デフォルトでは外部からGrafanaにアクセスできないので、サービスを修正する。
`$kubectl apply -f kube-prometheus-grafana.yaml`

