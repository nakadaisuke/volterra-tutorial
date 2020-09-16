# Volterra チュートリアル

本ドキュメントはVolterra (<https://www.volterra.io/>) の機能を使うにあたり、Volterraの基礎知識とVolterraのVoltMesh/VoltStackを使用するにあたり、基本的なコンフィグの習得を目的としています。

* 本ドキュメントはVolterraアンオフィシャルです。

## 簡単な説明

VolterraはKubernetes基盤(VoltStack)やApplication Securityを提供するゲートウェイ(VlotMesh)を提供します。
Freeのアカウントは <https://console.ves.volterra.io/signup/usage_plan> より取得できます。

## VoltMesh機能

* L3-L4セキュリティ
* L7セキュリティ
* APIゲートウェイ
* LB

他の機能はこちらを参照して下さい。<https://www.volterra.io/products/voltmesh>

## VoltStack機能

* マネージドKuberners
* クラスタリング
* Fleetマネジメント
* KMS

他の機能はこちらを参照して下さい。<https://www.volterra.io/products/voltstack>

## チュートリアル

1. [Volterraの基礎知識](./docs/1_volterra-tutorial.md)
2. [Volterra Nodeのインストールと設定](./docs/2_volterra-install.md)
3. [Virtual Kubernetesの設定](./docs/3_virtual_kubernetes.md)
4. [Ingress Gatewayの設定](./docs/4_ingress_gateway.md)
5. [複数Virtual siteの使い方](./docs/5_multiple_vsite.md)
6. [App to App 接続の設定](./docs/6_app_app.md)
7. [Application Delivery Controller](./docs/7_app_delivery_controller.md)
8. [Network policy](./docs/8_network_policy.md)
9. [Service policy (Ingress Gateway)](./docs/9_service_policy.md)
10. [WAF](./docs/10_waf.md)
