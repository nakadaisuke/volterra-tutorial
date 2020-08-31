# Ingress Gatewayの設定

作成したKubernetes Serviceはそのままでは外部からアクセスできないため、作成したVoltStack上のアプリケーションに外部から接続できるようにIngress Gatewayを設定します。Internet上のVoltMeshを利用しても良いですし、ローカルのVolterra Nodeの利用も可能です。

Ingress GatewayはHTTP/TCP loadbalancerとして動作します。Loadlabancerの宛先はOrigin Poolとして、定義します。

![ingress_gw1](./pics/ingress_gw1.png)

## Origin poolの作成

作成したNginxを外部からアクセスできるようにIngress Gatewayを設定します。作成したNginx ServiceをOrigin poolとして登録します。 Manage -> Origin Pools で `Add Origin Pool`を選択します。

- Name: は任意のPool名を入力します。(nginx-endpointなど)
- Basic Configuration: ”Select Type of Origin Server”は`k8sService Name of Origin Ser...`を選択します。
- Service Name: `Kubernetes service名.namespace`を入力します。 (nginx.trial など）
- Select Site or Virtual Site: `Virtual Site`を選択し、 Virtual Siteは作成したVirtual Siteを指定します。
- Select Network on the Site: `Vk8s Networks on Site`を指定します。
- Port: `80`を設定します。
