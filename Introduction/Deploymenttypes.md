## 部署

Envoy可用于各种不同的场景，但是在跨基础架构中进行所有主机网格部署时，它是最有用的。本节介绍三种推荐的部署方式，其复杂程度越来越高。

### 服务间

![部署图](service_to_service.svg)

上图显示了最简单的Envoy部署方式，使用Envoy作为通信总线，承担面向服务架构（SOA）内部所有的流量。在这种情况下，Envoy公开了几个用于本地来源流量的监听器，以及用于处理服务的流量。

#### 服务间出口监听器
这是应用程序与基础结构中的其他服务交互的端口。例如，http://localhost:9001 HTTP和gRPC请求使用HTTP/1.1主机头或HTTP/2：根据头来指导请求发往哪个远程群集。Envoy根据详细的配置处理服务发现，负载平衡，速率限制等。服务只需要了解本地的Envoy，不需要关心网络拓扑结构，无论是在开发还是在生产中运行。

此监听器支持HTTP/1.1或HTTP/2，具体取决于应用程序的功能。

#### 服务间入口监听器
这是远程Envoy想要与当地Envoy交谈时使用的端口。例如，http://localhost:9211 传入的请求被路由到配置的端口上的本地服务。可能会涉及多个应用程序端口，具体取决于应用程序或负载平衡需求（例如，如果服务同时需要HTTP端口和gRPC端口）。本地Envoy根据需要进行缓冲，断路等。

我们的默认配置对所有Envoy通信都使用HTTP/2，而不管应用程序在离开本地Envoy时是否使用HTTP/1.1或HTTP/2。HTTP/2支持长连接和显式重置通知，能够提供更好的性能。

#### 可选的外部服务出口监听器
通常，本地服务要与之通话的每个外部服务都使用明确的出口端口。这样做是因为一些外部服务SDK不轻易理解主机报文头，以支持标准的HTTP反向代理能力。例如，http://localhost:9250 可能被分配给发往DynamoDB的连接。我们建议为所有外部服务保持一致并使用本地端口路由，而不是为某些外部服务使用主机路由，为其他服务使用专用本地端口路由。

#### 集成发现服务
建议的配置使用外部发现服务进行所有群集发现。这为Envoy提供了在执行负载平衡，统计收集等时可能使用的详细的服务发现信息。

#### 配置模板
源代码发行版包含一个配置示例，与Lyft在生产环境中运行的版本非常相似。浏览此处获取[更多](../Buildingandinstallation/Referenceconfigurations.md)信息。

### 服务间+前端代理

![部署图](front_proxy.svg)

上图显示了服务部署，Envoy作为HTTP L7前端反向代理的群集。反向代理提供以下功能：

- 对外提供TLS安全，对内屏蔽TLS
- 支持HTTP/1.1和HTTP/2
- 完整的HTTP L7路由支持
- 提供标准入站端口，来访问Envoy集群服务，并使用发现服务进行主机查找。因此，前面的Envoy和任何其他的Envoy一样工作，除了他们没有与另一个服务进程搭配在一起。这意味着可以以相同的方式运行，并发出相同的统计数据。

#### 配置模板

源代码发行包含一个与Lyft在生产中运行的版本非常相似的示例前端代理配置。浏览此处获取[更多](../Buildingandinstallation/Referenceconfigurations.md)信息。

### 服务间、前端代理和双重代理


![部署图](double_proxy.svg)

上图显示了作为双重代理，运行了另一个Envoy做为前端代理。双重代理背后的想法是，尽可能地将TLS和客户端连接终止到用户（TLS握手的更短的往返时间，更快的TCP CWND扩展，更少的数据包丢失机会等），会更高效。在双重代理中终止的连接，然后被复用到在主数据中心中运行的HTTP/2长连接。

在上图中，在区域1中，运行的前端Envoy代理通过固定证书与在区域2中运行的前端Envoy代理进行身份验证。这允许在区域2中运行的前端Envoy实例，信任之前不能信任的入站转发请求（例如`x-forwarded-for`的HTTP头）。

#### 配置模板
源码分发包含一个与Lyft在生产中运行的版本非常相似的示例双重代理配置。浏览此处获取[更多](../Buildingandinstallation/Referenceconfigurations.md)信息。

## 返回
- [简介](../Introduction.md)
- [首页目录](../README.md)
