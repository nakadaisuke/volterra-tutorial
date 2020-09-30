# Virtual Kubernetesの設定

Virtual KubernetesはVolterra独自の概念です。Volterraでは複数のKubernetes Clusterを1つの仮想的なKubernetesとして扱います。
このため、Virtua Kubernetesには複数のVirtual Siteを設定することができ、1つのKubernetes manifestを複数のKubernetes clusterに配布することが可能です。

![vk8s](./pics/vk8s.png)

## ユーザーNamespaceの作成

Virtual KubernetesはユーザーNamepsaceに作成するため、Namespaceを作成します。
Generalの`My Namespaces`から、 `Add namespace`を開き、Namapace名を入れてSaveします。
作成したNamespaceを選択すると、ユーザーNamaspaceに入れます。

![namespace1](./pics/namespace1.png)
![namespace2](./pics/namespace2.png)

## Virtual Siteの作成

作成したNamespaceに移動し、Manage -> Virtual host ->Virtual sitesより `Add Virtual site`を選択します。
nameに virtual-site名、Site TypeはCEを選択し、Site Selector ExpressionではSiteに設定したラベルを選択します。 Continueを選択するとVirtual siteが作成されます。

以下の2つのVirutal siteを設定します。
Name: `pref-tokyo`
Site type: `CE`
Site Selecter Expression: `pref:tokyo`

Name: `pref-osaka`
Site type: `CE`
Site Selecter Expression: `pref:osaka`

![vsite1](./pics/vsite1.png)

## Virtual kubernetesの作成

Applications -> Virtual k8sより`Add Virtual K8s`を選択します。Nameを入力し、Select vsite refから作成した`pref-tokyo`を選択します。 Add Virtual k8sをクリックするとVirtual kubernetesが作成されます。
*作成まで数十秒かかります。

![vk8s1](./pics/vk8s1.png)
![vk8s2](./pics/vk8s2.png)

## deplyomentの作成

作成したVirtual K8sを選択するとKuberneresの作成画面が表示されます。通常のKubernetes Manifestと同様にDeploymentやServiceを作成すると、実際のSiteにワークロードが作成されます。

下のようにDeploymentを設定すると、該当するSiteにコンテナが立ち上がります。

```apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

![vk8s_deplyoment](./pics/vk8s_deployment.png)

## Serviceの作成

`Add service`を選択するとYaml(json)を入力する画面が開きます。
下のようにServiceを設定すると、該当するSiteにServiceが設定されます。

```apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: nginx
```

![vk8s_service](./pics/vk8s_service.png)

## Kubectlからのアクセス

kubectlで使用するkubeconfigはユーザーNamespaceのVirtual K8sからダウンロードします。

Applications -> Virtual k8sの作成済みvk8sの`・・・`から`Kubeconfig`を選択し、Kubecofigの期限を設定します。

![kubeconfig1](./pics/kubeconfig1.png)

`Downlaod Credential`をクリックするとKubeconfigがダウンロードされます。

![kubeconfig2](./pics/kubeconfig2.png)

kubectlを使用し、vk8sにアクセスします。

```kubectl --kubeconfig ~/Downloads/ves_trial_vk8s.yaml get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/1     0            0           9s
```
