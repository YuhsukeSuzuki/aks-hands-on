# Azure Kubernetes Service ハンズオン

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

`Kubernetes master is running at https://myakscluster-dns-1283394f.hcp.japaneast.azmk8s.io:443`
`healthmodel-replicaset-service is running at https://myakscluster-dns-1283394f.hcp.japaneast.azmk8s.io:443/api/v1/namespaces/kube-system/services/healthmodel-replicaset-service/proxy`
`CoreDNS is running at https://myakscluster-dns-1283394f.hcp.japaneast.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy`
`kubernetes-dashboard is running at https://myakscluster-dns-1283394f.hcp.japaneast.azmk8s.io:443/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy`
`Metrics-server is running at https://myakscluster-dns-1283394f.hcp.japaneast.azmk8s.io:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy`

AKS クラスタ上のノードが正常であることを確認する。

`$kubectl get nodes`

以下のような出力が得られれば正常。

`NAME                       STATUS   ROLES   AGE   VERSION`

`aks-agentpool-19694923-0   Ready    agent   18d   v1.14.6`

`aks-agentpool-19694923-1   Ready    agent   18d   v1.14.6`

`aks-agentpool-19694923-3   Ready    agent   18d   v1.14.6`

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
ハンズオンキットのディレクトリに移動し、以下のコマンドを実行する。

`$kubectl apply -f ./nginx-deployment.yaml`

以下のコマンドで、Pod が起動したことを確認する。

`kubectl get pods`

表示された Pod 名をコピーして、以下のコマンドで詳細を確認する。

`kubectl describe pods <Pod名>`

Pod に bash で接続する。

`kubectl exec <Pod名> -it /bin/bash`

この状態では、まだ Nginx へのサービスエンドポイントが作成されていないため、Nginx にHTTPアクセスすることができないため、次項でサービスを定義して Nginx を起動する。

## Nginx 用のサービスを定義する。

以下のコマンドを実行する。

`$kubectl apply -f ./nginx-service.yaml`

以下のコマンドで、サービスが作成されることを確認する。

`$kubectl get services -w`

サービスが作成され、External IP がアサインされるまで数分を要する。
External IP が付与されたら、ローカルPCのブラウザを使用して http://<External IP>/ へアクセスし、Nginx のページが表示されることを確認する。







