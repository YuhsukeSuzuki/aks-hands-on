# セキュリティ
## Namespace の作成

以下のコマンドを実行する。

```
  $kubectl apply -f security/namespaces.yaml
```

namespace が作成されたことを確認する。

```
  $kubectl get namespaces
```

以下の３つのnamespaceが追加されている。
- front-end
- back-end
- back-end-kj

## Podのデプロイ

以下のコマンドを実行する。

```
  $kubectl apply -f security/back-end-deployment.yaml
```

nginxの動作を確認するため、以下のコマンドを実行する。

```
  $kubectl run --rm -it --image=alpine client-pod --namespace back-end --generator=run-pod/v1 --generator=run-pod/v1
```

Podのシェルが起動したら、以下のコマンドで確認を行う。

```
  #wget -qO- http://back-end-nginx
```

以下のような出力が返ってくることを確認する。

```
  <!DOCTYPE html>
  <html>
  <head>
  <title>Welcome to nginx!</title>
  <style>
      body {
          width: 35em;
          margin: 0 auto;
          font-family: Tahoma, Verdana, Arial, sans-serif;
      }
  </style>
  </head>
  <body>
  <h1>Welcome to nginx!</h1>
  <p>If you see this page, the nginx web server is successfully installed and
  working. Further configuration is required.</p>

  <p>For online documentation and support please refer to
  <a href="http://nginx.org/">nginx.org</a>.<br/>
  Commercial support is available at
  <a href="http://nginx.com/">nginx.com</a>.</p>

  <p><em>Thank you for using nginx.</em></p>
  </body>
  </html>
```

Podからexitする。

```
  #exit
```

## サービスアカウント/Role/RoleBindingの作成
上記で作成した各namespaceへのアクセスを制御するため、サービスアカウント/Role/RoleBindingを作成する。（今回は、namespace: front-endのみ）

以下のコマンドを実行する。

```
  $kubectl apply -f security/front-end-user-role.yaml
```

以下のコマンドを実行してサービスアカウントを確認する。

```
  $kubectl get sa -n front-end
```

以下のコマンドを実行して、Roleを確認する。

```
  $kubectl get role -n front-end
```

以下のコマンドを実行して、RoleBindingを確認する。

```
  $kubectl get rolebinding -n front-end
```

## front-end-userでkubectlを使用する。
作成したサービスアカウントを使用して、kubectlを使用してクラスタの操作ができるように設定を行う。
今回は、サービスアカウントのシークレットで認証を行う。

### トークンと認証キーの確認
サービスアカウント作成時に、サービスアカウント用にトークンと認証キーが自動的に生成されている。
kubectlを使用して、サービスアカウントのトークンキーの取得をし、シークレットからアクセス用のトークンと認証キーを取得する。

以下のコマンドを実行して、サービスアカウントの詳細情報を表示する。

```
  $kubectl describe sa front-end-user -n front-end
```

以下のような内容が表示される。

```
  Name:                front-end-user
  Namespace:           front-end
  Labels:              <none>
  Annotations:         kubectl.kubernetes.io/last-applied-configuration:
                         {"apiVersion":"v1","kind":"ServiceAccount","metadata":{"annotations":{},"name":"front-end-user","namespace":"front-end"}}
  Image pull secrets:  <none>
  Mountable secrets:   front-end-user-token-8x2t7
  Tokens:              front-end-user-token-8x2t7
  Events:              <none>
```

*Tokens*の値をコピーする。（例では```front-end-user-token-8x2t7```）

先ほどの*Tokens*の値を使用し、以下のコマンドを実行する。

```
  $kubectl get secret <Tokensの値> -n front-end -o "jsonpath={.data.token}" | base64 -d
```

