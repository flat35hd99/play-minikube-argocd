# minikube + ArgoCDお試し

Windows 10で試した。

## minikubeインストール

公式の[startガイド](https://minikube.sigs.k8s.io/docs/start/)に沿ってインストール

```console
winget install minikube
minikube start
kubectl get po -A # Check installaion successfully
minikube dashboard
```

これでOK

## ArgoCD

### Install

公式の[getting started](https://argo-cd.readthedocs.io/en/stable/getting_started)

```console
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

CLIツールはWindowsの場合、シングルバイナリーが[GitHubのRelease](https://github.com/argoproj/argo-cd/releases/latest)からダウンロードできる。

以降、`argocd`はダウンロードしたバイナリーファイルへのパスに読み替えてね。

### Access ArgoCD server

ArgoCDのサービスへのアクセス方法が３つある

- LoadBalancerを通す方法
- Ingressを使う方法
- Port forwardingする方法

port forwarding は一時的 + サービスを曝さないということなので、選択。

```console
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Login

ログインしたいので、ユーザー名とパスワードが知りたい。ユーザ名は`admin`, パスワードは

```console
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
```

の結果をbase64でデコードしたもの。linuxの場合は`| base64 -d; echo`を最後につければOK.

ログインは

```console
argocd login localhost:8080
argocd account update-password
```

とする。`8080`は、ポートフォワーディングで指定したポート。

### Register a cluster

argocdでデプロイするクラスターを選択する。

まずクラスターの名前一覧を確認する。

```console
kubectl config get-contexts -o name
```

これを実行すると、minikubeを使っているだけの場合は`minikube`という文字列が出てくるはずなので、

```console
argocd cluster add minikube
```

とすればOK

### Create Apps

ブラウザで`https://localhost:8080`にアクセスすれば、いろいろできるようになってる。すごい。

## 気になったこと

データの保存先を指定していない。どこにデータがたまってるんだろう。プロダクション環境では絶対に指定したいはず。

ArgoCDのバックアップ？みたいなのほしくない？パッと見てみると、`argocd export`, `argocd import`というコマンドがある。[Disaster Recovery](https://argo-cd.readthedocs.io/en/stable/operator-manual/disaster_recovery/)参照。

クラスターのmetrixもほしいし、各podのmetrixもほしい、ほしくない？
