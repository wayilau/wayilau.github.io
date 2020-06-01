[TOC]
以istio安装文件中的BookInfo为例来学习istio.
# BookInfo安装
- 手动注入Envoy方式
````
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml)
````
在这里， istioctl是修改bookinfo.yaml文件， 将Envoy container部分注入进去。
- 如果是自动注入(安装过istio-sidecar-injector)， 将namespace标签化：

````
$ kubectl label namespace default istio-injection=enabled
$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
````
这样应用就就部署完了。

确认应用
````
$ kubectl get services
NAME                       CLUSTER-IP   EXTERNAL-IP   PORT(S)              AGE
details                    10.0.0.31    <none>        9080/TCP             6m
kubernetes                 10.0.0.1     <none>        443/TCP              7d
productpage                10.0.0.120   <none>        9080/TCP             6m
ratings                    10.0.0.15    <none>        9080/TCP             6m
reviews                    10.0.0.170   <none>        9080/TCP             6m
````

````
$ kubectl get pods
NAME                                        READY     STATUS    RESTARTS   AGE
details-v1-1520924117-48z17                 2/2       Running   0          6m
productpage-v1-560495357-jk1lz              2/2       Running   0          6m
ratings-v1-734492171-rnr5l                  2/2       Running   0          6m
reviews-v1-874083890-f0qf0                  2/2       Running   0          6m
reviews-v2-1343845940-b34q5                 2/2       Running   0          6m
reviews-v3-1813607990-8ch52                 2/2       Running   0          6m
````

配置ingress的ip与port
````
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
````
确认gateway安装成功：
````
$ kubectl get gateway
NAME               AGE
bookinfo-gateway   32s
````