以下のような長い文字列が表示されるので、メモ帳などにコピーしておく。（tokenとしておく）

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJmcm9udC1lbmQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoiZnJvbnQtZW5kLXVzZXItdG9rZW4tOHgydDciLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZnJvbnQtZW5kLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI1MDFkYTgyNy0xYmRlLTExZWEtYTUyZC1jNmRhM2NlYzVlNTkiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZnJvbnQtZW5kOmZyb250LWVuZC11c2VyIn0.FvDQzmgvSpM3QdduwfBe82Ym4v41stKNSbkkq9QGdfzjfxyUv0GLHtqlK3mr2r_AZNXPpCmuK2LdC_mR0YDX-NsyjC-tyL0S09fHlKM1Q5OIaB8_-PJlQXlf0eG4MI9fd6sIharm03VkVE3XoFajy2uZkcByXUoZVjnwUtpZTIa2lyCTI45MOCLWOvhRWfBAe8mj8o_a8Y4sZYYuKeMZv6_ye7gi5MNQeVKOQBa0xfPVJqNQIpgYPx4_OZCmUJ8mKH2XDwHniSWoJiNqsRBC1ORD-JJNNxURVCqPpf5-eNV4BlWhGvycTM1BkWoyGb-SR9z8HDEY1_SvLr6Sl_-e9VrH9MmbgpureeoSRE3WrWJKnfz9lr5FpVfoqgYUjotQEHKav2Ddg29YaaWJcPjkGTw0_NXBrPpByjegK4doiTPDKqrHEAXsUj8DOWdk7TgkXgpxOJTsYHCnpkRD14SSvPKHGfEDwaf4H1ruubi4Yk2j5gkwvBDQZ0Wmp-_Nf_j1mdVAY4bAA1aXctJQtwajTHtchlfNxJFhSUlmx3_QDKxfxs6sZoSr-rmZ70nVfwdBMEVjQfBk3LNQ0JaGvBHpeq2rMjllDQ2N1RUlygauANrcZb-SkE1NtSmsGOJRuqh0f31eNUkwAbqJ1b2Fyai5ZtNrhXmlOc1hqXTkY7kXXXX
````

再び*Tokens*の値を使用し、以下のコマンドを実行する。

```
  $kubectl get secret <Tokensの値> -n front-end -o "jsonpath={.data['ca\.crt']}"
```

また長い文字列が表示されるので、これもコピーしておく。（client-key-dataとしておく）

```
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUV4ekNDQXErZ0F3SUJBZ0lRUW40blRsSjFTMGlGMG9Eb3I1MXFOekFOQmdrcWhraUc5dzBCQVFzRkFEQU4KTVFzd0NRWURWUVFERXdKallUQWVGdzB4T1RFeU1Ea3dPRFV6TXpKYUZ3MDBPVEV5TURFd09UQXpNekphTUEweApDekFKQmdOVkJBTVRBbU5oTUlJQ0lqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FnOEFNSUlDQ2dLQ0FnRUF2YmxmCnd4L3gwMUp5VlhFbUl4cmE4dndSQ01FMUdkNGVjbVJpUFNiV0RlM2EraXFwd1FDamh1dzhkaU5rYjVqMzNkTDUKazlncnNndytrS3JLQm00UUlXSnVKMkNLdVJtOWZJRWhPSjQrR1VSMkdlWVBXbTRnRjN0SGoxUXpwcHkzeEZjNwpKQUlRQ0Mza2ErUnZBRURmeWxaNktpanFsNDV6MTd3T2tCZjBvVGMwZHNmOEVFaTMyUFlqemdtd21QMVlIMWJVCjJ6MitJS2N6THo3UC9QSjdSeHFuWTBQdE5xRlJGT3JXVDgzMUtKUklvTEFxT1E2ZnAzcEZvUU1rV012eUR1NGUKb2s4UjQxOXN3OFFMUXBMNEtSUFlGajlQYjV1bEk0bFhVb2tlSDB2LytzbzFVR1haYURyZGlCZXpJM3d1MEFOWgp0VVN0OTM4c3kyZUphM3hUZkEzNklmSUpVeC9FYThsQkE1Wi8yYmtMZmdmWVU5Mk1hVzZ1ZVhNSDZwOUVwQWt6CmZCSVZ6RnpXeTh2bkxJZFAxSzBwemdCNFJIVWp4MkVTWjZTcE8yT0J0VG1kOU1SU3NhQStaMGZEQmpURWJiSlMKbVlYdDRiWmFTendCMkRrM3NkNzBONlN5dGREdjRBaE9LZG5jV3hxb0s1eElhL2haU2dweWhTQlJiVXV0TEl2bQp0dTk4QjJYaDY0NVJQN1Z5eDBjTXdIUXJHUHlISFR0SlR5cXBVRUF4MzBBemcvWWc4M3JsaWRtYW9ibVBRZG8vCjV2OTlMTWRWS2tlUWdDMHBPRkN4RG5EaDdGcG1WVW0xS0x1d0lXRlk5L2JFQlE4TW83QmdtTk9SN0VLZ3l5VTkKS1NuUWJSZGNDSUlXczMrS2RWSkpTeExlTUZKVmF1WW1FTkVOR2ZNQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFILwpCQVFEQWdLa01BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dJQkFBVnFMbUYwCjU2UUdSWVphZ1hINDFtV3p5ZFVmeDZESWwwM2FSOFN1N3JzQVBmM0hMOFBlR1kxeXJQRml4Vzhtc2V2a3BKbGoKWFpzcEVCd253VWhvMWttejljOTY5R0VtaWZ6WHRPdzR2WnQ4dmZGVkxRUFNwejZ2aWRuSEVzNHdmMXNuYzBzUwplUDNGWkNnaUVNcUpwTVN3QmN6ZittZGFxdGpuT1h1NzNBV2orM1JVdUpBem9sUUkvUnJkSy8zRjBMRFNPUUpZClhXUU9LV043VjdzclJDY0U0NEpBMUtRak5UTDhPT0xKb0ptSzhHWkc5MVVrSTZRWnVYcFd1Nlgvc29sWEhVZk0KNlBjdWlaYkw5eGRqUkNjN0IwMjZWOVlSK1F0TE9OekltMERqQ2RDaDlWc29GNEd1NDdCZzJVQjMwYjBwTzdBQQpPZkczZkdCVXdFZXV4RHpmWnlmMmErcjAyYzVybis1OWgrZHJYZVR3dnk1SXE3U3FNVnpXdTl1MytjaGNLaE1TCk4rNkpuK1Jna1krcTVXbGpkNTg2QTBaU2toWE9vRlNGYURuVW5Wa1BiUTR3eWJOT3FxRzhtYjRsRldZWFV3RFQKVHVQL1V0Sjk1NTdyNXhoa1Rac3BnUmxRais5YmNQV0hrU0t3RkNYTURtaWxBZkczQVdjSFZad2NleGs0TnpiRQo2TkZRV1daT3VIa2l2M2VnZ3RyNW54MnV1V1FQZW5QcE5uaDVkN2RvUnp2NGhSdmlLTkxxRVpSMHBHbDJpR1cxCnR1aEtCbFdzZndDNUlJaFREU2NYOHgzYnpZMW5YSFYwRGErbnZpb1pJelNKUCtmTWdnTEEyM2tFTEpReEtKNVUKNEh4SG03Z3ozSXpaNFc3ZTNhRGlLUDYxM3pqZjlmMWljMUx6Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0YYYY
```

