# Network policy

Network PolicyはL3-L4のIngress/Egressのセキュリティを提供します。
Remote EndpointからLocal Endpointに入ってくるトラフィックをIngress、Local EndpintからRemote Endpointに出ていくトラフィックをEgressとなります。
例えば以下の場合、Remote Endpointは(8.8.8.8/32, 8.8.4.4/32)となり、Local Endpointは app:webが設定されたPodとなります。

![network_policy1](./pics/network_policy1.png)

以下の場合、Remote Endpointはapp:dbが設定されたPodとなり、Local Endpointは app:webが設定されたPodとなります。

![network_policy2](./pics/network_policy2.png)

## Network policyの構造

コンフィグNetrowk Policy RuleでRemote endpointの条件を作成し、Network PolicyでLocal Endpointに対してNetwork Policy Ruleを適用します。Network Policy SetでNetwork Policy RuleをNamespaceに対して適用します。

![network_policy3](./pics/network_policy3.png)

## Network Policy

### インターネットへの通信制御

namespaceは`seurity`とし、virtual-siteは`vsite-adc`を作成します。
ラベルが異なる2つのPod, app:allow-serverとapp:deny-serverを作成します。
ラベルが異なる2つのPod, app:ce-clientとapp:ce-otherを作成します。

ce-client

```kind: Deployment
apiVersion: apps/v1
metadata:
  name: ce-client
  namespace: security
  annotations:
    ves.io/virtual-sites: security/vsite-adc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ce-client
  template:
    metadata:
      labels:
        app: ce-client
    spec:
      containers:
        - name: ce-client
          image: dnakajima/netutils:1.3
```

```kind: Deployment
apiVersion: apps/v1
metadata:
  name: ce-other
  namespace: security
  annotations:
    ves.io/virtual-sites: security/vsite-adc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ce-other
  template:
    metadata:
      labels:
        app: ce-other
    spec:
      containers:
        - name: ce-other
          image: dnakajima/netutils:1.3
```

Network policyで使用するラベルは、2020/8/24時点でShared namespaceのknown labelsとknown keysに設定されているか、ves.io/app ラベルを使用する必要があります。

![shared_label](./pics/shared_label.png)

作成したPod, app:ce-clientのにGoogle-DNSへのアクセスを拒否します

![network_policy_block1](./pics/network_policy_block1.png)

Network Policy Ruleを2つ作成します。

- allow-any
    Action: Allow

    (** 暗黙のDenyがあるため、設定しないとすべての通信が拒否される）
- deny-google-dns
    Action: Deny

    Remote Endpoint: IP Prefix: Prefix [8.8.8.8/32, 8.8.4.4/32]

![network_policy_block2](./pics/network_policy_block2.png)

Network Policy を2つ作成します。

- ce-client-po
  - Local Endpoint: Label Selector, Selector
  - Expression: app:in(ce-client)
  - Ingress Rules: 1:allow-any
  - Egress Rules:  1: deny-google-dns, 2: allow-any

- ce-other-po
  - Ingress Rules: 1:allow-any
  - Egress Rules: 1:allow-any

![network_policy_block3](./pics/network_policy_block3.png)

Network Policy Setを作成します。

- po-set1
  - Policies: Select policy: [1: ce-client-po, 2: ce-others-po]

![network_policy_block4](./pics/network_policy_block4.png)

フィルターの確認はPodから行えます。Virtual K8sの Pods から対象のPodに Exec to Containerより接続できます。

![network_policy_block5](./pics/network_policy_block5.png)

選択後、Container to exec toから ce-clientやce-otherを選択し、Command to executeにbashを入れるとコンテナにbashで接続できます。

- kubeconfigをダウンロードし、kubectlで接続することも可能です。

![network_policy_block6](./pics/network_policy_block6.png)

ce-clientはgoogle-dnsのポリシーがかかっているため8.8.8.8にはpingできませんが、ce-otherはpingできることが確認できます。

![network_policy_block7](./pics/network_policy_block7.png)
![network_policy_block8](./pics/network_policy_block8.png)

### 同一Kubernetes Clouster内での通信制御

namespaceは`seurity`とし、virtual-siteは`vsite-adc`を作成します。
ラベルが異なる2つのPod, app:allow-serverとapp:deny-serverを作成します。
app:ce-client からのみapp:server-appへの通信を許可し、 app:ce-otherは拒否します

![network_policy_same_node1](./pics/network_policy_same_node1.png)

app:webのPodとServiceを作成します。

```kind: Deployment
apiVersion: apps/v1
metadata:
  name: server-app
  namespace: security
  annotations:
    ves.io/virtual-sites: security/vsite-adc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: server-app
  template:
    metadata:
      labels:
        app: server-app
    spec:
      containers:
        - name: server-app
          image: dnakajima/inbound-app:1.0
          ports:
            - containerPort: 8080
              protocol: TCP
```

```kind: Service
apiVersion: v1
metadata:
  name: web
  namespace: security
  labels:
    app: server-app
  annotations:
    ves.io/virtual-sites: security/vsite-adc
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  selector:
    app: server-app
  type: ClusterIP
```

Network Policy Ruleを1つ作成します。

- deny-server-app

  - Action: Deny
  - Remote Endpoint: Prefix Selector
  - Selection Expression: app:in(server-app)

![network_policy_same_node2](./pics/network_policy_same_node2.png)

Network Policy を1つ作成します。

- remote-app-ce-other
  - Local Endpoint: Label Selector
  - Selector Expression: app:in(ce-other)
  - Ingress Rules: 1:allow-any
  - Egress Rules:  1: deny-server-dns, 2: allow-any

![network_policy_same_node3](./pics/network_policy_same_node3.png)

Network Policy SetにNetwork Policyを追加します

- po-set1
  - Policies: Select policy: [1: ce-client-po
  - 2:remote-app-ce-other, 3: ce-others-po]

![network_policy_same_node4](./pics/network_policy_same_node4.png)

ce-otherはremote-app-ce-otherのポリシーがかかっているためserver-appにはcurlできませんが、ce-clientはcurlできることが確認できます。

![network_policy_same_node5](./pics/network_policy_same_node5.png)
![network_policy_same_node6](./pics/network_policy_same_node6.png)
