# OpenShiftハンズオン用マニフェストファイル

OpenShiftハンズオン用マニフェストファイルを管理します。

## Directory構成

| 名前     | 説明 |
| ----     | ---- |
| build    | Tektonのマニフェストファイル |
| delivery | ArgoCDのマニフェストファイル |
| monitor | 監視サービスのマニフェストファイル |
| services | 住所コード検索サービスのマニフェストファイル |

## CI/CD Pipeline 構築方法

### handson-demo namespaceを作成する
```shell
# openshift
$ oc new-project handson-demo
# kubernetes
$kubectl create ns handson-demo
```

### pvcを作成する
```shell
# create pvc
$ oc create -f build/storage/pvc.yaml -n handson-demo
persistentvolumeclaim/maven-repo-pvc created
persistentvolumeclaim/maven-repo-pvc2 created
```

### tasksを登録する
```shell
# register tekton tasks
$ oc create -f build/task -n handson-demo
task.tekton.dev/get-githash created
task.tekton.dev/git-clone created
task.tekton.dev/kaniko created
task.tekton.dev/maven created
```

### secret情報を修正して登録する

`secret.yaml` の<GitHubID>と<GitHubのUserToken>をそれぞれ置き換える

登録する
```yaml
$ oc create -f build/auth/secret.yaml -n handson-demo
secret/github-sec created
serviceaccount/build-bot created
```

### pipelineを登録する
```
oc create -f build/maven-pipeline.yaml -n handson-demo
pipeline.tekton.dev/openshift-handson-apps-build created
```

```shell
$ tkn pipelines list -n handson-demo
NAME                           AGE              LAST RUN                                 STARTED         DURATION   STATUS
openshift-handson-apps-build   41 seconds ago   openshift-handson-apps-build-run-pz2ql   4 seconds ago   ---        Running
```

```shell
tkn pipelines list -n handson-demo
tkn pipelines logs -f -n handson-demo
```

## 手動でaddresscode servicesをKubernetes環境にデプロイする方法

### 前提

* [kustomize](https://kubernetes-sigs.github.io/kustomize/installation/binaries/)をインストールすること
* `kubectl/oc` コマンドをインストールして、Kubernetes環境に接続できること

### 手順

```sh
# プロジェクトへの移動
$ git clone https://github.com/nelco-abm/openshift-handson-manifests.git
$ cd openshift-handson-manifests
# kustomizeのコンテナイメージをeditする
$ cd services/overlays/production
$ kustomize edit set image quay.io/soharaki/addresscode-app:new-tag
$ kustomize edit set image quay.io/soharaki/addresscode-database:new-tag
# ルートプロジェクトに戻ってマニフェストファイルを適用する
$ cd ../../..
$ kustomize build services/overlays/production/ | oc apply -f -
```

## TIPS

### pvcによって自動的に割り当てられたpvがDeletePolicyだった場合にRetainに置き換える

mavenのビルドを繰り返す際に、pv環境はそのままにした方がキャッシュが効きます。

```shell
# get pv name
kubectl get pv -o=jsonpath='{.items[0].metadata.name}'
# Override pv reclaim policy
kubectl patch pv <pv-name> -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
```

### TektonのダッシュボードでCI結果を見たい場合

[Tekton Dashbord](https://github.com/tektoncd/dashboard/blob/master/docs/install.md#installing-tekton-dashboard-on-kubernetes)に書いてある手順に基本的に従う。

1. k8s, openshiftに接続できるコンソールを開く

```shell
# deplloy the tekton dashbord
kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/tekton-dashboard-release.yaml
```

2. kubectl proxyコマンドでk8s内のserviceに接続する
```shell
kubectl proxy 
```

3. ブラウザで下記の通り入力してTektonダッシュボードを開く
http://localhost:8001/api/v1/namespaces/tekton-pipelines/services/tekton-dashboard:http/proxy/

### Tektonのビルドを手動で確認したい

現在、TektonのビルドはGitHubのWebhookによってのみ動作する。
しかし、手動で動かしたいユースケースもありえると思うのでいくつか方法を記載する。

1. GitHubWebhookの代わりにcurlで直接Eventを送信する

```sh
curl -v \
-H 'X-GitHub-Event: push' \
-H 'Content-Type: application/json' \
-d '{"action": "ref", "ref": "refs/heads/master"}' \
el-github-listener-interceptor-handson-demo.apps.sor-cluster.ndap-nelco.net 
```

2. `tkn`コマンドでpipelinerunを直接作成する

```sh
oc create -f build/maven-pipelinerun.yaml
```