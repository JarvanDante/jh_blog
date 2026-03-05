---
author: "Karson"
title: "EFK日志监控平台部署架构"
date: 2022-06-21
description: "ElasticSearch、Fluentd、Kibana组成的日志监控平台"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Karson
authorEmoji: 👻
tags:
- docker 
categories:
- k8s
- fluentd
- es
- kibana
---


## 组件介绍
+ ElasticSearch：日志存储 + 索引 + 搜索
+ Fluentd：日志采集、装饰、转换和传输
+ Kibana：日志查询展示
+ 生产环境 + Kafka

## minikube环境搭建EFK
+ Fluentd DaemonSet
+ 也可以采用minikube efk addon ,更简单

## 配置信息如下：
### 新建目录：
`/Users/wangdante/D/myk8s/efk`
```shell
tree efk
efk
├── elastic.yml
├── fluentd-daemonset.yml
├── fluentd-rbac.yml
├── kibana.yml
└── ns.yml
```

### ns.yml:
```yml
apiVersion: v1
kind: Namespace
metadata:
  name: logging
```

### elastic.yml:
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: elasticsearch
  namespace: logging
spec:
  selector:
    matchLabels:
      component: elasticsearch
  template:
    metadata:
      labels:
        component: elasticsearch
    spec:
      containers:
        - name: elasticsearch
          image: docker.elastic.co/elasticsearch/elasticsearch:6.5.4
          env:
            - name: discovery.type
              value: single-node
          ports:
            - containerPort: 9200
              name: http
              protocol: TCP
          resources:
            limits:
              cpu: 500m
              memory: 2Gi
            requests:
              cpu: 500m
              memory: 2Gi
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
  namespace: logging
  labels:
    service: elasticsearch
spec:
  type: NodePort
  selector:
    component: elasticsearch
  ports:
    - port: 9200
      targetPort: 9200
      nodePort: 31200
```

### kibana.yml:
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  namespace: logging
spec:
  selector:
    matchLabels:
      run: kibana
  template:
    metadata:
      labels:
        run: kibana
    spec:
      containers:
        - name: kibana
          image: docker.elastic.co/kibana/kibana:6.5.4
          env:
            - name: ELASTICSEARCH_URL
              value: http://elasticsearch:9200
            - name: XPACK_SECURITY_ENABLED
              value: "true"
          ports:
            - containerPort: 5601
              name: http
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: kibana
  namespace: logging
  labels:
    service: kibana
spec:
  type: NodePort
  selector:
    run: kibana
  ports:
    - port: 5601
      targetPort: 5601
      nodePort: 31601
```

### fluentd-rbac.yml:
```yml
apiVersion: apps/v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: fluentd
  namespace: kube-system
rules:
  - apiGroups:
      - ""
      resources:
        - pods
        - namespaces
      verbs:
        - get
        - list
        - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: ServiceAccount
    name: fluentd
    namespace: kube-system
```

### fluentd-daemonset.yml:
```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
    version: v1
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-logging
      version: v1
  template:
    metadata:
      labels:
      k8s-app: fluentd-logging
      version: v1
      kubernetes.io/cluster-service: "true"
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
      containers:
        - name: fluentd
          images: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
          env:
            - name: FLUENT_ELASTICSEARCH_HOST
              value: "elasticsearch.logging"
            - name: FLUENT_ELASTICSEARCH_PORT
              value: "9200"
            - name: FLUENT_ELASTICSEARCH_SCHEME
              value: "http"
            - name: FLUENT_UID
              value: "0"
            - name: FLUENTD_SYSTEMD_CONF
              value: disable
          resource:
            limit:
              memory: 200Mi
            requests:
              cpu: 100m
              memgory: 200Mi
          volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
        - name: varlibdockercontainers
          hostPath:
            path: /var/lib/docker/containers



---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: fluentd
  namespace: kube-system
rules:
  - apiGroups:
      - ""
      resources:
        - pods
        - namespaces
      verbs:
        - get
        - list
        - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: ServiceAccount
    name: fluentd
    namespace: kube-system
```

### 发布
```shell
$ kubectl apply -f ns.yml
$ kubectl get ns
NAME                   STATUS   AGE
default                Active   111m
kube-node-lease        Active   111m
kube-public            Active   111m
kube-system            Active   111m
kubernetes-dashboard   Active   102m
logging                Active   20s

$ kubectl apply -f elastic.yml
deployment.apps/elasticsearch created
service/elasticsearch created

$ kubectl get all -n logging
NAME                                 READY   STATUS    RESTARTS   AGE
pod/elasticsearch-659897658c-smhb7   1/1     Running   0          44s

NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/elasticsearch   NodePort   10.102.0.244   <none>        9200:31200/TCP   45s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/elasticsearch   1/1     1            1           45s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/elasticsearch-659897658c   1         1         1       45s
```


```shell
$ minikube service list
|----------------------|---------------------------|--------------|-----|
|      NAMESPACE       |           NAME            | TARGET PORT  | URL |
|----------------------|---------------------------|--------------|-----|
| default              | kubernetes                | No node port |     |
| kube-system          | kube-dns                  | No node port |     |
| kubernetes-dashboard | dashboard-metrics-scraper | No node port |     |
| kubernetes-dashboard | kubernetes-dashboard      | No node port |     |
| logging              | elasticsearch             |         9200 |     |
|----------------------|---------------------------|--------------|-----|
```

```shell
$ kubectl apply -f kibana.yml
deployment.apps/kibana created
service/kibana created

$ kubectl get all -n logging
NAME                                 READY   STATUS              RESTARTS   AGE
pod/elasticsearch-659897658c-smhb7   1/1     Running             0          5m36s
pod/kibana-78fdff978c-kkpgv          0/1     ContainerCreating   0          9s

NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/elasticsearch   NodePort   10.102.0.244    <none>        9200:31200/TCP   5m37s
service/kibana          NodePort   10.105.62.205   <none>        5601:31601/TCP   9s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/elasticsearch   1/1     1            1           5m37s
deployment.apps/kibana          0/1     1            0           9s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/elasticsearch-659897658c   1         1         1       5m37s
replicaset.apps/kibana-78fdff978c          1         1         0       9s
```



>PS:端口转发：
> `kubectl port-forward --address 0.0.0.0 -n logging kibana-78fdff978c-kkpgv 31602:5601`