## kubectlの設定

vimで、以下のファイルを開く。

```
  $vim ~/.kube/config
```

configには、以下のような内容が記載されている。

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: （略）
    server: （略）
  name: myAKSCluster
contexts:
- context:
    cluster: myAKSCluster
    namespace: default
    user: clusterUser_myResourceGroup_myAKSCluster
  name: myAKSCluster
kind: Config
preferences: {}
users:
- name: clusterUser_myResourceGroup_myAKSCluster
  user:
    client-certificate-data: 
    （略）
    client-key-data: 
    （略）
    token: 
    （略）
```

usersセクションに以下の情報を追加する。

```
- name: aks-cluster-front-end-user
  user:
    client-key-data: <コピーしたシークレットのclient-key-data>
    token: <コピーしたシークレットのtoken>
```

内容を保存し、vimを終了する。

今定義したユーザを利用して、kubectlのcontextを作成する。
以下のコマンドで、現在の設定を確認する。

```
  $kubectl config view
```

以下のような情報が得られる。

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: （略）
  name: aks-cluster
contexts:
- context:
    cluster: aks-cluster
    user: clusterUser_aks-rg_aks-cluster
  name: aks-cluster
current-context: aks-cluster
kind: Config
preferences: {}
users:
- name: aks-cluster-front-end-user
  user:
    client-key-data: REDACTED
    token: （略）
- name: clusterUser_aks-rg_aks-cluster
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
    token: （略）
```

clustersから対象のクラスタのnameをコピーする。

以下のコマンドを実行する。（context名はクラスタ名-ユーザ名などで設定する）

```
  $kubectl config set-context <クラスタ名>-front-end --cluster=<クラスタ名> --user=front-end-user --namespace=front-end
```

contextを切り替える。

```
  $kubectl config use-context <クラスタ名>-front-end
```

Podの一覧を取得してみる。

```
  $kubectl get pods
```

このcontextでは、namespace にdefaultではなく、front-endを指定しているため、```kubectl get pods -n front-end```と同じ実行結果となり、podは表示されない。
先の手順でnginxのpodをback-endにデプロイ済みなので、back-endをnamespaceに指定して、podの一覧を取得してみる。

```
  $kubectl get pods -n back-end
```

以下のようなエラーとなり、podの一覧を取得することができない。

```
  Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:front-end:front-end-user" cannot list resource "pods" in API group "" in the namespace "back-end"
```

これにより、それぞれのnamespaceにroleを作成し、rolebindingによってサービスアカウントと紐づけることで、namespaceへのアクセス操作を制御することが確認できた。
