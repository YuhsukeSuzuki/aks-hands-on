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

`$docker tag nginx <ACRレジストリ名>.azurecr.io/<リポジトリ名（任意）/hello-world-express`

例:`$docker tag nginx myacr.azurecr.io/myrepogitory/hello-world-express`

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

`$kubectl get comfigmaps`

コンフィグマップは Kubernetes クラスタ上に存在する KVS（etcd）に格納されている。

次に、Nginx のデプロイメントに以下の内容を追記する。

'''
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

'''

※ 追記済みのYAMLは nginx-deployment-proxy.yaml 

以下のコマンドで、Nginx のデプロイメントを更新する。

`$kubectl apply -f nginx-deployment.yaml`

以下のコマンドで、Nginx の Pod が更新されることを確認する。（Pod名が変更になっていれば更新完了）

`$kubectl get pods`

以下のコマンドで、Nginx の Pod 内のコンテナに接続し、/etc/nginx/nginx.conf の内容が ConfigMap のものと同一であることを確認する。

`$kubectl exec <NginxのPod名> -it /bin/bash`
`#cat /etc/nginx/nginx.conf`

以上で、ブラウザから http://<External IP>/app/ へアクセスし、Hello Worldが表示されれば正常。（URL末尾の/は必須）


