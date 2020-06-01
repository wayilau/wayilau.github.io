[TOC]
# istio功能
## 流量管理
[官方原文链接](https://istio.io/docs/concepts/traffic-management/)
本文不是对原文的翻译， 是对原文的理解。
此页面概述了交通管理在Istio中的运作方式， 包括流量管理原则的好处。

使用Istio的流量管理模型实质上解耦了流量和基础设施扩展，让你通过Pilot指定他们希望流量遵循的规则，而不是哪些特定的pod / VM， Pilot和Envoy会照顾其余的。
举例你希望将5%的流量导到蓝绿部署的应用中。或者根据请求的内容将流量导致特定版本的的服务。

下图所示：
![image](https://istio.io/docs/concepts/traffic-management/TrafficManagementOverview.svg)

将流量与基础架构扩展分离，使Istio能够提供各种流量管理功能，这些功能位于应用程序代码之外。 除了用于A / B测试，逐步推出和金丝雀版本的动态请求路由之外，它还使用超时，重试，断路器和故障注入来处理故障恢复，以测试跨服务的故障恢复策略的兼容性。 这些功能都是通过服务网格上部署的Envoy sidecars / proxies实现的。

### Pilot and Envoy
istio中的流量管理的核心组件是Pilot, 它管理与配置所有运行在pod中的Envoy proxy。Pilot允许您指定要用于在Envoy代理之间路由流量的规则，以及配置故障恢复功能（如超时，重试和断路器）。它还维护网格中所有服务的规范模型，并使用此模型让Envoy实例通过其发现服务了解网格中的其他Envoy实例。

![image](https://istio.io/docs/concepts/traffic-management/PilotAdapters.svg)

如上图所示，Pilot维护网格中服务的规范表示，该表示独立于底层平台。 Pilot中特定于平台的适配器负责适当地填充此规范模型。 例如，Pilot中的Kubernetes适配器实现必要的控制器，以观察Kubernetes API服务器是否更改了pod注册信息，入口资源和存储流量管理规则的第三方资源。 该数据被翻译成规范表示。 然后基于规范表示生成特定于特定的配置。

### Request routing
Istio引入了服务版本的概念，这是一种通过版本（v1，v2）或环境（staging，prod）细分服务实例的更细粒度的方法。这些变体不一定是不同的API版本：它们可以是对同一服务的迭代更改，部署在不同的环境（prod，staging，dev等）中。使用它的常见方案包括A / B测试或金丝雀部署,Istio的流量路由规则可以引用服务版本，以提供对服务之间流量的额外控制。

服务之间的通信：
![image](https://istio.io/docs/concepts/traffic-management/ServiceModel_Versions.svg)

客户端并不知道服务的版本， 依然是通过hostname/ip来进行访问， Envoy proxy拦截请求并将请求发给service。由Envoy 通过routing rules来决定真正要调用的服务版本。这样解耦应用与其依赖的服务代码。istio对同一版本的服务多个实例， 提供了负载均衡， 可以看[负载均衡原文](https://istio.io/docs/concepts/traffic-management/#discovery-and-load-balancing)。

### Ingress and engress
![image](https://istio.io/docs/concepts/traffic-management/ServiceModel_RequestFlow.svg)

通过在每一个services的pod里加上sidecar， 这样Envoy proxy相当于是一个流量的出入口， 可以用来进行A/B tests, 蓝绿发布， 同样通过sidecar的流量数据，我们可以做好多的统计，比如超时， 重试，断路器等.

### Discovery and load balancing
![image](https://istio.io/docs/concepts/traffic-management/LoadBalancing.svg)
istio的三种负载算法：round robin, random, and weighted least request.

负载过程中会走断路器，调用过程中会对instance做健康检查， 断路器的开启是健康检查错误率真达到设置的域值时就开启。
换句话说，当给定实例的运行状况检查失败次数超过预先指定的阈值时，它将从负载平衡池中弹出。

### Handing failures
以下几种错误都可以处理

- Timeouts

- Bounded retries with timeout budgets and variable jitter between retries

- Limits on number of concurrent connections and requests to upstream services

- Active (periodic) health checks on each member of the load balancing pool

- Fine-grained circuit breakers (passive health checks) – applied per instance in the load balancing pool

都可以在 istio traffic management rules里进行配置管理。

### Rule configuration
Istio提供了一个简单的配置模型来控制API调用和第4层流量如何跨应用程序部署中的各种服务流动。四种配置资源:VirtualService, DestinationRule, ServiceEntry, and Gateway:

- A VirtualService defines the rules that control how requests for a service are routed within an Istio service mesh.

- A DestinationRule configures the set of policies to be applied to a request after VirtualService routing has occurred.

- A ServiceEntry is commonly used to enable requests to services outside of an Istio service mesh.

- A Gateway configures a load balancer for HTTP/TCP traffic, most commonly operating at the edge of the mesh to enable ingress traffic for an application

这一块内容繁杂， 回头单开一页。