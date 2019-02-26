分布式系统为什么需要注册中心/服务发现？

- 服务端发现 - Eureka
- 客户端发现 - Nginx、Zookeeper、Kubernetes

服务注册实现：心跳检测，检测注册的服务是否健康

高可用：通过Eureka之间相互注册实现高可用

<hr>

客户端负载均衡组件：Ribbon

- 服务发现
- 服务选择规则
- 服务监听

Ribbon主要组件：

- ServerList
- IRule
- ServerListFilter

RestTemplate & Feign % Zuul 三者都使用到了Ribbon





Zuul

![](https://ws1.sinaimg.cn/large/006tNc79gy1fzfhd42ngej30i40dk0wq.jpg)



限流方案：

令牌桶限流

![](https://ws3.sinaimg.cn/large/006tNc79gy1fzfk53osq2j30j80cydiw.jpg)