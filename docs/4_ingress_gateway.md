# Ingress Gatewayの設定

作成したKubernetes Serviceはそのままでは外部からアクセスできないため、作成したVoltStack上のアプリケーションに外部から接続できるようにIngress Gatewayを設定します。Internet上のVoltMeshを利用しても良いですし、ローカルのVolterra Nodeの利用も可能です。

Ingress GatewayはHTTP/TCP loadbalancerとして動作します。Loadlabancerの宛先はOrigin Poolとして、定義します。

![ingress_gw1](./pics/ingress_gw1.png)

## Origin poolの作成

作成したNginxを外部からアクセスできるようにIngress Gatewayを設定します。作成したNginx ServiceをOrigin poolとして登録します。 Manage -> Origin Pools で `Add Origin Pool`を選択します。

以下の設定をします

- Name: `nginx-endpoint`
- Select Type of Origin Server: `k8sService Name of Origin Ser...`
- Service Name: `nginx.namespace`を入力します。 (`kubernetes service名.namespace`のフォーマット）
- Select Site or Virtual Site: `Virtual Site` -> `namespace/pref-tokyo`
- Select Network on the Site: `Vk8s Networks on Site`
- Port: `80`

## インターネットからの接続

### HTTP load balancerの設定

Manage -> HTTP Load Balancers で “Add HTTP load balancer”を選択します。

- Name: `nginx-lb`
- Domains: `dummy.localhost` (設定するとDNS infoにVolterraからdomain名が払い出されます。設定後に払い出されたドメイン名を設定してください。)
- Select Type of Load Balancer: `HTTP`
- Default Route Origin Pools: `namespace/nginx-endpoint` (上記で作成したOrigin pool)

設定するとDNS infoにVolterraからdomain名が払い出されます。作成したロードヴァランダーのDomainsに設定するか、任意のDNSサーバのCNAMEレコードに設定してください。
外部から設定したドメインにアクセスするとNginxのWebUIが表示されます。

![ingress_config](./pics/ingress_config.png)

ブラウザにドメインを入力すると表示されます。

![nginx_ss](./pics/nginx_ss.png)
>DNSの伝搬やコンフィグの反映に1-2分かかる場合があります。

## Local Interfaceからのアクセス

### HTTP Loadbalancerの設定

Manage -> HTTP Load Balancers で “Add HTTP load balancer”を選択します。

- Name: `nginx-lb`
- Domains: `nginx.localhost`
- Select Type of Load Balancer: `HTTP`
- Default Route Origin Pools: `namespace/nginx-endpoint` (上記で作成したOrigin pool)
- VIP Configuration: `Show Advanced Fields`を有効にし、`Advertise Custom`を指定
- Configureを選択
- Select Where to Advertise: `virtual-site` -> namespace/
- Site Network: `Inside and Outside Network`
- Virtual Site Reference: `namespace/pref-tokyo`

> virtual-siteで`pref-osaka`を作成し、Virtual Site Referenceに`pref-osaka`を設定すると、pref-osakaのVolterra Nodeにアクセスし、pref-tokyoのアプリにアクセスできます。

ローカルDNSがない場合は/etc/hostsに設定したドメイン名とエッジノードのIPアドレスを入力するか、Curlで -H “Host: domain name”で確認します。

```curl http://192.168.2.197 -H "Host: localhost.com"
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

## トラブルシューティング

Nginxにアクセスできない場合、同一Virtual-siteにUbuntuなどのコンテナを立ち上げ、Service経由でNginxにアクセスできるか確認してください。

Service経由でアクセスできる場合、Origin Poolが正常に稼働しているか確認してください。
Origin poolの`Show Child Object`内のGlobal Satusが空欄の場合は、設定のService nameが間違っていたり、Virtual-siteが異なるサイトを指定している可能性があります。

![trouble_originpool](./pics/trouble_originpool.png)
