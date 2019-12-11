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
