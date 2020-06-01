[TOC]
上一篇中做了istio的相关介绍， 这一篇开始正式安装与使用istio.
# 安装istio
当前的安装过程都是在Kubernetes下进行， 使用helm安装， 其它平台，后续补充。

## 下载
[下载istio](https://istio.io/docs/setup/kubernetes/download-release/)
或者直接在命令行中执行如下命令
````
curl -L https://git.io/getLatestIstio | sh -
````
这样istio就下载到当前目录下。目前最新版本是1.0.2。
````
cd istio-1.0.2
````
目录包括：
- install/ 里边包括kubernetes中需要的.yaml安装文件
- samples/ demo的yaml
- istioctl bin目录下， istio的命令行工具
- istio.VERSION 配置文件

安装过程
1. 安装CRD资源
````
 kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
````
2. 安装istio的核心组件， 有四种选择， 四种选择互斥， 选一种即可。

Option 1: Install Istio without mutual TLS authentication between sidecars
Visit our mutual TLS authentication between sidecars concept page for more information.

Choose this option for:

Clusters with existing applications,
Applications where services with an Istio sidecar need to be able to communicate with other non-Istio Kubernetes services,
Applications that use liveness and readiness probes,
Headless services, or
StatefulSets
To install Istio without mutual TLS authentication between sidecars:
````
$ kubectl apply -f install/kubernetes/istio-demo.yaml
````
Option 2: Install Istio with default mutual TLS authentication
Use this option only on a fresh Kubernetes cluster where newly deployed workloads are guaranteed to have Istio sidecars installed.

To Install Istio and enforce mutual TLS authentication between sidecars by default:
````
$ kubectl apply -f install/kubernetes/istio-demo-auth.yaml
````
Option 3: Render Kubernetes manifest with Helm and deploy with kubectl
Follow our setup instructions to render the Kubernetes manifest with Helm and deploy with kubectl.

Option 4: Use Helm and Tiller to manage the Istio deployment
Follow our instructions on how to use Helm and Tiller to manage the Istio deployment.

我们选择第三种来安装：[官方安装地址](https://istio.io/docs/setup/kubernetes/helm-install/#option-1-install-with-helm-via-helm-template)
### 安装方式1
````
render istio's core components to a kubernetes manifest called istio.yaml.
helm template install/kubernetes/helm/istio --name istio --namespace istio-system > $HOME/istio.yaml

$ kubectl create namespace istio-system
````

### 安装方式2
````
kubectl apply -f install/kubernetes/helm/helm-service-account.yaml
helm install install/kubernetes/helm/istio --name istio --namespace istio-system
````

这样istio就安装成功了。
我们来看一下组件：
````
$ kubectl get svc -n istio-system
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)                                                               AGE
istio-citadel              ClusterIP      10.47.247.12    <none>            8060/TCP,9093/TCP                                                     7m
istio-egressgateway        ClusterIP      10.47.243.117   <none>            80/TCP,443/TCP                                                        7m
istio-galley               ClusterIP      10.47.254.90    <none>            443/TCP                                                               7m
istio-ingress              LoadBalancer   10.47.244.111   35.194.55.10      80:32000/TCP,443:30814/TCP                                            7m
istio-ingressgateway       LoadBalancer   10.47.241.20    130.211.167.230   80:31380/TCP,443:31390/TCP,31400:31400/TCP                            7m
istio-pilot                ClusterIP      10.47.250.56    <none>            15003/TCP,15005/TCP,15007/TCP,15010/TCP,15011/TCP,8080/TCP,9093/TCP   7m
istio-policy               ClusterIP      10.47.245.228   <none>            9091/TCP,15004/TCP,9093/TCP                                           7m
istio-sidecar-injector     ClusterIP      10.47.245.22    <none>            443/TCP                                                               7m
istio-statsd-prom-bridge   ClusterIP      10.47.252.184   <none>            9102/TCP,9125/UDP                                                     7m
istio-telemetry            ClusterIP      10.47.250.107   <none>            9091/TCP,15004/TCP,9093/TCP,42422/TCP                                 7m
prometheus                 ClusterIP      10.47.253.148   <none>            9090/TCP                                                              7m
````


## 部署应用
现在istio安装完成以后， 你可以部署一个demo应用， 在samples文件夹下。我们现在部署一个BookInfo

- 如果你已经运行了istio-sidecar-injector， 你可以直接使用"kubectl apply"部署应用, istio-sidecar-injector会自动的注入Envoy containers到你的应用pod里去。injector会假设这些pod运行在标签化的域中， 标签是:
````
istio-injectio=enabled
$ kubectl label namespace <namespace> istio-injection=enabled
kubectl create -n <namespace> -f <your-app-spec>.yaml
````

- 如果你没有安装istio-sidecar-injector， 你必须在应用deploy之前手动的将Envoy container注入到应用的pod。
````
$ istioctl kube-inject -f <your-app-spec>.yaml | kubectl apply -f -
````

## 卸载istio
我们是以helm方式来部署的，所以同样有它的卸载方式。
上边有好几种option, 对每种otpion执行卸载的方式不相同。
option1:
````
$ kubectl delete -f $HOME/istio.yaml
````
option2:
````
$ helm delete --purge istio
````
如果你的helm version是2.9.0之前， 在部署新版istion之前还需要以下操作：
````
$ kubectl -n istio-system delete job --all
````

清除CRDS
````
kubectl delete -f install/kubernetes/helm/istio/templates/crds.yaml -n istio-system
````

下一篇学习对应用管理。
