NetworkPolcy
==================================

---------------------------------
概要
---------------------------------

    - NetworkPolicyは、Kubernetesクラスター内のPod間の通信を制御するためのルール
    - デフォルトではKubernetesのPodはすべてのPodに通信可能ですが、NetworkPolicyを使うことで特定の通信だけを許可できる

---------------------------------
定義
---------------------------------

例：「curlのPodから外への通信は、ポート80のTCPのみを許可する」NetworkPolicyのマニフェスト

.. code-block:: yaml

    apiVersion: "networking.k8s.io/v1"
    kind: NetworkPolicy
    metadata:
        name: app-allow-external-80
        namespace: default
    spec:
        policyTypes:     # Egressのルールのみを適用（Ingressは対象外）
        - Egress
        podSelector:     # ルールの適用対象がcurlのPodであることを表す
            matchLabels:
                app: app-monitor
        egress:          
        - to:
            podSelector: 
                matchLabels:
                    app: istio
            namespaceSelector: 
                matchLabels:
                    app: istio-system
          ports:           # 80番ポートでTCPのみを許可
            - port: 80
            protocol: TCP


上記ソースコードについて部分ごとに説明します。

.. code-block:: yaml

    apiVersion: "networking.k8s.io/v1"

NetworkPolicy を定義するための Kubernetes API のバージョンです。  
NetworkPolicy は networking.k8s.io/v1 を利用

.. code-block:: yaml

    kind: NetworkPolicy

このマニフェストが NetworkPolicy リソース であることを示す。
NetworkPolicy は、Pod の通信を制御するための Kubernetes リソースです。

.. code-block:: yaml

    metadata:
        name: app-allow-external-80
        namespace: default

このリソースのメタ情報を定義するブロックです。
nameは、NetworkPolicy の名前です。
NameSpaceは、NetworkPolicy を作成する名前空間の名前です。

.. code-block:: yaml

    spec:
        policyTypes:     # Egressのルールのみを適用（Ingressは対象外）
        - Egress
    
policyTypesは、NetworkPolicy で制御する通信方向を指定する。
通信方向には、主にIngressとEgressがある。
今回だと、Egress（外向き通信）、つまりPodから外部への通信を制御する

.. code-block:: yaml

    podSelector:     # ルールの適用対象がcurlのPodであることを表す
        matchLabels:
            app: app-monitor
        
NetworkPolicy を適用する Pod を選択する設定する。
matchLabelsは、Pod のラベルを使って対象を指定する。
今回だと、app=app-monitorというラベルを持つPodを対象とする。

.. code-block:: yaml

    egress:          
        - to:
            podSelector:     
                matchLabels:
                    app: istio
            namespaceSelector: 
                matchLabels:
                    app: istio-system
          ports:
            - port: 80
            protocol: TCP

egressは、Egress(外向き通信)の許可ルールを定義する部分です。
NetworkPolicy は基本的に、許可リスト方式です。

Namespace 名で指定したい場合でも、NetworkPolicy では通常 namespaceSelector を使うため、Namespace にラベルを付けて指定する。
PodSelectorは、通信先 Pod の条件を指定する。
portsは、許可する通信プロトコルとポートを指定する。
今回だと、以下のような意味になる。

app=istio-system ラベルを持つ Namespace 内にある、app=istio ラベルを持つ PodへのTCP 80番ポート宛ての通信通信を許可する

※ portsに"-"がない理由は、toとportsをAND条件とするため。