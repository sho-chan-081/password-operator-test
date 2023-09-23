## Password Operator概要

* 出来ること：```kind```が```Password```となっているマニフェストファイルをkube applyすると自動的にパスワードを生成して、Secretにパスワードを格納してくれる。

### 処理の流れ
1. マニフェストファイルをkube apply ***する
1. マニフェストファイルをもとに```Password```オブジェクトが作られる
1. ControllerはReconcile機能で```Password```オブジェクトの作成等の状況をWatch
1. Passwordオブジェクトが作成されたら、対応した```Secret```を作成。Secretのfield欄に自動生成されたPasswordが保存される。

## 課題
* PasswordStatusにReasonフィールドを追加する
* 何故かValidationWebhookのCRDが上手いことk8sに入らないので復習する時に見直す

## MEMO

* api/*でCRDを定義するGoファイルが置かれている。ここのGOファイルを編集することで、CRDのマニフェストファイルを生成することができる。
* ↑ make manifestsでCRDマニフェストファイルを生成。config/crd配下のyamlファイルに変更が加わる。
* make installで接続先クラスターにCRDをデプロイ。(今回はminikube)
※make installでmake manifests & minikubeにデプロイまで一気通貫で行える。make runはコントローラーをminikubeで動かすときに必要。

	```
	minikube image load password-operator:v1
	```

client.IgnoreNotFound(err)でオブジェクト(deploymentとか)が削除されたときにReconsileを止める？＆オブジェクト消えたからもう処理しませんでっていうことをしている。

	```
	// Fetch Password object
		var password secretv1alpha1.Password
		if err := r.Get(ctx, req.NamespacedName, &password); err != nil {
			logger.Error(err, "Fetch Password object - failed")
			return ctrl.Result{}, client.IgnoreNotFound(err)
		}
	```

* kubectl apply -f した時にvalidation error(パスワードが短すぎます的な）するときは、admission webhookを実装する。

  ```
  kubebuilder create webhook --group secret --version v1alpha1 --kind Password --programmatic-validation
  ```

# password-operator-test
// TODO(user): Add simple overview of use/purpose

## Description
// TODO(user): An in-depth paragraph about your project and overview of use

## Getting Started
You’ll need a Kubernetes cluster to run against. You can use [KIND](https://sigs.k8s.io/kind) to get a local cluster for testing, or run against a remote cluster.
**Note:** Your controller will automatically use the current context in your kubeconfig file (i.e. whatever cluster `kubectl cluster-info` shows).

### Running on the cluster
1. Install Instances of Custom Resources:

```sh
kubectl apply -f config/samples/
```

2. Build and push your image to the location specified by `IMG`:

```sh
make docker-build docker-push IMG=<some-registry>/password-operator-test:tag
```

3. Deploy the controller to the cluster with the image specified by `IMG`:

```sh
make deploy IMG=<some-registry>/password-operator-test:tag
```

### Uninstall CRDs
To delete the CRDs from the cluster:

```sh
make uninstall
```

### Undeploy controller
UnDeploy the controller to the cluster:

```sh
make undeploy
```

## Contributing
// TODO(user): Add detailed information on how you would like others to contribute to this project

### How it works
This project aims to follow the Kubernetes [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)

It uses [Controllers](https://kubernetes.io/docs/concepts/architecture/controller/)
which provides a reconcile function responsible for synchronizing resources untile the desired state is reached on the cluster

### Test It Out
1. Install the CRDs into the cluster:

```sh
make install
```

2. Run your controller (this will run in the foreground, so switch to a new terminal if you want to leave it running):

```sh
make run
```

**NOTE:** You can also run this in one step by running: `make install run`

### Modifying the API definitions
If you are editing the API definitions, generate the manifests such as CRs or CRDs using:

```sh
make manifests
```

**NOTE:** Run `make --help` for more information on all potential `make` targets

More information can be found via the [Kubebuilder Documentation](https://book.kubebuilder.io/introduction.html)

## License

Copyright 2023.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
