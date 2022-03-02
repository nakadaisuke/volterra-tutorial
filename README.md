# F5 Distributed Cloud Services チュートリアル

本ドキュメントはF5 Distributed Cloud Services (DCS) (<https://www.f5.com/cloud>) の機能を使うにあたり、DCSの基礎知識とDCSのMesh/AppStackを使用するにあたり、基本的なコンフィグの習得を目的としています。

* 本ドキュメントはF5 Distributed Cloud Serviceアンオフィシャルです。

## 簡単な説明

DCSはKubernetes基盤(AppStack)やApplication Securityを提供するゲートウェイ(Mesh)を提供します。
Freeのアカウントは <https://console.ves.volterra.io/signup/usage_plan> より取得できます。

## Mesh機能

MeshはApp to App通信の制御や暗号化、高度なIngress/Egressゲートウェイ機能、ロードバランスなどを提供します。

* L3-L4セキュリティ
* L7セキュリティ
* APIゲートウェイ
* LB

他の機能はこちらを参照して下さい。<https://www.f5.com/cloud/products/platform-overview>

## AppStack機能

AppStackはマネージドKubernetesとして、ノード管理、アプリケーション配信、エンタープライズレベルのシークレットなどの機能を提供します。

* マネージドKuberners
* クラスタリング
* Fleetマネジメント
* KMS

他の機能はこちらを参照して下さい。<https://www.f5.com/cloud/products/platform-overview>

## チュートリアル

1. [DCSの基礎知識](./docs/1_dcs-tutorial.md)
2. [DCS Nodeのインストールと設定](./docs/2_dcs-install.md)
3. [Virtual Kubernetesの設定](./docs/3_virtual_kubernetes.md)
4. [Ingress Gatewayの設定](./docs/4_ingress_gateway.md)
5. [複数Virtual siteの使い方](./docs/5_multiple_vsite.md)
6. [App to App 接続の設定](./docs/6_app_app.md)
7. [Application Delivery Controller](./docs/7_app_delivery_controller.md)
8. [Network policy](./docs/8_network_policy.md)
9. [Service policy (Ingress Gateway)](./docs/9_service_policy.md)
